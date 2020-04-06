# mongoDB

## 基本概念

1、组成  
> 数据库 可以理解为mysql中对应的数据库  
> 集合 可以对应为mysql中对应的表  
> 文档 可对应为mysql中的记录  

2.数据关系  
> 1.一对一  
> 2.一对多  
> 3.多对多  

## 登录示例

```mongodb
./mongo --host 127.0.0.1 --port 27017 --username root --password  --authenticationDatabase admin
```

## 语句

1.数据库与集合
_____

```db
use dbname; //dbname这个库如果存在，则切换，不存在则创建
db //查看当前在哪个库
```

show collections 查看当前库有哪些集合

```db
> show collections;
coll1
inventory
numbers
test
```

2.插入文档
_____

> collection:某个集合; doc:表示文档对象  

insert(doc) 向集合中插入一个或多个文档，如果此集合存在，则直接插入，如果不存在，则创建这个集合后再插入文档

```db
db.<collection>.insert(doc)
db.<collection>.insert({name:"abc",age:14}) //插入一个文档
db.<collection>.insert([{name:"张三"},{name:"李四"}]) //插入多个
db.<collection>.insert({name:"自设id",_id:"hello"})//_id也可以传入，但值一定要唯一，很多时候没有必要
```

以下两个语意更清晰  

```db
db.<collection>.insertOne(doc) //插入一个文档
db.<collection>.insertMany(doc) //插入多个文档
```

3.查找
_____

find($filter) 返回数组 查找集合中所有符合条件的文档$filter可以为空对象，或不写

```db
db.<collection>.find({_id:"hello"})
db.<collection>.find($filter).count() //查找数量，常用
db.<collection>.find($filter).length() //查找数量
db.<collection>.findOne($filter) //查找一个，返回对象
db.<collection>.find( { "size.uom": "in" } )//还可以这样
```

`注意：查找和顺序有关系 例如`

```db
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } ) //可以找到内容
db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  ) //找不到内容，因为顺序不对
```

4.更新(update)
_____

update($filter, $update) 修改内容，第一个是条件，第二个是要修改的内容,默认只修改一个

```db
db.<collection>.update($filter, {$set:{name:"沙和尚"}}) //如果要修改指定字段，要加修饰符
db.<collection>.update($filter, {$push:{name:"沙和尚"}}) //向数组中添加元素，不考虑是否重复
db.<collection>.update($filter, {$addToSet:{name:"沙和尚"}}) //向数组中添加元素，不重复

db.<collection>.update($filter, {$unset:{name:"沙和尚"}}) //删除一个属性  
db.<collection>.updateMany($filter, $update) //同时修改多个符合条件的item  
db.<collection>.updateOne($filter, $update) //默认只修改一个  
db.<collection>.replaceOne($filter, $update) //替换一条文档记录
```

4.删除(remove)
_____

```db
db.<collection>.remove($filter) //删除符合条件的所有item  
db.<collection>.remove($filter, true)  //只删除一个的话，要加第二个参数 为true  
db.<collection>.remove({}) //全删，清空了，性能略差,直接删除集合就ok了  
db.<collection>.deleteOne() //删一个  
db.<collection>.deleteMany() //删多个  
```

drop() 删除集合

```db
db.<collection>.drop()
```

db.dropDatabase() 删除数据库  

批量插入数据

```db
var arr=[]
for (var i=1; i<=1000; i++) {
    arr.push({num:i})
}
db.numbers.insert(arr)
```

符号
_____

skip(10)  跳过10条文档记录  
limit(15) 取最多15条记录  

```db
db.<collection>.find().skip(10).limit(10) //函数顺序无关
```

and逻辑

```db
db.<collection>.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } ) //and逻辑

db.<collection>.find( { dim_cm: { $gt: 15, $lt: 20 } } ) //需要同时符合两个条件
```

$all

```db
db.<collection>.find( { tags: ["red", "blank"] } ) //有一个符合条件的就找出来
db.<collection>.find( { tags: {$all, ["red", "blank"]} } )  //两个都查到才符合条件
```

$elemMatch 用来查询数组的，需要数组中包含符合条件的item  

```db
>db.<collection>.find( { num: {$elemMatch:{ $gt: 15, $lt: 20} } } )
{ "_id" : ObjectId("5e8a07baeb3226ed340db2e3"), "num" : [ 48, 16 ] }
```

$in 查出的内容不限制，可以是数组或者其它

```db
>db.numbers.find( { num: {$in:[16, 20] } } )
{ "_id" : ObjectId("5e881532b5e57a7b1609a56b"), "num" : 16 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a56f"), "num" : 20 }
{ "_id" : ObjectId("5e8a07baeb3226ed340db2e3"), "num" : [ 48, 16 ] }
```

逻辑符号表  
| $eq | $gt | $gte | $lt | $lte | $ne | $nin|
| :-----| ----: | :----: | :----: | :----: | :----: | :----: |
| equal | greate | great or equal | less | less or equal |not equal|not in |
| == | > | >= | < | <= | != | not in |

$lt

```db
db.inventory.find( { "size.h": { $lt: 15 } } )
```

$nin

```db
db.numbers.find( { num: {$nin:[16, 20] } } )
```

$and

```db
> db.numbers.find({$and:[{num:{$gt:4}},{num:{$lt:6}}]}).limit(3)
{ "_id" : ObjectId("5e881532b5e57a7b1609a560"), "num" : 5 }
```

$not

```db
> db.numbers.find({num:{$not:{$gt:4}}}).limit(3)
{ "_id" : ObjectId("5e881532b5e57a7b1609a55c"), "num" : 1 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55d"), "num" : 2 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55e"), "num" : 3 }
```

$nor

```db
> db.numbers.find({$nor:[{num:{$gt:4}},{num:{$lt:2}}]})
{ "_id" : ObjectId("5e881532b5e57a7b1609a55d"), "num" : 2 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55e"), "num" : 3 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55f"), "num" : 4 }
```

$or

```db
> db.numbers.find({$or:[{num:{$gt:4}},{num:{$lt:2}}]}).limit(3)
{ "_id" : ObjectId("5e881532b5e57a7b1609a55c"), "num" : 1 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a560"), "num" : 5 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a561"), "num" : 6 }
```

$inc

```db
> db.numbers.updateMany({num:{$lt:10}}, {$inc:{num:5}})
{ "acknowledged" : true, "matchedCount" : 9, "modifiedCount" : 9 }
```

sort() 1生序，-1降序

```db
> db.numbers.find().limit(5).sort({num:-1})
{ "_id" : ObjectId("5e881532b5e57a7b1609a943"), "num" : 1005 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a942"), "num" : 1004 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a941"), "num" : 1003 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a940"), "num" : 1002 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a93f"), "num" : 1001 }
```

find()第二个参数对象，对象1表示查出来，0表示不查此字段

```db
> db.numbers.find({}, {num:1}).limit(3)
{ "_id" : ObjectId("5e881532b5e57a7b1609a55c"), "num" : 6 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55d"), "num" : 7 }
{ "_id" : ObjectId("5e881532b5e57a7b1609a55e"), "num" : 8 }
```

更多内容参考[官网](https://docs.mongodb.com/manual/tutorial/query-arrays/)
