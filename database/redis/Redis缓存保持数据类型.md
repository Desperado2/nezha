## 一、背景
在通用的数据开放平台中，支持用户编写基于Groovy脚本的接口，Groovy脚本中可以查询数据库，然后对数据库中的数据进行一些处理。平台支持任何接口都可以启用缓存。

缓存不是缓存整个脚本的结果，而是只支持缓存数据库查询语句的结果，这样Groovy脚本中的其他逻辑依然可以处理数据。之前一直运行良好没有问题，最近数据接口开发人员反馈，某个新上的接口一旦开启缓存就会报错，结果沟通与排查发现， 是因为在缓存数据库查询结果到Redis中的时候，丢失了数据类型，导致后续的Groovy处理逻辑出现异常。



## 二、问题复现
首先来看这个Groovy查询脚本。这个脚本查询数据库中的数据，然后针对数据中的日期时间字段进行需要的处理，然后返回数据。

```groovy
def result_list = db.selectList('''
    select 
        id, title, url, publish_time
    from t1
    order by publish_time desc limit 50
''')

def date = (new Date().time / 1000).longValue()
for (data in result_list) {
    def publishTime = (data.publishTime.time / 1000).longValue()
    def diff_time = Math.round((date - publishTime) / 60)
    if (diff_time < 60) {
        data.put("diffTime", String.valueOf(diff_time) + '分钟前')
    } else if (diff_time >= 60 && diff_time < 1440) {
        data.put("diffTime", String.valueOf(Math.round(diff_time / 60)) + '小时前')
    } else {
        data.put("diffTime", data.publishTime)
    }
}
return result_list
```

	在SQL的查询结果中，包含了<font style="color:#DF2A3F;">publish_time </font>字段，该字段在StarRocks中的类型为DATETIME 类型，<font style="color:#DF2A3F;">db.selectList </font>方法的实现中，是直接将数据库返回的结果，包装为一个List<Map<String, Object>>进行返回，这时候我们在结果中会发现 <font style="color:rgb(223, 42, 63);">publish_time </font>字段的类型为Date，因此开发同学直接在下面的处理逻辑中使用了 <font style="color:#DF2A3F;">data.publishTime.time </font>方法来获取时间戳，这样如果从数据库中获取数据都是没有问题。

由于平台本身逻辑为开发阶段即使开启缓存也不会走缓存，因此在开发阶段，没有发现这个错误，接口上线发布之后，并且开启缓存之后，发现接口异常，报错信息显示：<font style="color:#DF2A3F;">属性丢失，Long类型没有time属性。</font>



## 三、问题分析
接口在开启缓存之后出现问题，也就是表示缓存中的数据存在问题，平台设计为既支持本地缓存也支持Redis缓存，测试之后发现，都存在问题，因此对缓存模块代码进行检查。

缓存采用AOP实现，通过拦截自定义注解来对数据库的查询结果进行缓存，简化后的缓存代码如下：

```java
@Around("dataCacheLayerPointcut()")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    CacheParams cacheParams = parseCacheParams(apiInfoContent);
    Boolean isLocalTest = apiInfoContent.getIsLocalTest();
    String cacheKey = null;
    // 不是开发环境 并且 启用缓存
    if(!isLocalTest && cacheParams.getEnableCache()){
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        //参数名数组
        String[] parameters =  methodSignature.getParameterNames();
        //参数值
        Object[] args = joinPoint.getArgs();
        DataCacheLayer dataCacheLayer = methodSignature.getMethod()
                                .getDeclaredAnnotation(DataCacheLayer.class);
        DataCacheType dataCacheType = dataCacheLayer.dataCacheType();
        cacheKey = generalCacheKey(dataCacheType, parameters, args);
        // 查询缓存
        Object cacheData = queryCache(cacheKey);
        if(cacheData != null){
            return JSONArray.parseArray(cacheData.toString(), Map.class);
        }
    }
    // 走原始逻辑
    Object proceed = joinPoint.proceed();
    // 设置缓存
    if(!isLocalTest && cacheParams.getEnableCache()){
        addCache(cacheKey, proceed);
    }
    return proceed;
}
```

从这个代码来看，我们发现从缓存中拿出来的数据是字符串，然后将其转换为了List<Map>, 这时候我们就发现，从字符串转换过来的数据，其数据类型发生了变化，<font style="color:#DF2A3F;">publishTime</font> 字段的类型从Date转变为了 Long，我们这时候继续查看具体的缓存代码，这里只看Redis缓存的实现。

```java
@Override
public void set(String cacheKey, Object value, Long cacheTime) {
    try {
        String jsonString = JSON.toJSONString(value);
        redisUtil.set(buildApiInfoKey(cacheKey), jsonString, cacheTime);
    }catch (Exception e){
        LOGGER.error("写缓存失败",e);
    }
}

@Override
public String get(String cacheKey) {
    return redisUtil.get(buildApiInfoKey(cacheKey));
}

```

	可以看到我们将一个对象转换为了一个json字符串，将其存储到了Redis中，然后在读取的时候也是读取到了一个字符串。分析到这里，我们认为这是由于json字符串转换导致，因此我们想到，如果我们在Redis中不存储转换之后的字符串而是直接存储对象，是否就能够解决这个问题。

经过实践之后发现，这样并不能解决问题，因为数据在存储到Redis中的时候，会经过Redis的数据序列化策略。默认的数据序列化策略，是将数据序列化为json进行存储，这时候Date类型的字段经过序列化和反序列化之后就变成了Long类型的时间戳。

经过分析和测试，明确了问题出现的根本原因，是因为Redis的数据序列化和反序列导致的；为了保持平台的灵活性和通用性，决定对平台缓存逻辑进行优化。



## 四、问题解决
经过调研，可以自定义Redis的序列化和反序列化逻辑。因此如果我们在数据存储进行Redis的时候保持其类型，那么在将数据读取出来之后，就可以根据类型进行恢复，这样就不会发现数据类型丢失。

经过分析之后，发现目前只有Date类型的存在问题，因此我们目前只考虑了Date类型的丢失问题，具体的解决方案为：

    1. 在将数据写入Redis的时候，也就是序列化阶段，判断数据是否为Date类型，如果是，将其值转换为特定的字符串数据格式：yyyy-MM-dd HH:mm:ss[type:date] ，然后将其写入Redis。
    2. 在从Redis读取数据的时候，也就是发序列化阶段，判断数据的值是否以[type:date]结尾，如果是将其转换为Date 类型的值。

自定义的序列化器如下：

```java
public class ComplexTypeRedisSerializer implements RedisSerializer<Object> {

    // 使用 FastDateFormat，它是线程安全的
    private static final FastDateFormat fastDateFormat = FastDateFormat.getInstance("yyyy-MM-dd HH:mm:ss");
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");


    @Override
    public byte[] serialize(Object value) throws SerializationException {
        if (value == null) {
            return new byte[0];
        }
        // 对所有 Map 和 List 类型的数据，遍历处理其中的 Date 字段并嵌入类型信息
        processDates(value);
        // 使用 fastjson 将对象序列化为 JSON 字符串
        String jsonString = JSON.toJSONString(value, SerializerFeature.WRITE_MAP_NULL_FEATURES, SerializerFeature.QuoteFieldNames);
        return jsonString.getBytes(StandardCharsets.UTF_8);
    }

    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length == 0) {
            return null;
        }
        // 将字节数组转换为 JSON 字符串
        String jsonString = new String(bytes, StandardCharsets.UTF_8);
        // 反序列化 JSON 字符串为 Map 或其他对象
        Object jsonObject = JSON.parse(jsonString);
        recursiveParse(jsonObject);
        // 根据字段的类型信息来还原字段类型
        return jsonObject;
    }


    // 递归解析 JSON 对象
    private  void recursiveParse(Object object) {
        if (object instanceof JSONObject) {
            JSONObject jsonObject = (JSONObject) object;
            for (Map.Entry<String, Object> entry : jsonObject.entrySet()) {
                // 递归调用解析每一个字段
                Object value = entry.getValue();
                if(value != null && value.toString().endsWith("[type:date]")){
                    jsonObject.put(entry.getKey(), getDate(value.toString()));
                }else{
                    recursiveParse(value);
                }
            }
        } else if (object instanceof JSONArray) {
            JSONArray jsonArray = (JSONArray) object;
            for (Object o : jsonArray) {
                // 递归调用解析每个数组元素
                if(o != null && o.toString().endsWith("[type:date]")){
                    jsonArray.add(getDate(o.toString()));
                }else{
                    recursiveParse(o);
                }

            }
        }
    }


    private Object getDate(String value) {
        if(value.endsWith("[type:date]")){
            String dateString = value.substring(0, value.indexOf("[type:date]"));
            try {
               return Date.from(LocalDateTime.parse(dateString, formatter).atZone(ZoneId.systemDefault()).toInstant());
            }catch (Exception e){
                return dateString;
            }
        }
        return value;
    }

    /**
     *  递归处理Map或者List<Map>中的Date类型
     * @param object 对象
     */
    private  void processDates(Object object) {
        if(object instanceof PageList){
            object = ((PageList) object).getDataList();
        }
        if (object instanceof Map) {
            // 如果是 Map 类型，递归处理每个条目
            Map<Object, Object> map = (Map<Object, Object>) object;
            for (Map.Entry<?, ?> entry : map.entrySet()) {
                // 如果值是 Date 类型，更新为格式化字符串
                if (entry.getValue() instanceof Date) {
                    map.put(entry.getKey(), fastDateFormat.format(entry.getValue()) + "[type:date]");
                }else{
                    // 递归更新每个条目的值
                    processDates(entry.getValue());
                }
            }
        } else if (object instanceof List) {
            // 如果是 List 类型，递归处理每个元素
            List<Object> list = (List<Object>) object;
            for (int i = 0; i < list.size(); i++) {
                Object item = list.get(i);
                // 如果元素是 Date 类型，修改它
                if (item instanceof Date) {
                    list.set(i, fastDateFormat.format(item) + "[type:date]");
                }else{
                    // 递归更新列表中的元素
                    processDates(item);
                }
            }
        }
    }
}

```

然后将自定义的序列化器设置给Redis的value的序列化策略。

```java
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    // 设置value的序列化规则和 key的序列化规则
    redisTemplate.setValueSerializer(new ComplexTypeRedisSerializer());
    // 使用自定义的序列化器
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.afterPropertiesSet();
    return redisTemplate;
}
```



## 五、存在待优化的地方
目前整个判断是否包含Date类型的数据的逻辑比较粗暴，直接通过遍历的方式，后续可以优化，在其中加入特殊标识来标识是存在，提示数据处理速度。

