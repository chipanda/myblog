---
title: Kibana基础使用
tags:
  - Kibana
categories:
  - Other
date: 2018-03-01 15:13:32
---


## 目录

* 一、Discover（发现）
    * Apache Lucene
    * filter
    * 时间过滤器
* 二、Visualize（可视化）
    * Metric（度量）
    * Pie（饼图）
    * Vertical Bar（柱状图）
    * Data Table（表格）
* 三、Dashboard（仪表盘）及其它
    * Dashboard
    * 非索引字段
    * 资料
<!-- more -->
*******

## 一、Discover（发现）

通过指定检索条件查询符合的日志记录，常用于日志所含信息的分析

### 1、Apache Lucene

查询可分解为术语和运算符。 有两种类型的术语：单一术语和短语。

单一术语："app.xxx.show_fail"
短语： "gen fail", 注意使用双引号，单引号会被识别为两个单一术语gen和fail

1. 字段查询

    entry_key:"app.xxx.show_fail" 或 "app.xxx.show_fail" 或 entry_key:app.xxx.show_fail

2. 通配符

    ？单个 app.xxx.show_fa?l        
    \* 多个 app.xxx.show_f*

3. 范围查询

    闭区间：[]   
    开区间：{}  
    package_info.version:[0 TO 271} 

4. boolean运算符

    `AND NOT OR + -`     
    +表示其后面的术语必须存在而不等于AND
    app.xxx.show_fail app.xxx.show_success 等价于 app.xxx.show_fail OR app.xxx.show_success

5. 分组

    `()`        
    (app.xxx.show_success OR app.xxx.show_fail) AND package_info.version:[0 TO 271}

6. 需转义的字符

    `+ - && || ! ( ) { } [ ] ^ " ~ * ? : \`

### 2、filter    

对指定的索引过滤，包含is、is not、is one of、is not one of、exist、do not exist

### 3、时间过滤器

quick、relative、absolute

*******

## 二、Visualize（可视化）

通过指定检索条件统计符合的日志记录，常用于日志数量分布的统计

常用的数量统计类型：

* Count，统计个数
* Unique Count，根据指定字段如userid，去重统计个数
* Average，值为数值类型的索引，可对其计算平均值

常用的分组统计类型：

* Date Histogram，按时间对数据分组，可指定时间间隔
* Range，对数值类型的索引按区间分组
* Terms，某个索引的值本身是有限且有意义的（如城市、错误类型），可排序，并选择指定数量的分组
* Filter，每一个filter作为一个分组

### 1、Metric（度量）

一般用于统计某个指标的个数

### 2、Pie（饼图）

一般用于统计某类指标中各项指标的分布情况，可自动得出百分比、个数。

1. 通过query和metrics过滤基础指标
2. 添加分组条件，将饼图分割
3. 添加多个分组条件，在前一个分组的每组数据下继续分组统计数据

### 3、Vertical Bar（柱状图）

一般用于统计某类指标在二维情况下的分布情况，x轴通常是时间线或城市等可区分的一条维度，y轴通常为各项指标的个数

1. 通过query查询出待分析数据
2. 指定y轴、x轴
3. 在x轴上的单项可分组
4. 通过“Metric && Axes”调整展示

### 4、Data Table（表格）

1. 通过query查询出待分析数据
2. 添加metrics，指定统计的列数据
3. Split Rows，指定行的分组条件

### 5、Coordinate Map（坐标图，热力图）

1. 通过query和metrics过滤基础指标
2. 选择定位所用的坐标字段

*******

## 三、Dashboard（仪表盘）及其它

### 1、Dashboard

Dashboard的内容由Discover和Visualize组成

### 2、非索引字段

* 已经加入的索引才可以进行对其值计算或分组统计
* entry_detail因为内部字段的值类型不确定，被统一转为string类型，无法深入解析索引，可以使用filter加字符串搜索手动分组

### 3、资料

* [Apache Lucene](http://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
* [Kibana英文](https://www.elastic.co/guide/en/kibana/5.5/tutorial-visualizing.html)
* [Kibana中文](http://cwiki.apachecn.org/pages/viewpage.action?pageId=8159377#app-switcher)
