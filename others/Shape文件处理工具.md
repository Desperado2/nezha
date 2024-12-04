## 一、Shape文件介绍
ESRI Shapefile（shp），或简称shapefile，是[美国环境系统研究所公司](https://baike.baidu.com/item/%E7%BE%8E%E5%9B%BD%E7%8E%AF%E5%A2%83%E7%B3%BB%E7%BB%9F%E7%A0%94%E7%A9%B6%E6%89%80%E5%85%AC%E5%8F%B8/1528104)（ESRI）开发的一种[空间数据](https://baike.baidu.com/item/%E7%A9%BA%E9%97%B4%E6%95%B0%E6%8D%AE/2271605)开放格式。[1]该文件格式已经成为了地理信息软件界的一个开放标准，这表明ESRI公司在全球的[地理信息系统](https://baike.baidu.com/item/%E5%9C%B0%E7%90%86%E4%BF%A1%E6%81%AF%E7%B3%BB%E7%BB%9F/171830)市场的重要性。Shapefile也是一种重要的交换格式，它能够在ESRI与其他公司的产品之间进行数据互操作。

Shapefile文件用于描述几何体对象：[点](https://baike.baidu.com/item/%E7%82%B9)，折线与[多边形](https://baike.baidu.com/item/%E5%A4%9A%E8%BE%B9%E5%BD%A2)。例如，Shapefile文件可以存储[井](https://baike.baidu.com/item/%E4%BA%95)、河流、[湖泊](https://baike.baidu.com/item/%E6%B9%96%E6%B3%8A)等空间对象的几何位置。除了几何位置，shp文件也可以存储这些空间对象的属性，例如一条河流的名字，一个城市的温度等等。



## 二、Shape文件组成
Shapefile文件指的是一种文件存储的方法，实际上该种文件格式是由多个文件组成的。其中，要组成一个Shapefile，有三个文件是必不可少的，它们分别是".shp", ".shx"与 ".dbf"文件。表示同一数据的一组文件其文件名前缀应该相同。例如，存储一个关于湖的几何与属性数据，就必须有lake.shp，lake.shx与lake.dbf三个文件。而其中“真正”的Shapefile的后缀为shp，然而仅有这个文件数据是不完整的，必须要把其他两个附带上才能构成一组完整的地理数据。除了这三个必须的文件以外，还有八个可选的文件，使用它们可以增强空间数据的表达能力。所有的文件名都必须遵循MS DOS的8.3文件名标准（文件前缀名8个字符，后缀名3个字符，如shapefil.shp），以方便与一些老的应用程序保持兼容性，尽管现在许多新的程序都能够支持长文件名。此外，所有的文件都必须位于同一个目录之中。

必须的文件:

+ .shp— 图形格式，用于保存元素的几何实体。
+ .shx— 图形索引格式。几何体位置索引，记录每一个几何体在shp文件之中的位置，能够加快向前或向后搜索一个几何体的效率。
+ .dbf— 属性数据格式，以dBase IV的数据表格式存储每个几何形状的属性数据。

其他可选的文件：

+ .prj— 投帧式，用于保存地理坐标系统与投影信息，是一个存储well-known text投影描述符的文本文件。
+ .sbnand.sbx— 几何体的空间索引
+ .fbnand.fbx— 只读的Shapefiles的几何体的空间索引
+ .ainand.aih— 列表中活动字段的属性索引。
+ .ixs— 可读写Shapefile文件的地理编码索引
+ .mxs— 可读写Shapefile文件的地理编码索引(ODB格式)
+ .atx—.dbf文件的属性索引，其文件名格式为_shapefile_._columnname_.atx(ArcGIS 8及之后的版本)
+ .shp.xml— 以XML格式保存元数据。
+ .cpg— 用于描述.dbf文件的[代码页](https://baike.baidu.com/item/%E4%BB%A3%E7%A0%81%E9%A1%B5)，指明其使用的[字符编码](https://baike.baidu.com/item/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81)。

在每个.shp,.shx与.dbf文件之中，图形在每个文件的排序是一致的。也就是说，.shp的第一条记录与.shx及.dbf之中的第一条记录相对应，如此类推。此外，在.shp与.shx之中，有许多字段的[字节序](https://baike.baidu.com/item/%E5%AD%97%E8%8A%82%E5%BA%8F)是不一样的。因此用户在编写读取这些文件格式的程序时，必须十分小心地处理不同文件的不同字节序。

Shapefile通常以X与Y的方式来处理地理坐标，一般X对应经度，Y对应纬度，用户必须注意X，Y的顺序。

**Shapefile图形格式 (.shp)**

Shapefile格式的主文件包含了地理参照数据。该文件由一个定长的文件头和一个或若干个变长的记录数据组成。每一条变长数据记录包含一个记录头和一些记录内容。详细的数据存储格式由_Esri Shapefile技术描述_.提供。注意，虽然Shapefile文件的后缀名与[AutoCAD](https://baike.baidu.com/item/AutoCAD)的图形字体源格式它们的[文件后缀名](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E5%90%8E%E7%BC%80%E5%90%8D/492702)相同的，都是.shp，请不要把它们混淆。

主文件头包含17个字段，共100个字节，其中包含九个4字节（32位有符号整数，int32）整数字段，紧接着是八个8字节（[双精度浮点数](https://baike.baidu.com/item/%E5%8F%8C%E7%B2%BE%E5%BA%A6%E6%B5%AE%E7%82%B9%E6%95%B0)）有符号浮点数字段。



## 三、MySQL对地理空间数据的支持
##### 1.Geometry介绍
+ MySQL中支持的几何数据类型包括Geometry（几何）、Point（点）、LineString（线）、Polygon（面）  
以及集合类型的MultiPoint（多点）、MultiLineString（多线）、MultiPolygon（多面）、GeometryCollection（混合数据类型）
+ 其中，Geometry可以表示其他任意类型的值，剩下的只能表示单个类型的值

##### 2.Geometry类型
注意：数据中间不能有多余的空格

| 名称 | 类型 | 例如 |
| --- | --- | --- |
| Point | 点坐标 | POINT(103 35) |
| LineString | 线坐标 | LINESTRING(103 35,103 36,104 36,105 37) |
| Polygon | 面坐标 | POLYGON(103 35,104 35,104 36,103 36,103 35) |
| MultiPoint | 多点 | MULTIPOINT(103 35, 104 34,105 35) |
| MultiLineString | 多线 | MULTILINESTRING((103 35, 104 35), (105 36, 105 37)) |
| MultiPolygon | 多面 | MULTIPOLYGON(((103 35,104 35,104 36,103 36,103 35)),((103 36,104 36,104 37,103 36))) |
| GeometryCollection | 混合类型 | GEOMETRYCOLLECTION(POINT(103 35), LINESTRING(103 35, 103 37)) |


**3、Geometry数据格式**

MySQ L支持WKB,WKT数据生成空间数据类型，提供如下函数：

+ WKT（文本格式：在代码中的格式）

GeomFromText（wtk [，srid）PointFromText LINESTRINGFROMTEXT ......

+ WKB（二进制格式：存储在Geometry类型的表字段中）

GeomFromWKB（wtk [，srid）GeomFromWKB GeomFromWKB ......

**4、Geometry的常用函数**

##### 1.构造函数
构造函数会获取一种几何类型或几何的文本说明，然后创建一个几何

+ ST_Point：文本格式转Point格式（例如存表的时候）
+ ST_PointFromText：Point格式转文本格式（例如查询的时候）
+ ST_Polygon：文本格式转Polygon格式
+ ST_PolygonFromText：Polygon格式转文本格式
+ ST_PointFromWKB：以熟知二进制 (WKB) 表示和空间参考 ID 作为输入参数返回 ST_Point 类型的对象

##### 2.存取器函数
函数都采用一个或多个几何作为输入，并返回关于几何的特定信息

+ 获取线/面对象四至：st_xmin(geometry)、st_ymin(geometry)、st_xmax(geometry)、st_ymax(geometry)
+ ST_AsText：获取一个几何类型，然后返回其可识别的文本表示
+ ST_AsGeoJSON：将Geometry格式转为JSON格式
+ ST_Centroid：以面或多面为参数输入，然后返回位于几何的包络矩形中心的点
+ ST_Length：用于返回线串或多线串的长度
+ ST_MaxX：以几何为参数，返回最大的 X 坐标
+ ST_SRID：以几何对象作为输入参数，并返回其空间参考 ID
+ ST_X：返回点坐标的 X 坐标
+ ST_Y：返回点坐标的 Y 坐标

##### 3.关系函数
关系函数将几何作为输入并确定各几何之间是否存在特定关系

+ ST_Contains：判断第一个几何对象是否完全包含第二个几何对象
+ ST_Disjoint：判断两个几何对象无交集
+ ST_Equals：判断两个几何对象是否完全相同

##### 4.几何函数
函数利用空间数据并对其执行分析，然后返回新的空间数据

+ ST_Buffer：获取几何对象和距离，然后返回表示围绕源对象的缓冲区的几何对象（例如可以使用线坐标，构造一个线坐标50米之内的面）
+ ST_Distance：用于返回两个几何之间的距离。这一距离是两个几何的最近折点之间的距离
+ ST_Difference：获取两个几何对象，然后返回表示两个源对象之差的几何对象（例如，计算两个面积差）

**更多相关函数可参考：**[ArcMap](https://desktop.arcgis.com/zh-cn/arcmap/10.3/manage-data/using-sql-with-gdbs/a-quick-tour-of-sql-functions-used-with-st-geometry.htm)



## 四、Shape文件解析
shape文件解析使用geopandas进行解析和坐标的转换。

**核心解析逻辑如下**：

```python
# 引入geopandas
from geopandas.io.file import read_file

# 读取prj文件
crs = read_crs_from_prj_file(self.prj)
# 读取shp文件信息，变成GeoDataFrame
data = read_file(shp, encoding="gbk")
# 设置数据原始坐标系数据
data.to_crs(crs=crs)
# 转换成WGS84投影坐标系
df = data.to_crs(crs="+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")

# 转换成GEOJSON
outpoint_f = 'geometry.json'
df2.to_file(outpoint_f, driver='GeoJSON', encoding='utf-8')

# 生成可视化图片
df2.plot(color='red')
pyplot.savefig('geometry.svg', dpi=1000)

# 导出excel
columnname = list(df)
df.to_excel(self.save_folder + '/' + 'geometry.xlsx', index=None, header=columnname)
    
def read_crs_from_prj_file(self, prjfile: str):
    """
    读取prj文件，获取坐标
    :param prjfile: prj文件地址
    :return: prj文件内容
    """
    f = open(prjfile, "r")
    result = f.read().encode("utf-8")
    f.close()
    return str(result, encoding='utf-8')
```

## 五、两种解析方式
**1.命令行解析**

本方式解析方式灵活。可随意修改代码进行解析，但是必须要懂得python，能够配置相应的python开发环境以及修改python代码。

**代码如下：**

```python
import geopandas
import json
import geodaisy.converters as convert
import matplotlib.pyplot as pyplot


def read_crs_from_prj_file(prjfile: str):
    """
    读取prj文件，获取坐标
    :param prjfile: prj文件地址
    :return: prj文件内容
    """
    f = open(prjfile, "r")
    result = f.read().encode("utf-8")
    f.close()
    print("prj文件读取成功")
    return str(result, encoding='utf-8')


def convert_geometry(file: str, crs: str):
    """
    转换shape文件的坐标系，进行列名转换，生成数据的可视化图片
    :param file: shape文件
    :param crs: prj文件内容
    :param column_name_map: 要进行转换列名的列的字典
        {
            "原列名1":"新列名1",
            "原列名2":"新列名2",
               .         .
               .         .
               .         .
            "原列名n":"新列名n"
        }
    :return: 新生成的文件
    """
    data = geopandas.read_file(file, encoding="gbk")
    data.to_crs(crs=crs)
    reprojected_data = data.to_crs(crs="+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")
    reprojected_data.to_file("result.shp", encoding="gbk")
    print("转换格式后的文件生成成功，名称为：result.shp")
    reprojected_data.plot(color='red')

    pyplot.savefig('geometry.svg', dpi=1000)
    print("数据可视化图片生成成功，名称为: geometry.svg")
    return "result.shp"


def read_shape(file: str, column_name_map=None):
    """
    将shape文件转换为GEOJson文件
    :param file: shape文件地址
    :return: GeoJson文件名称
    """
    df2 = geopandas.read_file(file, encoding="gbk")
    if column_name_map is not None:
        df2 = df2.rename(columns=column_name_map)
    outpoint_f = 'geometry.json'
    df2.to_file(outpoint_f, driver='GeoJSON', encoding='utf-8')
    print("GeoJson文件生成成功，名称为: geometry.json")
    return outpoint_f


def parse_geo_json_file(file: str):
    """
    将GEOJson文件转换为列表对象
    :param file: GeoJOSN文件地址
    :return: 对象列表
    """
    with open(file, 'r', encoding='utf8')as fp:
        json_data = json.load(fp)
        datass = json_data['features']
        values_list = []
        for a in datass:
            b = a
            value = b['properties']
            columns_list = []
            for x in value:
                columns_list.append(x)
            values = {}
            for key in columns_list:
                vv = value.get(key)
                if vv is None:
                    vv = ''
                values[key] = vv
            value2 = b['geometry']
            corrd = str(value2['coordinates'][0])
            values['coordinate'] = corrd
            values_list.append(values)
    return values_list


def parse_geo_json_file_by_wkt(file: str):
    """
    将GEOJson文件转换为列表对象,同时将GEOJson转换为wkt
    :param file: GEOJson文件地址
    :return: 对象列表
    """
    with open(file, 'r', encoding='utf8')as fp:
        json_data = json.load(fp)
        datass = json_data['features']
        values_list = []
        for a in datass:
            b = a
            value = b['properties']
            columns_list = []
            for x in value:
                columns_list.append(x)
            values = {}
            for key in columns_list:
                vv = value.get(key)
                if vv is None:
                    vv = ''
                values[key] = vv
            value2 = b['geometry']
            wkt = convert.geojson_to_wkt(value2)
            values['coordinate'] = wkt
            values_list.append(values)
    return values_list


def parse_geo_json_file_by_wkt_and_text(file: str, need_muiliti: bool):
    """
    将GEOJson文件转换为列表对象,同时将GEOJson转换为wkt
    :param file: GEOJson文件地址
    :return: 对象列表
    """
    with open(file, 'r', encoding='utf8')as fp:
        json_data = json.load(fp)
        datass = json_data['features']
        values_list = []
        for a in datass:

            b = a
            value = b['properties']
            columns_list = []
            for x in value:
                columns_list.append(x)
            values = {}
            values['unique_id'] = str(uuid.uuid4()).replace("-", '')
            for key in columns_list:
                vv = value.get(key)
                if vv is None:
                    vv = ''
                values[key] = vv
            value2 = b['geometry']

            corrd = str(value2['coordinates'][0])
            values['geometry'] = corrd

            wkt = convert.geojson_to_wkt(value2)
            if need_muiliti and wkt.startswith("POLYGON"):
                wkt = wkt.replace("POLYGON", "MULTIPOLYGON (")
                wkt = wkt + ")"
            values['coordinate'] = wkt
            values_list.append(values)
            values['shape_type'] = 'Polygon'
    return values_list


def general_sql(table_name: str, values_list: list):
    """
    生成sql文件
    :param table_name: 表名
    :param values_list: 数据对象列表
    :return: sql语句文件地址
    """
    sql_list = []
    for data in values_list:
        keys = []
        values = []
        for dd in data:
            keys.append(dd)
            if dd == 'coordinate':
                values.append("ST_GeomFromText('" + str(data[dd]) + "')")
            else:
                values.append("'" + str(data[dd]) + "'")

        sql_list.append("insert into {}({}) values ({})".format(table_name, ','.join(keys), ','.join(values)))
    with open("geometry.sql", "w+", encoding='utf-8') as f:
        f.write(";\n".join(sql_list))
    print("SQL文件生成成功，名称为: geometry.sql")
    return "geometry.sql"


if __name__ == '__main__':
    """
        示例代码解析
    """
    columns_name_map = {
        "投影面积": "projectionArea",
        "序号": "xh",
        "西至": "west",
        "茶叶品种": "teaType",
        "审核人": "auditor",
        "土地座落": "located",
        "备注": "bz",
        "茶地权属": "teaOwnership",
        "乡镇": "town",
        "经营方式": "businessPractice",
        "村名": "villageName",
        "茶地类型": "teaLandType",
        "茶叶长势": "teaGrowing",
        "图斑号": "patternNumber",
        "身份证号1": "idNumber1",
        "联系电话2": "tel2",
        "村名新": "newVillageName",
        "联系电话1": "tel1",
        "身份证号2": "idNumber2",
        "登记日期": "recordDate",
        "协助调查人": "assistingInvest",
        "东至": "east",
        "土壤类型": "soilType",
        "调查人": "inquirer",
        "coordinate": "coordinate",
        "表面积": "area",
        "组别": "group",
        "使用人": "user",
        "shape_le_1": "shapeLe1",
        "保护等级": "saveLevel",
        "shape_le_2": "shapeLe2",
        "shape_le_3": "shapeLe3",
        "备注2016": "bz2016",
        "坡度": "slope",
        "茶树龄": "teaAge",
        "shape_leng": "shapeLeng",
        "南至": "south",
        "北至": "north",
        "shape_area": "shapeArea",
        "权利人": "rightPeo",
        "茶地现状": "teaCondition",
        "等级类别": "levelType",
        "使用期限": "useAgeLimit"
    }
    crs = read_crs_from_prj_file("C:\\Users\\wz\\Downloads\\西湖龙井地块\\二期总库2000.prj")
    result = convert_geometry("C:\\Users\\wz\\Downloads\\西湖龙井地块\\二期总库2000.shp", crs, columns_name_map)
    json_file = read_shape(result)
    data = parse_geo_json_file_by_wkt(json_file)
    sql_list = general_sql('tabless', data)


```



**2.使用ShapeUtils.exe解析（打开有点慢）**

本方式使用简单，只要使用软件输入相关shape文件地址信息，即可查看shape结构、生成示意图、生成geojson、生成sql、生成excel等，但也相对限制了灵活度。

**代码如下：**

```python
# -*- coding: utf-8 -*-

# Form implementation generated from reading ui file 'windows.ui'
#
# Created by: PyQt5 UI code generator 5.15.4
#
# WARNING: Any manual changes made to this file will be lost when pyuic5 is
# run again.  Do not edit this file unless you know what you are doing.


import sys

import cv2
from PyQt5.QtWidgets import QApplication, QMainWindow, QMessageBox, QTableWidgetItem, QAbstractItemView, \
    QWidget, QHBoxLayout, QLabel, QGraphicsScene, QGraphicsPixmapItem
from PyQt5 import QtCore, QtWidgets
from PyQt5.Qt import QThread, QMutex, pyqtSignal
from PyQt5.QtWidgets import QFileDialog
from PyQt5.QtCore import QSize, Qt
from PyQt5.QtGui import QFontMetrics, QFont, QMovie, \
    QMoveEvent, QImage, QPixmap

import os
from geopandas.io.file import read_file
import json
import geodaisy.converters as convert
from geopandas.geodataframe import GeoDataFrame
from matplotlib import pyplot
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg
from fiona import _shim, schema

curr_tab_index = 0
qmut_1 = QMutex()  # 创建线程锁


class Ui_MainWindow(object):
    def setupUi(self, MainWindow):
        MainWindow.setObjectName("MainWindow")
        MainWindow.resize(800, 600)
        self.centralwidget = QtWidgets.QWidget(MainWindow)
        self.centralwidget.setObjectName("centralwidget")
        self.lookStructs = QtWidgets.QTabWidget(self.centralwidget)
        self.lookStructs.setGeometry(QtCore.QRect(0, 0, 801, 601))
        self.lookStructs.setObjectName("lookStructs")
        self.tab = QtWidgets.QWidget()
        self.tab.setObjectName("tab")
        self.label = QtWidgets.QLabel(self.tab)
        self.label.setGeometry(QtCore.QRect(20, 40, 101, 16))
        self.label.setObjectName("label")
        self.label_2 = QtWidgets.QLabel(self.tab)
        self.label_2.setGeometry(QtCore.QRect(20, 110, 101, 16))
        self.label_2.setObjectName("label_2")
        self.label_3 = QtWidgets.QLabel(self.tab)
        self.label_3.setGeometry(QtCore.QRect(20, 90, 101, 16))
        self.label_3.setText("")
        self.label_3.setObjectName("label_3")
        self.shapeFile = QtWidgets.QLineEdit(self.tab)
        self.shapeFile.setGeometry(QtCore.QRect(90, 30, 531, 41))
        self.shapeFile.setObjectName("shapeFile")
        self.prjShape = QtWidgets.QLineEdit(self.tab)
        self.prjShape.setGeometry(QtCore.QRect(90, 100, 531, 41))
        self.prjShape.setObjectName("prjShape")
        self.chooseShapeFile = QtWidgets.QPushButton(self.tab)
        self.chooseShapeFile.setGeometry(QtCore.QRect(670, 30, 101, 41))
        self.chooseShapeFile.setObjectName("chooseShapeFile")
        self.choosePrjFile = QtWidgets.QPushButton(self.tab)
        self.choosePrjFile.setGeometry(QtCore.QRect(670, 100, 101, 41))
        self.choosePrjFile.setObjectName("choosePrjFile")
        self.label_4 = QtWidgets.QLabel(self.tab)
        self.label_4.setGeometry(QtCore.QRect(20, 180, 54, 12))
        self.label_4.setObjectName("label_4")
        self.dataStructsTable = QtWidgets.QTableWidget(self.tab)
        self.dataStructsTable.setGeometry(QtCore.QRect(90, 170, 681, 321))
        self.dataStructsTable.setObjectName("dataStructsTable")
        self.dataStructsTable.setColumnCount(0)
        self.dataStructsTable.setRowCount(0)
        self.parseDataButton = QtWidgets.QPushButton(self.tab)
        self.parseDataButton.setGeometry(QtCore.QRect(90, 500, 141, 41))
        self.parseDataButton.setObjectName("parseDataButton")
        self.lookStructs.addTab(self.tab, "")
        self.tab_6 = QtWidgets.QWidget()
        self.tab_6.setObjectName("tab_6")
        self.chooseShapeFile_2 = QtWidgets.QPushButton(self.tab_6)
        self.chooseShapeFile_2.setGeometry(QtCore.QRect(670, 30, 101, 41))
        self.chooseShapeFile_2.setObjectName("chooseShapeFile_2")
        self.choosePrjFile_2 = QtWidgets.QPushButton(self.tab_6)
        self.choosePrjFile_2.setGeometry(QtCore.QRect(670, 100, 101, 41))
        self.choosePrjFile_2.setObjectName("choosePrjFile_2")
        self.shapeFile_2 = QtWidgets.QLineEdit(self.tab_6)
        self.shapeFile_2.setGeometry(QtCore.QRect(90, 30, 531, 41))
        self.shapeFile_2.setObjectName("shapeFile_2")
        self.label_20 = QtWidgets.QLabel(self.tab_6)
        self.label_20.setGeometry(QtCore.QRect(20, 110, 101, 16))
        self.label_20.setObjectName("label_20")
        self.parseDataButton_2 = QtWidgets.QPushButton(self.tab_6)
        self.parseDataButton_2.setGeometry(QtCore.QRect(90, 500, 141, 41))
        self.parseDataButton_2.setObjectName("parseDataButton_2")
        self.label_21 = QtWidgets.QLabel(self.tab_6)
        self.label_21.setGeometry(QtCore.QRect(20, 40, 101, 16))
        self.label_21.setObjectName("label_21")
        self.prjShape_2 = QtWidgets.QLineEdit(self.tab_6)
        self.prjShape_2.setGeometry(QtCore.QRect(90, 100, 531, 41))
        self.prjShape_2.setObjectName("prjShape_2")

        self.zoomout = QtWidgets.QPushButton(self.tab_6)
        self.zoomout.setGeometry(QtCore.QRect(20, 160, 40, 20))
        self.zoomout.setObjectName("zoomout")

        self.zoomin = QtWidgets.QPushButton(self.tab_6)
        self.zoomin.setGeometry(QtCore.QRect(20, 180, 40, 20))
        self.zoomin.setObjectName("zoomin")


        self.graphicsView = QtWidgets.QGraphicsView(self.tab_6)
        self.graphicsView.setGeometry(QtCore.QRect(90, 150, 681, 341))
        self.graphicsView.setObjectName("graphicsView")


        self.lookStructs.addTab(self.tab_6, "")
        self.tab_2 = QtWidgets.QWidget()
        self.tab_2.setObjectName("tab_2")
        self.convertJSButton = QtWidgets.QPushButton(self.tab_2)
        self.convertJSButton.setGeometry(QtCore.QRect(90, 500, 141, 41))
        self.convertJSButton.setObjectName("convertJSButton")
        self.js_shapeFile = QtWidgets.QLineEdit(self.tab_2)
        self.js_shapeFile.setGeometry(QtCore.QRect(90, 30, 531, 41))
        self.js_shapeFile.setObjectName("js_shapeFile")
        self.chooseJSShapeFile = QtWidgets.QPushButton(self.tab_2)
        self.chooseJSShapeFile.setGeometry(QtCore.QRect(670, 30, 101, 41))
        self.chooseJSShapeFile.setObjectName("chooseJSShapeFile")
        self.label_5 = QtWidgets.QLabel(self.tab_2)
        self.label_5.setGeometry(QtCore.QRect(20, 40, 101, 16))
        self.label_5.setObjectName("label_5")
        self.label_6 = QtWidgets.QLabel(self.tab_2)
        self.label_6.setGeometry(QtCore.QRect(20, 110, 101, 16))
        self.label_6.setObjectName("label_6")
        self.js_prjShape = QtWidgets.QLineEdit(self.tab_2)
        self.js_prjShape.setGeometry(QtCore.QRect(90, 100, 531, 41))
        self.js_prjShape.setObjectName("js_prjShape")
        self.chooseJSPrjFile = QtWidgets.QPushButton(self.tab_2)
        self.chooseJSPrjFile.setGeometry(QtCore.QRect(670, 100, 101, 41))
        self.chooseJSPrjFile.setObjectName("chooseJSPrjFile")
        self.label_11 = QtWidgets.QLabel(self.tab_2)
        self.label_11.setGeometry(QtCore.QRect(10, 180, 81, 16))
        self.label_11.setObjectName("label_11")
        self.geojsonSaveFile = QtWidgets.QLineEdit(self.tab_2)
        self.geojsonSaveFile.setGeometry(QtCore.QRect(90, 170, 531, 41))
        self.geojsonSaveFile.setObjectName("geojsonSaveFile")
        self.chooseGeoJsonSaveFile = QtWidgets.QPushButton(self.tab_2)
        self.chooseGeoJsonSaveFile.setGeometry(QtCore.QRect(670, 170, 101, 41))
        self.chooseGeoJsonSaveFile.setObjectName("chooseGeoJsonSaveFile")
        self.label_12 = QtWidgets.QLabel(self.tab_2)
        self.label_12.setGeometry(QtCore.QRect(10, 260, 54, 12))
        self.label_12.setObjectName("label_12")
        self.label_13 = QtWidgets.QLabel(self.tab_2)
        self.label_13.setGeometry(QtCore.QRect(20, 330, 54, 12))
        self.label_13.setObjectName("label_13")
        self.columnConvertMap = QtWidgets.QTextEdit(self.tab_2)
        self.columnConvertMap.setGeometry(QtCore.QRect(90, 310, 531, 181))
        self.columnConvertMap.setObjectName("columnConvertMap")
        self.checkBox = QtWidgets.QCheckBox(self.tab_2)
        self.checkBox.setGeometry(QtCore.QRect(90, 260, 211, 21))
        self.checkBox.setObjectName("checkBox")

        self.lookStructs.addTab(self.tab_2, "")
        self.tab_3 = QtWidgets.QWidget()
        self.tab_3.setObjectName("tab_3")
        self.convertSQLButton = QtWidgets.QPushButton(self.tab_3)
        self.convertSQLButton.setGeometry(QtCore.QRect(90, 500, 141, 41))
        self.convertSQLButton.setObjectName("convertSQLButton")
        self.label_14 = QtWidgets.QLabel(self.tab_3)
        self.label_14.setGeometry(QtCore.QRect(10, 180, 81, 16))
        self.label_14.setObjectName("label_14")
        self.sql_columnConvertMap = QtWidgets.QTextEdit(self.tab_3)
        self.sql_columnConvertMap.setGeometry(QtCore.QRect(90, 310, 531, 181))
        self.sql_columnConvertMap.setObjectName("sql_columnConvertMap")
        self.sql_shapeFile = QtWidgets.QLineEdit(self.tab_3)
        self.sql_shapeFile.setGeometry(QtCore.QRect(90, 30, 531, 41))
        self.sql_shapeFile.setObjectName("sql_shapeFile")
        self.sql_chooseShapeFile = QtWidgets.QPushButton(self.tab_3)
        self.sql_chooseShapeFile.setGeometry(QtCore.QRect(670, 30, 101, 41))
        self.sql_chooseShapeFile.setObjectName("sql_chooseShapeFile")
        self.label_15 = QtWidgets.QLabel(self.tab_3)
        self.label_15.setGeometry(QtCore.QRect(10, 240, 54, 12))
        self.label_15.setObjectName("label_15")
        self.sqlchooseGeoJsonSaveFile = QtWidgets.QPushButton(self.tab_3)
        self.sqlchooseGeoJsonSaveFile.setGeometry(QtCore.QRect(670, 170, 101, 41))
        self.sqlchooseGeoJsonSaveFile.setObjectName("sqlchooseGeoJsonSaveFile")
        self.sql_choosePrjFile = QtWidgets.QPushButton(self.tab_3)
        self.sql_choosePrjFile.setGeometry(QtCore.QRect(670, 100, 101, 41))
        self.sql_choosePrjFile.setObjectName("sql_choosePrjFile")
        self.label_16 = QtWidgets.QLabel(self.tab_3)
        self.label_16.setGeometry(QtCore.QRect(20, 330, 54, 12))
        self.label_16.setObjectName("label_16")
        self.label_7 = QtWidgets.QLabel(self.tab_3)
        self.label_7.setGeometry(QtCore.QRect(20, 110, 101, 16))
        self.label_7.setObjectName("label_7")
        self.sql_prjShape = QtWidgets.QLineEdit(self.tab_3)
        self.sql_prjShape.setGeometry(QtCore.QRect(90, 100, 531, 41))
        self.sql_prjShape.setObjectName("sql_prjShape")
        self.label_8 = QtWidgets.QLabel(self.tab_3)
        self.label_8.setGeometry(QtCore.QRect(20, 40, 101, 16))
        self.label_8.setObjectName("label_8")
        self.sqlSaveFile = QtWidgets.QLineEdit(self.tab_3)
        self.sqlSaveFile.setGeometry(QtCore.QRect(90, 170, 531, 41))
        self.sqlSaveFile.setObjectName("sqlSaveFile")
        self.checkBox_3 = QtWidgets.QCheckBox(self.tab_3)
        self.checkBox_3.setGeometry(QtCore.QRect(90, 240, 201, 21))
        self.checkBox_3.setObjectName("checkBox_3")
        self.checkBox_4 = QtWidgets.QCheckBox(self.tab_3)
        self.checkBox_4.setGeometry(QtCore.QRect(310, 240, 161, 21))
        self.checkBox_4.setObjectName("checkBox_4")
        self.lookStructs.addTab(self.tab_3, "")

        self.label_111 = QtWidgets.QLabel(self.tab_3)
        self.label_111.setGeometry(QtCore.QRect(10, 280, 101, 16))
        self.label_111.setObjectName("label_111")

        self.label_222 = QtWidgets.QLabel(self.tab_3)
        self.label_222.setGeometry(QtCore.QRect(90, 280, 101, 16))
        self.label_222.setObjectName("label_222")

        self.sql_coord_name = QtWidgets.QLineEdit(self.tab_3)
        self.sql_coord_name.setGeometry(QtCore.QRect(170, 280, 100, 21))
        self.sql_coord_name.setObjectName("sql_coord_name")

        self.label_333 = QtWidgets.QLabel(self.tab_3)
        self.label_333.setGeometry(QtCore.QRect(280, 280, 161, 16))
        self.label_333.setObjectName("label_333")

        self.sql_coord_text_name = QtWidgets.QLineEdit(self.tab_3)
        self.sql_coord_text_name.setGeometry(QtCore.QRect(400, 280, 100, 21))
        self.sql_coord_text_name.setObjectName("sql_coord_text_name")

        self.label_444 = QtWidgets.QLabel(self.tab_3)
        self.label_444.setGeometry(QtCore.QRect(500, 280, 161, 16))
        self.label_444.setObjectName("label_444")

        self.sql_table_name = QtWidgets.QLineEdit(self.tab_3)
        self.sql_table_name.setGeometry(QtCore.QRect(530, 280, 100, 21))
        self.sql_table_name.setObjectName("sql_table_name")



        self.tab_4 = QtWidgets.QWidget()
        self.tab_4.setObjectName("tab_4")
        self.convertExcelButton = QtWidgets.QPushButton(self.tab_4)
        self.convertExcelButton.setGeometry(QtCore.QRect(90, 500, 141, 41))
        self.convertExcelButton.setObjectName("convertExcelButton")
        self.label_17 = QtWidgets.QLabel(self.tab_4)
        self.label_17.setGeometry(QtCore.QRect(10, 180, 81, 16))
        self.label_17.setObjectName("label_17")
        self.ex_columnConvertMap = QtWidgets.QTextEdit(self.tab_4)
        self.ex_columnConvertMap.setGeometry(QtCore.QRect(90, 310, 531, 181))
        self.ex_columnConvertMap.setObjectName("ex_columnConvertMap")
        self.ex_shapeFile = QtWidgets.QLineEdit(self.tab_4)
        self.ex_shapeFile.setGeometry(QtCore.QRect(90, 30, 531, 41))
        self.ex_shapeFile.setObjectName("ex_shapeFile")
        self.ex_chooseShapeFile = QtWidgets.QPushButton(self.tab_4)
        self.ex_chooseShapeFile.setGeometry(QtCore.QRect(670, 30, 101, 41))
        self.ex_chooseShapeFile.setObjectName("ex_chooseShapeFile")
        self.label_18 = QtWidgets.QLabel(self.tab_4)
        self.label_18.setGeometry(QtCore.QRect(10, 260, 54, 12))
        self.label_18.setObjectName("label_18")
        self.ex_chooseGeoJsonSaveFile = QtWidgets.QPushButton(self.tab_4)
        self.ex_chooseGeoJsonSaveFile.setGeometry(QtCore.QRect(670, 170, 101, 41))
        self.ex_chooseGeoJsonSaveFile.setObjectName("ex_chooseGeoJsonSaveFile")
        self.ex_choosePrjFile = QtWidgets.QPushButton(self.tab_4)
        self.ex_choosePrjFile.setGeometry(QtCore.QRect(670, 100, 101, 41))
        self.ex_choosePrjFile.setObjectName("ex_choosePrjFile")
        self.label_19 = QtWidgets.QLabel(self.tab_4)
        self.label_19.setGeometry(QtCore.QRect(20, 330, 54, 12))
        self.label_19.setObjectName("label_19")
        self.label_9 = QtWidgets.QLabel(self.tab_4)
        self.label_9.setGeometry(QtCore.QRect(20, 110, 101, 16))
        self.label_9.setObjectName("label_9")
        self.ex_prjShape = QtWidgets.QLineEdit(self.tab_4)
        self.ex_prjShape.setGeometry(QtCore.QRect(90, 100, 531, 41))
        self.ex_prjShape.setObjectName("ex_prjShape")
        self.label_10 = QtWidgets.QLabel(self.tab_4)
        self.label_10.setGeometry(QtCore.QRect(20, 40, 101, 16))
        self.label_10.setObjectName("label_10")
        self.ex_geojsonSaveFile = QtWidgets.QLineEdit(self.tab_4)
        self.ex_geojsonSaveFile.setGeometry(QtCore.QRect(90, 170, 531, 41))
        self.ex_geojsonSaveFile.setObjectName("ex_geojsonSaveFile")
        self.checkBox_6 = QtWidgets.QCheckBox(self.tab_4)
        self.checkBox_6.setGeometry(QtCore.QRect(90, 260, 221, 21))
        self.checkBox_6.setObjectName("checkBox_6")
        self.lookStructs.addTab(self.tab_4, "")
        self.tab_5 = QtWidgets.QWidget()
        self.tab_5.setObjectName("tab_5")
        self.textBrowser = QtWidgets.QTextEdit(self.tab_5)
        self.textBrowser.setGeometry(QtCore.QRect(0, 0, 801, 551))
        self.textBrowser.setObjectName("textBrowser")
        self.lookStructs.addTab(self.tab_5, "")
        MainWindow.setCentralWidget(self.centralwidget)
        self.statusbar = QtWidgets.QStatusBar(MainWindow)
        self.statusbar.setObjectName("statusbar")
        MainWindow.setStatusBar(self.statusbar)

        self.retranslateUi(MainWindow)
        self.lookStructs.setCurrentIndex(1)
        QtCore.QMetaObject.connectSlotsByName(MainWindow)

        self.lookStructs.setCurrentIndex(0)
        self.lookStructs.currentChanged.connect(self.tab_change)

        self.choosePrjFile.clicked.connect(self.choose_prj_file)
        self.choosePrjFile_2.clicked.connect(self.choose_prj_file)
        self.chooseJSPrjFile.clicked.connect(self.choose_prj_file)
        self.sql_choosePrjFile.clicked.connect(self.choose_prj_file)
        self.ex_choosePrjFile.clicked.connect(self.choose_prj_file)

        self.chooseShapeFile.clicked.connect(self.choose_shp_file)
        self.chooseShapeFile_2.clicked.connect(self.choose_shp_file)
        self.chooseJSShapeFile.clicked.connect(self.choose_shp_file)
        self.sql_chooseShapeFile.clicked.connect(self.choose_shp_file)
        self.ex_chooseShapeFile.clicked.connect(self.choose_shp_file)

        self.chooseGeoJsonSaveFile.clicked.connect(self.choose_save_folder)
        self.sqlchooseGeoJsonSaveFile.clicked.connect(self.choose_save_folder)
        self.ex_chooseGeoJsonSaveFile.clicked.connect(self.choose_save_folder)

        self.parseDataButton.clicked.connect(self.get_data_structs)
        self.parseDataButton_2.clicked.connect(self.get_data_image)
        self.convertJSButton.clicked.connect(self.get_data_geoJSON)
        self.convertSQLButton.clicked.connect(self.get_data_sql)
        self.convertExcelButton.clicked.connect(self.get_data_excel)

        self.zoomout.clicked.connect(self.on_zoomout_clicked)
        self.zoomin.clicked.connect(self.on_zoomin_clicked)


    def retranslateUi(self, MainWindow):
        _translate = QtCore.QCoreApplication.translate
        MainWindow.setWindowTitle(_translate("MainWindow", "Shape工具"))
        self.label.setText(_translate("MainWindow", "Shape文件:"))
        self.label_2.setText(_translate("MainWindow", "Prj文件:"))
        self.chooseShapeFile.setText(_translate("MainWindow", "选择文件"))
        self.choosePrjFile.setText(_translate("MainWindow", "选择文件"))
        self.label_4.setText(_translate("MainWindow", "数据结构"))
        self.parseDataButton.setText(_translate("MainWindow", "开始解析"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab), _translate("MainWindow", "查看数据结构"))
        self.chooseShapeFile_2.setText(_translate("MainWindow", "选择文件"))
        self.choosePrjFile_2.setText(_translate("MainWindow", "选择文件"))
        self.label_20.setText(_translate("MainWindow", "Prj文件:"))
        self.parseDataButton_2.setText(_translate("MainWindow", "开始解析"))
        self.label_21.setText(_translate("MainWindow", "Shape文件:"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab_6), _translate("MainWindow", "查看图形分布"))
        self.convertJSButton.setText(_translate("MainWindow", "开始转换"))
        self.chooseJSShapeFile.setText(_translate("MainWindow", "选择文件"))
        self.label_5.setText(_translate("MainWindow", "Shape文件:"))
        self.label_6.setText(_translate("MainWindow", "Prj文件:"))
        self.chooseJSPrjFile.setText(_translate("MainWindow", "选择文件"))
        self.label_11.setText(_translate("MainWindow", "文件保存位置"))
        self.chooseGeoJsonSaveFile.setText(_translate("MainWindow", "选择文件夹"))
        self.label_12.setText(_translate("MainWindow", "转换选项"))
        self.label_13.setText(_translate("MainWindow", "字段映射"))
        self.label_111.setText(_translate("MainWindow", "地理信息"))
        self.label_222.setText(_translate("MainWindow", "地理信息字段:"))
        self.label_333.setText(_translate("MainWindow", "文本格式地理信息字段:"))
        self.label_444.setText(_translate("MainWindow", "表名:"))
        self.checkBox.setText(_translate("MainWindow", "是否根据Prj文件进行经纬度转换"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab_2), _translate("MainWindow", "转GEOJSON"))
        self.convertSQLButton.setText(_translate("MainWindow", "开始转换"))
        self.label_14.setText(_translate("MainWindow", "文件保存位置"))
        self.sql_chooseShapeFile.setText(_translate("MainWindow", "选择文件"))
        self.label_15.setText(_translate("MainWindow", "转换选项"))
        self.sqlchooseGeoJsonSaveFile.setText(_translate("MainWindow", "选择文件夹"))
        self.sql_choosePrjFile.setText(_translate("MainWindow", "选择文件"))
        self.label_16.setText(_translate("MainWindow", "字段映射"))
        self.label_7.setText(_translate("MainWindow", "Prj文件:"))
        self.label_8.setText(_translate("MainWindow", "Shape文件:"))
        self.checkBox_3.setText(_translate("MainWindow", "是否根据Prj文件进行经纬度转换"))
        self.checkBox_4.setText(_translate("MainWindow", "是否转为Muiliti数据集"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab_3), _translate("MainWindow", "转SQL"))
        self.convertExcelButton.setText(_translate("MainWindow", "开始转换"))
        self.label_17.setText(_translate("MainWindow", "文件保存位置"))
        self.ex_chooseShapeFile.setText(_translate("MainWindow", "选择文件"))
        self.label_18.setText(_translate("MainWindow", "转换选项"))
        self.ex_chooseGeoJsonSaveFile.setText(_translate("MainWindow", "选择文件夹"))
        self.ex_choosePrjFile.setText(_translate("MainWindow", "选择文件"))
        self.label_19.setText(_translate("MainWindow", "字段映射"))
        self.label_9.setText(_translate("MainWindow", "Prj文件:"))
        self.label_10.setText(_translate("MainWindow", "Shape文件:"))
        self.checkBox_6.setText(_translate("MainWindow", "是否根据Prj文件进行经纬度转换"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab_4), _translate("MainWindow", "转EXCEL"))
        self.lookStructs.setTabText(self.lookStructs.indexOf(self.tab_5), _translate("MainWindow", "使用说明"))
        self.zoomout.setText(_translate("MainWindow", "放大"))
        self.zoomin.setText(_translate("MainWindow", "缩小"))
        self.sql_coord_name.setText("coordinate")

        cc = """
                                              使用说明手册
        一、查看数据结构：
            选择后缀名为.shp的文件。
            选择后缀名为.prj的文件。
            点击转换，等待转换结束，即可查看到前10条数据。

         
        二、查看图形分布
            选择后缀名为.shp的文件。
            选择后缀名为.prj的文件。
            点击转换，等待转换结束，即可看见图形分布。
            点击 放大，可以放大图形。
            点击 缩小，可以缩小图形。
            
        三、转GEOJSON
            选择后缀名为.shp的文件。
            选择后缀名为.prj的文件。
            选择保存生成的GEOJSON的文件夹。
            转换选项：如果勾选，则将会根据prj文件中的信息将坐标转为WGS84，如不选则保存原数据。
            字段映射：如果原始shape文件中字段名称为 汉字等，可以再次提供一个字段映射的map。在生成sql文件的时候会进行字段的相应转换。

                {
                    "原始字段名1":"目标字段名1",
                    "原始字段名2":"目标字段名2"
                }

            点击转换，等待转换结束，即可在选择的保存文件夹下生成geometry.json文件。

        四、转SQL
            选择后缀名为.shp的文件。
            选择后缀名为.prj的文件。
            选择保存生成的GEOJSON的文件夹。
            转换选项：
                1.如果勾选，则将会根据prj文件中的信息将坐标转为WGS84，如不选则保存原数据。
                2.如果勾选，则会将POLYGON格式转换为MULTIPOLYGON格式。
                
            地理信息：
                地理信息字段：数据库存储地理信息的字段，类型应该为Point、Polygon、multipolygon等类型，必填，	MULTIPOLYGON(((120.15168627949 30.2256835231913)))
                文本格式地理信息字段：数据库存储地理信息文本格式的字段，[[120.1516862794896, 30.22568352319127]]
            字段映射：如果原始shape文件中字段名称为 汉字等，可以再次提供一个字段映射的map。在生成sql文件的时候会进行字段的相应转换。

                {
                    "原始字段名1":"目标字段名1",
                    "原始字段名2":"目标字段名2"
                }

            点击转换，等待转换结束，即可在选择的保存文件夹下生成geometry.sql文件。

        五、转EXCEL
            选择后缀名为.shp的文件。
            选择后缀名为.prj的文件。
            选择保存生成的EXCEL的文件夹。
            转换选项：如果勾选，则将会根据prj文件中的信息将坐标转为WGS84，如不选则保存原数据。
            字段映射：如果原始shape文件中字段名称为 汉字等，可以再次提供一个字段映射的map。在生成EXCEL文件的时候会进行字段的相应转换。

                {
                    "原始字段名1":"目标字段名1",
                    "原始字段名2":"目标字段名2"
                }

            点击转换，等待转换结束，即可在选择的保存文件夹下生成geometry.excel文件。

            """

        self.textBrowser.setMarkdown(cc)

    def tab_change(self, x):
        global curr_tab_index
        curr_tab_index = x

    def messageDialog(self, msg):
        # 核心功能代码就两行，可以加到需要的地方
        if hasattr(self, "loading_mask"):
            self.loading_mask.deleteLater()
        msg_box = QMessageBox.about(self.centralwidget, '提示', msg)

    def choose_prj_file(self):
        # 打开文件路径
        file_name, file_type = QFileDialog.getOpenFileName(self.centralwidget, "选取.prj文件", os.getcwd(), "*.prj")
        global curr_tab_index
        if curr_tab_index == 0:
            self.prjShape.setText(file_name)
        elif curr_tab_index == 1:
            self.prjShape_2.setText(file_name)
        elif curr_tab_index == 2:
            self.js_prjShape.setText(file_name)
        elif curr_tab_index == 3:
            self.sql_prjShape.setText(file_name)
        else:
            self.ex_prjShape.setText(file_name)

    def choose_shp_file(self):
        # 打开文件路径
        file_name, file_type = QFileDialog.getOpenFileName(self.centralwidget, "选取.shp文件", os.getcwd(), "*.shp")
        global curr_tab_index
        if curr_tab_index == 0:
            self.shapeFile.setText(file_name)
        elif curr_tab_index == 1:
            self.shapeFile_2.setText(file_name)
        elif curr_tab_index == 2:
            self.js_shapeFile.setText(file_name)
        elif curr_tab_index == 3:
            self.sql_shapeFile.setText(file_name)
        else:
            self.ex_shapeFile.setText(file_name)

    def choose_save_folder(self):
        # 打开文件路径
        folder_name = QFileDialog.getExistingDirectory(self.centralwidget, "选取输出文件夹", os.getcwd())
        global curr_tab_index
        if curr_tab_index == 2:
            self.geojsonSaveFile.setText(folder_name)
        elif curr_tab_index == 3:
            self.sqlSaveFile.setText(folder_name)
        else:
            self.ex_geojsonSaveFile.setText(folder_name)

    def get_data_structs(self):
        try:
            shp_file = self.shapeFile.text()
            if shp_file is None or shp_file == '':
                self.messageDialog("shape文件没有选择")
                return

            self.loading_mask = LoadingMask(MainWindow, tip='正在解析...', gif=get_resource_path('./loading.gif'))
            MainWindow.installEventFilter(self.loading_mask)
            self.loading_mask.show()
            self.loading_mask.moveWithParent()

            # 创建线程
            self.thread = ReadShapeStructs(shp_file, {})
            # 连接信号
            self.thread._signal.connect(self.draw_structs)  # 进程连接回传到GUI的事件
            # 开始线程
            self.thread.start()
        except:
            self.messageDialog("转换失败")

    def get_data_image(self):
        try:
            shp_file = self.shapeFile_2.text()
            if shp_file is None or shp_file == '':
                self.messageDialog("shape文件没有选择")
                return

            self.loading_mask = LoadingMask(MainWindow, tip='正在解析...', gif=get_resource_path('./loading.gif'))
            MainWindow.installEventFilter(self.loading_mask)
            self.loading_mask.show()
            self.loading_mask.moveWithParent()

            # 创建线程
            self.thread = ReadShapeImage(shp_file, {})
            # 连接信号
            self.thread._signal.connect(self.draw_image)  # 进程连接回传到GUI的事件
            # 开始线程
            self.thread.start()
        except:
            self.messageDialog("转换失败")

    def get_data_geoJSON(self):
        try:
            shp_file = self.js_shapeFile.text()
            prj_file = self.js_prjShape.text()
            save_folder = self.geojsonSaveFile.text()
            column_map = self.columnConvertMap.toPlainText()
            cov_ll = self.checkBox.checkState()
            if shp_file is None or shp_file == '':
                self.messageDialog("shape文件没有选择")
                return
            if cov_ll == 2 and (prj_file is None or prj_file == ''):
                self.messageDialog("prj文件没有选择")
                return
            if save_folder is None or save_folder == '':
                self.messageDialog("文件保存地址没有选择")
                return

            self.loading_mask = LoadingMask(MainWindow, tip='正在转换...', gif=get_resource_path('./loading.gif'))
            MainWindow.installEventFilter(self.loading_mask)
            self.loading_mask.show()
            self.loading_mask.moveWithParent()

            if column_map is None or len(column_map) <= 0:
                column_map = '1222'
            # 创建线程
            self.thread = ConvertGeojson(shp_file, prj_file, save_folder, json.loads(column_map), cov_ll)
            # 连接信号
            self.thread._signal.connect(self.messageDialog)  # 进程连接回传到GUI的事件
            # 开始线程
            self.thread.start()
        except:
            self.messageDialog("转换失败")

    def get_data_sql(self):
        try:
            sql_table_name = self.sql_table_name.text()
            sql_coord_name = self.sql_coord_name.text()
            sql_coord_text_name = self.sql_coord_text_name.text()
            shp_file = self.sql_shapeFile.text()
            prj_file = self.sql_prjShape.text()
            save_folder = self.sqlSaveFile.text()
            column_map = self.sql_columnConvertMap.toPlainText()
            cov_ll = self.checkBox_3.checkState()
            cov_muiliti = self.checkBox_4.checkState()
            if shp_file is None or shp_file == '':
                self.messageDialog("shape文件没有选择")
                return
            if cov_ll == 2 and (prj_file is None or prj_file == ''):
                self.messageDialog("prj文件没有选择")
                return
            if save_folder is None or save_folder == '':
                self.messageDialog("文件保存地址没有选择")
                return
            if sql_coord_name is None or sql_coord_name == '':
                self.messageDialog("地理信息字段不能为空")
                return

            self.loading_mask = LoadingMask(MainWindow, tip='正在转换...', gif=get_resource_path('./loading.gif'))
            MainWindow.installEventFilter(self.loading_mask)
            self.loading_mask.show()
            self.loading_mask.moveWithParent()

            # 创建线程
            if column_map is None or len(column_map) <= 0:
                column_map = '1222'
            self.thread = ConvertSQL(shp_file, prj_file, save_folder, json.loads(column_map), cov_ll, cov_muiliti,sql_coord_name,sql_coord_text_name,sql_table_name)
            # 连接信号
            self.thread._signal.connect(self.messageDialog)  # 进程连接回传到GUI的事件
            # 开始线程
            self.thread.start()
        except:
            self.messageDialog("转换失败")


    def draw_structs(self, df):
        try:
            columnname = list(df)[0:-2]
            self.dataStructsTable.setColumnCount(len(columnname))
            self.dataStructsTable.setRowCount(min(10, len(df)))
            self.dataStructsTable.setEditTriggers(QAbstractItemView.NoEditTriggers)
            self.dataStructsTable.setHorizontalHeaderLabels(columnname)
            for i in range(min(10, len(df))):
                data = list(df.loc[i].values)
                for j in range(len(columnname)):
                    self.dataStructsTable.setItem(i, j, QTableWidgetItem(str(data[j] if data[j] is not None else '')))

            self.dataStructsTable.show()
            self.loading_mask.deleteLater()
        except:
            self.messageDialog("转换失败")

    def draw_image(self):
        try:
            self.loading_mask.deleteLater()
            img = cv2.imread("geometry.png")  # 读取图像
            #img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)  # 转换图像通道
            x = img.shape[1]  # 获取图像大小
            y = img.shape[0]
            self.zoomscale = 1  # 图片放缩尺度
            frame = QImage(img, x, y, QImage.Format_RGB888)
            pix = QPixmap.fromImage(frame)
            self.item = QGraphicsPixmapItem(pix)  # 创建像素图元
            self.scene = QGraphicsScene()  # 创建场景
            self.scene.addItem(self.item)
            self.graphicsView.setScene(self.scene)
        except:
            self.messageDialog("转换失败")

    def on_zoomin_clicked(self):
        """
        点击缩小图像
        """
        # TODO: not implemented yet
        try:
            if hasattr(self, 'zoomscale'):
                self.zoomscale = self.zoomscale - 0.05
                if self.zoomscale <= 0:
                    self.zoomscale = 0.2
                self.item.setScale(self.zoomscale)  # 缩小图像
        except:
            self.messageDialog("缩放失败")

    def on_zoomout_clicked(self):
        """
        点击方法图像
        """
        # TODO: not implemented yet
        try:
            if hasattr(self, 'zoomscale'):
                self.zoomscale = self.zoomscale + 0.05
                if self.zoomscale >= 1.2:
                    self.zoomscale = 1.2
                self.item.setScale(self.zoomscale)  # 放大图像
        except:
            self.messageDialog("缩放失败")


    def get_data_excel(self):
        try:
            shp_file = self.ex_shapeFile.text()
            prj_file = self.ex_prjShape.text()
            save_folder = self.ex_geojsonSaveFile.text()
            column_map = self.ex_columnConvertMap.toPlainText()
            cov_ll = self.checkBox_6.checkState()
            # cov_muiliti = self.checkBox_7.checkState()
            if shp_file is None or shp_file == '':
                self.messageDialog("shape文件没有选择")
                return
            if cov_ll == 2 and (prj_file is None or prj_file == ''):
                self.messageDialog("prj文件没有选择")
                return
            if save_folder is None or save_folder == '':
                self.messageDialog("文件保存地址没有选择")
                return

            self.loading_mask = LoadingMask(MainWindow, tip='正在转换...', gif=get_resource_path('./loading.gif'))
            MainWindow.installEventFilter(self.loading_mask)
            self.loading_mask.show()
            self.loading_mask.moveWithParent()
            # 创建线程
            if column_map is None or len(column_map) <= 0:
                column_map = '1233'
            self.thread = ConvertExcel(shp_file, prj_file, save_folder, json.loads(column_map), cov_ll)
            # 连接信号
            self.thread._signal.connect(self.messageDialog)  # 进程连接回传到GUI的事件
            # 开始线程
            self.thread.start()
        except:
            self.messageDialog("转换失败")

    def draw_structs(self, df):
        try:
            columnname = list(df)[0:-2]
            self.dataStructsTable.setColumnCount(len(columnname))
            self.dataStructsTable.setRowCount(min(10, len(df)))
            self.dataStructsTable.setEditTriggers(QAbstractItemView.NoEditTriggers)
            self.dataStructsTable.setHorizontalHeaderLabels(columnname)
            for i in range(min(10, len(df))):
                data = list(df.loc[i].values)
                for j in range(len(columnname)):
                    self.dataStructsTable.setItem(i, j, QTableWidgetItem(str(data[j] if data[j] is not None else '')))

            self.dataStructsTable.show()
            self.loading_mask.deleteLater()
        except:
            self.messageDialog("转换失败")


def get_resource_path(relative_path):
    if getattr(sys, 'frozen', False) and hasattr(sys, '_MEIPASS'):
        return os.path.join(sys._MEIPASS, relative_path)
    return os.path.join(os.path.abspath("."), relative_path)

# 继承QThread
class ReadShapeStructs(QThread):  # 线程1
    # 通过类成员对象定义信号对象
    _signal = pyqtSignal(GeoDataFrame)

    def __init__(self, file: str, column_name_map: dict):
        super().__init__()
        self.file = file
        self.column_name_map = column_name_map

    def run(self):
        qmut_1.lock()  # 加锁
        df2 = read_file(self.file, encoding="gbk")
        if self.column_name_map is not None:
            df2 = df2.rename(columns=self.column_name_map)
        self._signal.emit(df2)
        qmut_1.unlock()  # 解锁#


        # 继承QThread
class ReadShapeImage(QThread):  # 线程1
    # 通过类成员对象定义信号对象
    _signal = pyqtSignal(GeoDataFrame)

    def __init__(self, file: str, column_name_map: dict):
        super().__init__()
        self.file = file
        self.column_name_map = column_name_map

    def run(self):
        qmut_1.lock()  # 加锁
        df2 = read_file(self.file, encoding="gbk")
        df2.plot(color='red')
        pyplot.savefig('geometry.png', dpi=1000)
        self._signal.emit(df2)
        qmut_1.unlock()  # 解锁

# 继承QThread
class ConvertGeojson(QThread):  # 线程1
    # 通过类成员对象定义信号对象
    _signal = pyqtSignal(str)

    def __init__(self, shp: str, prj: str, save_folder: str, column_name_map: dict, cov_ll: int):
        super().__init__()
        self.shp = shp
        self.prj = prj
        self.save_folder = save_folder
        self.cov_ll = cov_ll
        self.column_name_map = column_name_map

    def run(self):
        qmut_1.lock()  # 加锁
        if self.cov_ll == 2:
            crs = self.read_crs_from_prj_file(self.prj)
            data = read_file(self.shp, encoding="gbk")
            data.to_crs(crs=crs)
            df = data.to_crs(crs="+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")
        else:
            df = read_file(self.shp, encoding="gbk")

        if self.column_name_map is not None and type(self.column_name_map) == dict:
            df = df.rename(columns=self.column_name_map)
        outpoint_f = 'geometry.json'
        df.to_file(self.save_folder + '/' + outpoint_f, driver='GeoJSON', encoding='utf-8')
        self._signal.emit('GeoJson文件生成成功，文件地址：' + self.save_folder + '/' + outpoint_f)
        qmut_1.unlock()  # 解锁

    def read_crs_from_prj_file(self, prjfile: str):
        """
        读取prj文件，获取坐标
        :param prjfile: prj文件地址
        :return: prj文件内容
        """
        f = open(prjfile, "r")
        result = f.read().encode("utf-8")
        f.close()
        return str(result, encoding='utf-8')


class Figure_Bar(FigureCanvasQTAgg):

    def __init__(self, width, height, parent=None):
        self.fig = pyplot.figure(figsize=(width, height), facecolor='#666666', dpi=100, edgecolor='#0000FF')
        FigureCanvasQTAgg.__init__(self, self.fig)
        self.setParent(parent)

        self.myAxes = self.fig.add_subplot(111)

        self.x = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9]

    def ShowImage(self, axisX, axisY):
        # 由于图片需要反复绘制，所以每次绘制前清空，然后绘图，最后刷新显示。
        self.myAxes.clear()
        self.myAxes.bar(self.x, axisY, width=0.02, color='blue', align='center', tick_label=axisX)
        # 刷新画布，否则不刷新显示
        self.fig.canvas.draw()

    def ShowImage1(self, df):
        # 由于图片需要反复绘制，所以每次绘制前清空，然后绘图，最后刷新显示。
        self.myAxes.clear()
        self.myAxes.plot(df)
        df.plot(color='red')
        # 刷新画布，否则不刷新显示
        self.fig.canvas.draw(pyplot.show())


    # 继承QThread
class ConvertExcel(QThread):  # 线程1
    # 通过类成员对象定义信号对象
    _signal = pyqtSignal(str)

    def __init__(self, shp: str, prj: str, save_folder: str, column_name_map: dict, cov_ll: int):
        super().__init__()
        self.shp = shp
        self.prj = prj
        self.save_folder = save_folder
        self.cov_ll = cov_ll
        self.column_name_map = column_name_map

    def run(self):
        qmut_1.lock()  # 加锁
        if self.cov_ll == 2:
            crs = self.read_crs_from_prj_file(self.prj)
            data = read_file(self.shp, encoding="gbk")
            data.to_crs(crs=crs)
            df = data.to_crs(crs="+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")
        else:
            df = read_file(self.shp, encoding="gbk")
        if self.column_name_map is not None and type(self.column_name_map) == dict:
            df = df.rename(columns=self.column_name_map)
        columnname = list(df)
        df.to_excel(self.save_folder + '/' + 'geometry.xlsx', index=None, header=columnname)
        self._signal.emit('excel文件生成成功，文件地址：' + self.save_folder + '/' + 'geometry.xlsx')
        qmut_1.unlock()  # 解锁

    def read_crs_from_prj_file(self, prjfile: str):
        """
        读取prj文件，获取坐标
        :param prjfile: prj文件地址
        :return: prj文件内容
        """
        f = open(prjfile, "r")
        result = f.read().encode("utf-8")
        f.close()
        return str(result, encoding='utf-8')

# 继承QThread
class ConvertSQL(QThread):  # 线程1
    # 通过类成员对象定义信号对象
    _signal = pyqtSignal(str)

    def __init__(self, shp: str, prj: str, save_folder: str, column_name_map: dict, cov_ll: int, cov_muiliti: int, sql_coord_name: str, sql_coord_text_name: str,sql_table_name:str):
        super().__init__()
        self.shp = shp
        self.prj = prj
        self.save_folder = save_folder
        self.cov_ll = cov_ll
        self.cov_muiliti = cov_muiliti
        self.column_name_map = column_name_map
        self.sql_coord_name = sql_coord_name
        self.sql_coord_text_name = sql_coord_text_name
        self.sql_table_name = sql_table_name

    def run(self):
        qmut_1.lock()  # 加锁
        if self.cov_ll == 2:
            crs = self.read_crs_from_prj_file(self.prj)
            data = read_file(self.shp, encoding="gbk")
            data.to_crs(crs=crs)
            df = data.to_crs(crs="+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")
        else:
            df = read_file(self.shp, encoding="gbk")

        if self.column_name_map is not None and type(self.column_name_map) == dict:
            df = df.rename(columns=self.column_name_map)
        outpoint_f = 'geometry.json'
        df.to_file(self.save_folder + '/' + outpoint_f, driver='GeoJSON', encoding='utf-8')

        with open(self.save_folder + '/' + outpoint_f, 'r', encoding='utf8')as fp:
            json_data = json.load(fp)
            datass = json_data['features']
            values_list = []
            for a in datass:
                b = a
                value = b['properties']
                columns_list = []
                for x in value:
                    columns_list.append(x)
                values = {}
                for key in columns_list:
                    vv = value.get(key)
                    if vv is None:
                        vv = ''
                    values[key] = vv
                value2 = b['geometry']

                corrd = str(value2['coordinates'][0])
                if self.sql_coord_text_name is not None and self.sql_coord_text_name != '':
                    values[self.sql_coord_text_name] = corrd

                if self.sql_coord_name is not None and self.sql_coord_name != '':
                    wkt = convert.geojson_to_wkt(value2)
                    if self.cov_muiliti == 2 and wkt.startswith("POLYGON"):
                        wkt = wkt.replace("POLYGON", "MULTIPOLYGON (")
                        wkt = wkt + ")"
                    values[self.sql_coord_name] = wkt
                    values_list.append(values)

        sql_list = []
        for data in values_list:
            keys = []
            values = []
            for dd in data:
                keys.append('`' + dd + '`')
                if dd == self.sql_coord_name:
                    values.append("ST_GeomFromText('" + str(data[dd]) + "')")
                else:
                    values.append("'" + str(data[dd]) + "'")

            sql_list.append("insert into {}({}) values ({})".format(self.sql_table_name, ','.join(keys), ','.join(values)))
        with open(self.save_folder + '/geometry.sql', "w+", encoding='utf-8') as f:
            f.write(";\n".join(sql_list))
        self._signal.emit('sql文件生成成功，文件地址：' + self.save_folder + '/geometry.sql')
        qmut_1.unlock()  # 解锁

    def read_crs_from_prj_file(self, prjfile: str):
        """
        读取prj文件，获取坐标
        :param prjfile: prj文件地址
        :return: prj文件内容
        """
        f = open(prjfile, "r")
        result = f.read().encode("utf-8")
        f.close()
        return str(result, encoding='utf-8')


class LoadingMask(QMainWindow):

    def __init__(self, parent, gif=None, tip=None):
        super(LoadingMask, self).__init__(parent)

        parent.installEventFilter(self)

        self.label = QLabel()

        if not tip is None:
            self.label.setText(tip)
            font = QFont('Microsoft YaHei', 10, QFont.Normal)
            font_metrics = QFontMetrics(font)
            self.label.setFont(font)
            self.label.setFixedSize(font_metrics.width(tip, len(tip)) + 10, font_metrics.height() + 5)
            self.label.setAlignment(Qt.AlignCenter)
            self.label.setStyleSheet(
                'QLabel{background-color: rgba(0,0,0,70%);border-radius: 4px; color: white; padding: 5px;}')

        if not gif is None:
            self.movie = QMovie(gif)
            self.label.setMovie(self.movie)
            self.label.setFixedSize(QSize(160, 160))
            self.label.setScaledContents(True)
            self.movie.start()

        layout = QHBoxLayout()
        widget = QWidget()
        widget.setFixedSize(800, 600)
        widget.setLayout(layout)
        layout.addWidget(self.label)

        self.setCentralWidget(widget)
        self.setWindowOpacity(0.8)
        self.setWindowFlags(Qt.FramelessWindowHint | Qt.Dialog)
        self.hide()

    def eventFilter(self, widget, event):
        if widget == self.parent() and type(event) == QMoveEvent:
            self.moveWithParent()
            return True
        return super(LoadingMask, self).eventFilter(widget, event)

    def moveWithParent(self):
        if self.isVisible():
            self.move(self.parent().geometry().x(), self.parent().geometry().y())
            self.setFixedSize(QSize(self.parent().geometry().width(), self.parent().geometry().height()))


if __name__ == '__main__':
    app = QApplication(sys.argv)
    MainWindow = QMainWindow()
    ui = Ui_MainWindow()
    ui.setupUi(MainWindow)
    MainWindow.show()
    sys.exit(app.exec_())
```

## 六、相关文件下载：
**命令行解析文件下载：**[shape-file-parse.py](https://www.yuque.com/attachments/yuque/0/2022/txt/167378/1648102801433-ff308d4a-7e5f-4bfc-b7c4-01d1866717af.txt)

**工具源码下载：**[shape-parse-utils.py](https://www.yuque.com/attachments/yuque/0/2022/txt/167378/1648102854224-a906df86-258a-411d-ab4b-c3f5381a23f9.txt)

