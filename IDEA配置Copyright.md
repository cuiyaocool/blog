## 设置路径
```
File > Settings > Editor > Copyright > Copyright Profiles
```
## 具体操作
点击+新增模板   
系统给出一个模板：
```
Copyright (c) $today.year. Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
Morbi non lorem porttitor neque feugiat blandit. Ut vitae ipsum eget quam lacinia accumsan. 
Etiam sed turpis ac ipsum condimentum fringilla. Maecenas magna. 
Proin dapibus sapien vel ante. Aliquam erat volutpat. Pellentesque sagittis ligula eget metus. 
Vestibulum commodo. Ut rhoncus gravida arcu. 
```
## 可用的变量
可以自定义修改，如上**$today.year**是一个变量，类似的还有：
|Name|Type|Comment|
|---|---|---|
|$today	|DateInfo|	当前日期时间对象|
|$file.fileName	|String|	当前文件的名称|
|$file.pathName	|String|	当前文件的完整路径|
|$file.className	|String|	当前文件的类名|
|$file.qualifiedClassName	|String|	当前文件的权限定名|
|$file.lastModified	|DateInfo|	上一次修改的日期时间对象|
|$project.name	|String|	当前项目名|
|$module.name	|String|	当前 Module 名|
|$username	|String|	当前用户名（系统用户名）|


其中，DateInfo 对象有如下属性和方法：

|Name|	Type|	Comment|
|-|-|-|
|year|	int|	当前年份|
|month|	int|	当前月份|
|day|	int|	当前日期（1-31）|
|hour|	int|	当前小时（0-11）|
|hour24|	int|	当前小时（0-23）|
|minute|	int|	当前分钟（0-59）|
|second|	int|	当前秒数（0-59）|
|format(String format)|	String|	时间日期格式化方法，参考:java.text.SimpleDateFormat format|

## 举个栗子
```
 @(#)$file.fileName, $today.format("yyyy-MM-dd")

 Copyright $today.year  XXX, Inc. All rights reserved.
 XXX PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
```
## 实验展示  
在需要添加版权信息的文件中，alt+insert，选择copyright即可，上面的例子添加后的结果是：
```
/*
 *  @(#)ClassName.java, 2020-04-08
 *
 *  Copyright 2020  XXX, Inc. All rights reserved.
 *  XXX PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
 */
```
