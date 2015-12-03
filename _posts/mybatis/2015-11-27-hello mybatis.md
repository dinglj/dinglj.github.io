---
layout : post
title : "hello mybatis"
category : mybatis
duoshuo: true
date : 2015-11-27
tags : [mybatis]
SyntaxHihglighter: true
shTheme: shThemeEclipse # shThemeDefault  shThemeDjango  shThemeEclipse  shThemeEmacs  shThemeFadeToGrey  shThemeMidnight  shThemeRDark

---
#mybatis 笔记

---
### mapper xml 字符串比较 ###
>错误写法：if test="status == 'Y'"  
结果：抛异常NumberFormatException异常！提示内容非常少，看不出问题在哪里！  
正确写法：if test='status == "y"'  
还可以这样写：if test="status == 'y'.toString()"  
这明显单引号是指字符串，从逻辑上没有理由不支持第一种写法？这样的设计这是操蛋！浪费人时间！
记录下来，供人家参考！
---

