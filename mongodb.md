# mongodb笔记

### 常用命令

use 数据库名称

db.dropDatabase() //删除数据库

db.createCollection("name") // 显式创建集合

show collections 或 show tables  // 查看当前库中的表

db.collection.drop() 或 db.集合名.drop()

db.集合名.insert({"articleid":"100000","userid":"1001"}) // 插入



db.comment.insertMany([
	{..},
	{..}
]) 批量插入



// ---------------查询

db.集合名.find() // 查找全部

db.集合名.find({userid:'1003'}) //条件查询

db.集合名.find({userid:'1003'},{userid:1,nickname:1}) //条件查询 部分显示 设置0为不显示


// ----------------更新
db.comment.update({_id:"1"}, {likenum:NumberInt(1001)}) // 会全部覆盖，其他字段没有了

db.comment.update({_id:"2"}, {$set:{likenum:NumberInt(889)}}) // 局部覆盖 只更新set了的字段

db.comment.update({userid:"1003"},
	{$set:{nickname:"凯撒大帝"}},{multi:true}) // 不设置multi默认只更新第一条数据
	
	
// -------------------删除
db.集合名称.remove({_id: "1"}) 

db.集合名称.remove({}) //全部删除