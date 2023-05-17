# mongodb笔记

### 常用命令



```sql
-- 使用 也可以用于创建创建
use 数据库名称

-- 查看所以数据库
show dbs

-- 删除数据库
db.dropDatabase()



-- 显式创建集合
db.createCollection("name") 

-- 查看当前库中的表
show collections  或 show tables  

-- 删除集合
db.集合名称.drop()
```



### 添加

```sql
-- 插入
db.集合名称.insert({"articleid":"100000","userid":"1001"}) 

-- 批量插入
db.集合名称.insertMany([
	{..},
	{..}
]) 
```



### **查询**

```sql
db.集合名称.find() // 查找全部

db.集合名称.find({userid:'1003'}) //条件查询

db.集合名称.find({userid:'1003'},{userid:1,nickname:1}) //条件查询 部分显示 设置0为不显示
```



### 更新

```sql
-- 会全部覆盖，其他字段没有了
-- 条件字段不能加双引号
db.comment.update({_id:"1"}, {likenum:NumberInt(1001)}) 

-- 局部覆盖 只更新set了的字段
db.comment.update({_id:"2"}, {$set:{likenum:NumberInt(889)}}) 

-- 不设置multi默认只更新第一条数据
db.comment.update({userid:"1003"},
	{$set:{nickname:"凯撒大帝"}},{multi:true}) 
```





### **删除**

```sql
db.集合名称.remove({_id: "1"}) 

--全部删除
db.集合名称.remove({}) 
```



