---
title: python json模块
tags: python
categories: python
---
>python json模块

<!--more-->

# python json模块

#### json简介

JSON：**JavaScript Object Notation** 【JavaScript 对象表示法】

JSON 是一种轻量级的数据交换格式，完全独立于任何程序语言的文本格式。一般，后台应用程序将响应数据封装成JSON格式返回。

**JSON的基本语法如下:**

JSON名称/值对。JSON 数据的书写格式是：名称/值对。名称/值对包括字段名称（在双引号中），然后着是一个冒号(:)，最后是值。

**JSON最常用的格式是对象的键值对：**

key只能是string, value可以是 object、array、string、number、true/false、null

```json
{
"sites": [
{ "name":"360" , "url":"www.360.com" }, 
{ "name":"google" , "url":"www.google.com" }, 
{ "name":"baidu" , "url":"www.baidu.com" }
]
}
```

- 键通过双引号包裹，后面跟冒号“：”，然后跟该键的值；
- 值可以是字符串、数字、数组等数据类型；
- 对象与对象之间用逗号隔开；
- “{}”用来保存对象；
- “[]”用来保存数组；

**json跟python中的字典看起来很像，两者之间的区别？**

1）json的key只能是字符串，dict的key可以是任何可hash的对象，例如：字符串、数字、元组等；

2）字典是一种数据结构，json是一种数据格式；字典有很多内置函数，有多种调用方法，而json是数据打包的一种格式，并不像字典具备操作性；

3）json的字符串强制用双引号，dict的字符串可以用单引号、双引号；

一般而言，我们会把json转化为python中的字典或者列表，再对其进行操作。

**Python处理json的模块：json**

Pythone3的标准库JSON模块，可以很方便的帮我们进行json数据的转换和处理，这里主要指序列化（json.dumps()、json.dump()）和反序列化（json.loads()、json.load()）。

**序列化和反序列化：**

将对象转换为可通过网络传输或可以存储到本地磁盘的数据格式（如：XML、JSON或特定格式的字节串）的过程称为序列化；反之，则称为反序列化。

**常用的JSON模块方法：**

- json.dumps():将Python中的对象转换为JSON中的字符串对象

  dumps 将一个字典转换成 json

- json.dump()：将python对象转换成JSON字符串输出到fp流中。

  dump 将一个文件转换成json

- json.loads():将JSON中的字符串对象转换为Python中的对象

  loads 读取sring 转化成字典

- json.load():读取包含json对象的文件。

  load 读取filename转化成字典

> 带s的都是和字符串相关的，不带s的都是和文件相关的。



JSON和Python内置的数据类型对应如下：

| json类型   | python类型 |
| ---------- | ---------- |
| object     | dict       |
| array      | list       |
| string     | str        |
| number     | int/float  |
| true/false | True/False |
| null       | None       |



#### dump 

json 模块的 dump 方法可以将 Python 对象序列化为 JSON 格式并写入到json文件中。

```
import json
Dict = {"name":"tom"}
with open("./file.json", "w") as f:
    json.dump(Dict, f)
```

 

#### load

json 模块的 load 方法将文件类对象转为 Python 对象

json.load()用于从json文件中读取数据

```
import json
with open("./file.json", "r") as f:  
    Dict=json.load(f) 
    print(Dict)
```

 

#### dumps

json 模块的 dumps 方法可以将 Python 对象转为 JSON 格式字符串

```
import json
Dict={"name": "tom"}
Str=json.dumps(Dict)
with open("./file.json", "w") as f:
    f.write(Str)
```

 json序列化时，默认遇到中文会转换成unicode，如果想要保留中文在序列化时，在dumps函数中添加参数ensure_ascii=False即可解决。

#### loads

json 模块的 loads 方法可以将 JSON 格式数据转为 Python 对象

```
import json
Dict={"name": "tom"}
Str=json.dumps(Dict)
Dict1 = json.loads(Str)
```