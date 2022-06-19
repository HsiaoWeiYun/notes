### CRUD Operations - Update

* db.collection.deleteOne()
* db.collection.deleteMany()
```text
格式: db.collection.deleteOne(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>,
      hint: <document|string>        // Available starting in MongoDB 4.4
   }
)
刪除第一筆找到的資料

filter: 搜尋條件
writeConcern: writeConcern策略
collation: 尚未花時間搞懂
hint: 尚未花時間搞懂

待補: 搞清楚什麼是collation、hint

假設有下列資料
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
] );
刪除第一筆item為journal的資料
db.inventory.deleteOne({item: "journal"});

格式: db.collection.deleteMany(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)

filter: 搜尋條件
writeConcern: writeConcern策略
collation: 尚未花時間搞懂

刪除全部的資料
db.inventory.deleteMany({});

刪除status為A的資料
db.inventory.deleteMany({ status : "A" })
```
* 刪除行為
  1. 只能刪除資料'無法'刪除index
  2. 原子性 (章節待補)
  3. Write Concern (章節待補)
