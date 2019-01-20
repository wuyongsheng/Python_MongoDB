>  工作中，经常会对MongoDB里面的数据进行读写，比如使用MongoDB对爬虫爬取的数据进行存取，MongoDB支持包括Python在内的多种编程语言的操作，这篇笔记主要是记叙Python对MongoDB的一些常用操作，参考了菜鸟教程。
###  Python3 JSON 数据解析
由于MongoDB使用的数据类型是 BSON（类似 JSON），所有对MongoDB进行操作时，经常涉及对 JSON 数据格式的读写。

Python3 中可以使用 json 模块来对 JSON 数据进行编解码，它包含了两个函数：
-  json.dumps(): 对数据进行编码。
-  json.loads(): 对数据进行解码。

在json的编解码过程中，python 的原始类型与json类型会相互转换

**json.dumps 与 json.loads 实例**
以下实例演示了 Python 数据结构转换为JSON：
```
import json
 
# Python 字典类型转换为 JSON 对象
data = {
    'no' : 1,
    'name' : 'Runoob',
    'url' : 'http://www.runoob.com'
}
 
json_str = json.dumps(data)
print ("Python 原始数据：", repr(data))
print ("JSON 对象：", json_str)

# 输出：
# Python 原始数据： {'no': 1, 'name': 'wysh', 'url': 'http://wysh.site'}
# JSON 对象： {"no": 1, "name": "wysh", "url": "http://wysh.site"}

```
![json_dumps.jpg](https://upload-images.jianshu.io/upload_images/12273007-1127579dc575f46c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过输出的结果可以看出，简单类型通过编码后跟其原始的repr()输出结果非常相似。

接着以上实例，我们可以将一个JSON编码的字符串转换回一个Python数据结构：
```
import json

# Python 字典类型转换为 JSON 对象
data1 = {
    'no': 1,
    'name': 'wysh',
    'url': 'http://wysh.site'
}

json_str = json.dumps(data1)
print("Python 原始数据：", repr(data1))
print("JSON 对象：", json_str)

# 将 JSON 对象转换为 Python 字典
data2 = json.loads(json_str)
print("data2['name']: ", data2['name'])
print("data2['url']: ", data2['url'])
```
输出的结果为：
```
JSON 对象： {"no": 1, "name": "wysh", "url": "http://wysh.site"}
data2['name']:  wysh
data2['url']:  http://wysh.site
```
如果你要处理的是文件而不是字符串，你可以使用 json.dump() 和 json.load() 来编码和解码JSON数据。
例如：使用 json.dump()将Python字典的内容写入本地的磁盘文件中，代码如下：
```
import json
data={"name":"吴","age":26}
# 写入 JSON 数据到本地的磁盘文件：c:\1.json
with open(r'c:\1.json', 'w+',encoding='utf-8') as f:
    json.dump(data, f)
```
查询运行结束之后，可以看到，本地的生产了一个 1.json文件，内容如下：

![json_dump.jpg](https://upload-images.jianshu.io/upload_images/12273007-9deb2742d4de59b3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 使用 json.load() 解码本地文件的JSON数据。
 代码如下：
 ```
 import json
# 读取本地磁盘 c:\1.json 的 json 文件
with open(r'c:\1.json', 'r') as f:
    data = json.load(f)
print(repr(data))
print("name: "+data["name"])
```
运行结果如下图：

 ![json_load.jpg](https://upload-images.jianshu.io/upload_images/12273007-9dd3d766bfef42bc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 建立 MongoDB 的连接

- 安装pymongo

使用 pip3 install pymongo 安装MongoDB 驱动，如果已经安装，会有相应的提示：
```
C:\Users\Administrator>pip3 install pymongo
Requirement already satisfied: pymongo in e:\python\python36-32\lib\si
te-packages (3.7.2)
```
-  创建一个数据库

创建数据库需要使用 MongoClient 对象，并且指定连接的 URL 地址和要创建的数据库名。

如下实例中，我们创建的数据库 wysh :
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
```
- 判断数据库是否已存在

我们可以读取 MongoDB 中的所有数据库，并判断指定的数据库是否存在：
```
import pymongo

myclient = pymongo.MongoClient('mongodb://localhost:27017/')

dblist = myclient.list_database_names()
# dblist = myclient.database_names()
print(dblist)
if "wysh" in dblist:
    print("数据库已存在！")
```
***注意:** 在 MongoDB 中，数据库只有在内容插入后才会创建! 就是说，数据库创建后要创建集合(数据表)并插入一个文档(记录)，数据库才会真正创建。*

输出结果如下：
```
['admin', 'config', 'local', 'nihao', 'test']
```
-  创建集合

MongoDB 中的集合类似 SQL 的表。

MongoDB 使用数据库对象来创建集合，实例如下：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
wysh = myclient["wysh"]

mycol = wysh["sites"]
```
-  判断集合是否已存在

我们可以读取 MongoDB 数据库中的所有集合，并判断指定的集合是否存在：
```
import pymongo

myclient = pymongo.MongoClient('mongodb://localhost:27017/')

mydb = myclient['wysh']
mycol = mydb["sites"]
collist = mydb.list_collection_names()
print(collist)
# collist = mydb.collection_names()
if "sites" in collist:  # 判断 sites 集合是否存在
    print("集合已存在！")
```
输出的结果为空

###  Python Mongodb 插入文档
MongoDB 中的一个文档类似 SQL 表中的一条记录。

- 插入集合

集合中插入文档使用 insert_one() 方法，该方法的第一参数是字典 name => value 对。

以下实例向 sites 集合中插入文档：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

mydict = {"name": "wysh", "alexa": "10000", "url": "https://wysh.site"}

x = mycol.insert_one(mydict)
print(x)
```
输出的结果为：
```
<pymongo.results.InsertOneResult object at 0x024DCDA0>
```
-  返回 _id 字段

insert_one() 方法返回 InsertOneResult 对象，该对象包含 inserted_id 属性，它是插入文档的 id 值。
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

mydict = {"name": "wysh", "alexa": "10000", "url": "https://wysh.site"}

x = mycol.insert_one(mydict)
print(x.inserted_id)
```
返回结果：
```
5c447686d7a6232500733686
```
如果我们在插入文档时没有指定 _id，MongoDB 会为每个文档添加一个唯一的 id。

- 插入多个文档

集合中插入多个文档使用 insert_many() 方法，该方法的第一参数是字典列表。
```
mylist = [
    {"name": "Taobao", "alexa": "100", "url": "https://www.taobao.com"},
    {"name": "QQ", "alexa": "101", "url": "https://www.qq.com"},
    {"name": "Facebook", "alexa": "10", "url": "https://www.facebook.com"},
    {"name": "知乎", "alexa": "103", "url": "https://www.zhihu.com"},
    {"name": "Github", "alexa": "109", "url": "https://www.github.com"}
]

x = mycol.insert_many(mylist)
```
###  Python Mongodb 查询文档
MongoDB 中使用了 find 和 find_one 方法来查询集合中的数据，它类似于 SQL 中的 SELECT 语句。

- 查询一条数据

我们可以使用 find_one() 方法来查询集合中的一条数据。

查询 sites 文档中的第一条数据
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

x = mycol.find_one()

print(x)
```
查询的结果如下：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

x = mycol.find_one()

print(x)
```
查询结果如下：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
```
-  查询集合中所有数据

find() 方法可以查询集合中的所有数据，类似 SQL 中的 SELECT * 操作。

以下实例查找 sites 集合中的所有数据：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

for x in mycol.find():
    print(x)
```
输出结果如下：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c4477c9d7a6234cbca81219'), 'name': 'Taobao', 'alexa': '100', 'url': 'https://www.taobao.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121a'), 'name': 'QQ', 'alexa': '101', 'url': 'https://www.qq.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121b'), 'name': 'Facebook', 'alexa': '10', 'url': 'https://www.facebook.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121c'), 'name': '知乎', 'alexa': '103', 'url': 'https://www.zhihu.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121d'), 'name': 'Github', 'alexa': '109', 'url': 'https://www.github.com'}
```
-  查询指定字段的数据

我们可以使用 find() 方法来查询指定字段的数据，将要返回的字段对应值设置为 1。
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

for x in mycol.find({}, {"_id": 0, "name": 1, "alexa": 1}):
    print(x)
```
输出的结果如下：
```
{'name': 'wysh', 'alexa': '10000'}
{'name': 'wysh', 'alexa': '10000'}
{'name': 'Taobao', 'alexa': '100'}
{'name': 'QQ', 'alexa': '101'}
{'name': 'Facebook', 'alexa': '10'}
{'name': '知乎', 'alexa': '103'}
{'name': 'Github', 'alexa': '109'}
```
输出的结果为：
```
{'name': 'wysh', 'alexa': '10000'}
{'name': 'wysh', 'alexa': '10000'}
{'name': 'Taobao', 'alexa': '100'}
{'name': 'QQ', 'alexa': '101'}
{'name': 'Facebook', 'alexa': '10'}
{'name': '知乎', 'alexa': '103'}
{'name': 'Github', 'alexa': '109'}
```
-  根据指定条件查询

我们可以在 find() 中设置参数来过滤数据。

以下实例查找 name 字段为 "wysh" 的数据：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": "wysh"}

mydoc = mycol.find(myquery)

for x in mydoc:
    print(x)
```
输出结果为：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
```

-  高级查询


查询的条件语句中，我们还可以使用修饰符。

以下实例用于读取 name 字段中第一个字母 ASCII 值大于 "H" 的数据，大于的修饰符条件为 {"$gt": "R"} :
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": {"$gt": "R"}}

mydoc = mycol.find(myquery)

for x in mydoc:
    print(x)
```
输出的结果为：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c4477c9d7a6234cbca81219'), 'name': 'Taobao', 'alexa': '100', 'url': 'https://www.taobao.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121c'), 'name': '知乎', 'alexa': '103', 'url': 'https://www.zhihu.com'}
```

-  使用正则表达式查询

我们还可以使用正则表达式作为修饰符。

正则表达式修饰符只用于搜索字符串的字段。

以下实例用于读取 name 字段中第一个字母为 "w" 的数据，正则表达式修饰符条件为 {"$regex": "^W"} :
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": {"$regex": "^w"}}

mydoc = mycol.find(myquery)

for x in mydoc:
    print(x)
```
输出的结果为：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
```

###  Python Mongodb 修改文档

我们可以在 MongoDB 中使用 update_one() 方法修改文档中的记录。该方法第一个参数为查询的条件，第二个参数为要修改的字段。

如果查找到的匹配数据多余一条，则只会修改第一条。
以下实例将 alexa 字段的值 10000 改为 12345:
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"alexa": "10000"}
newvalues = {"$set": {"alexa": "12345"}}

mycol.update_one(myquery, newvalues)

# 输出修改后的  "sites"  集合
for x in mycol.find():
    print(x)
```
输出的结果为：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '12345', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '10000', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c4477c9d7a6234cbca81219'), 'name': 'Taobao', 'alexa': '100', 'url': 'https://www.taobao.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121a'), 'name': 'QQ', 'alexa': '101', 'url': 'https://www.qq.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121b'), 'name': 'Facebook', 'alexa': '10', 'url': 'https://www.facebook.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121c'), 'name': '知乎', 'alexa': '103', 'url': 'https://www.zhihu.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121d'), 'name': 'Github', 'alexa': '109', 'url': 'https://www.github.com'}
```
update_one() 方法只能修匹配到的第一条记录，如果要修改所有匹配到的记录，可以使用 update_many()。

以下实例将查找所有以 F 开头的 name 字段，并将匹配到所有记录的 alexa 字段修改为 123：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": {"$regex": "^w"}}
newvalues = {"$set": {"alexa": "123"}}

x = mycol.update_many(myquery, newvalues)

print(x.modified_count, "文档已修改")
```
输出结果如下：
```
2 文档已修改
```
###  排序
sort() 方法可以指定升序或降序排序。

sort() 方法第一个参数为要排序的字段，第二个字段指定排序规则，1 为升序，-1 为降序，默认为升序。

- 对字段 alexa 按升序排序：

```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

mydoc = mycol.find().sort("alexa",-1)
for x in mydoc:
    print(x)
```
结果如下：
```
{'_id': ObjectId('5c44753fd7a623426018eaed'), 'name': 'wysh', 'alexa': '123', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c447686d7a6232500733686'), 'name': 'wysh', 'alexa': '123', 'url': 'https://wysh.site'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121d'), 'name': 'Github', 'alexa': '109', 'url': 'https://www.github.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121c'), 'name': '知乎', 'alexa': '103', 'url': 'https://www.zhihu.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121a'), 'name': 'QQ', 'alexa': '101', 'url': 'https://www.qq.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca81219'), 'name': 'Taobao', 'alexa': '100', 'url': 'https://www.taobao.com'}
{'_id': ObjectId('5c4477c9d7a6234cbca8121b'), 'name': 'Facebook', 'alexa': '10', 'url': 'https://www.facebook.com'}
```

###  Python Mongodb 删除数据

我们可以使用 delete_one() 方法来删除一个文档，该方法第一个参数为查询对象，指定要删除哪些数据。

以下实例删除 name 字段值为 "Taobao" 的文档：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": "Taobao"}

mycol.delete_one(myquery)

# 删除后输出
for x in mycol.find():
    print(x)
```
输出的结果中，name 字段值为 "Taobao" 的文档被删除

-  删除多个文档

我们可以使用 delete_many() 方法来删除多个文档，该方法第一个参数为查询对象，指定要删除哪些数据。

删除所有 name 字段中以 w 开头的文档:
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

myquery = {"name": {"$regex": "^w"}}

x = mycol.delete_many(myquery)

print(x.deleted_count, "个文档已删除")
```
输出的结果为：
```
2 个文档已删除
```
-  删除集合中的所有文档

delete_many() 方法如果传入的是一个空的查询对象，则会删除集合中的所有文档：
```
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/")
mydb = myclient["wysh"]
mycol = mydb["sites"]

x = mycol.delete_many({})

print(x.deleted_count, "个文档已删除")
```
输出结果为：
```
4 个文档已删除
```
