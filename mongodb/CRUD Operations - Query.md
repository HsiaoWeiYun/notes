### CRUD Operations - Query

* 指定相等條件
```text
格式: { field: <value> }
db.inventory.find({status: "D"})
```

* 指定查詢運算子 - $eq
```text
格式: { <field>: { $eq: <value> } }
等價: { field: <value> }, 這種查詢也可用於正規表示式

如果搜尋條件是一個document, 則這個doc內容順序很重要, 順序不同無法查出
ex:
    假設有資料{
    _id: ObjectId("62999c317f1a29a399289188"),
    item: 'journal',
    qty: 25,
    size: { h: 14, w: 21, uom: 'cm' },
    status: 'A'
  }
    查詢語法(可查出): db.inventory.find({size: {$eq: { h: 14, w: 21, uom: 'cm' }}});
    查詢語法(無法查出): db.inventory.find({size: {$eq: { uom: 'cm', h: 14, w: 21 }}})
    
如果搜尋條件是一個陣列, 則會搜出等價陣列或者字段中包含等價陣列, 陣列元素是順序敏感的
ex:
    假設有資料
    { _id: 1, item: { name: "ab", code: "123" }, qty: 15, tags: [ "A", "B", "C" ] }
    { _id: 2, item: { name: "cd", code: "123" }, qty: 20, tags: [ "B" ] }
    { _id: 3, item: { name: "ij", code: "456" }, qty: 25, tags: [ "A", "B" ] }
    { _id: 4, item: { name: "xy", code: "456" }, qty: 30, tags: [ "B", "A" ] }
    { _id: 5, item: { name: "mn", code: "000" }, qty: 20, tags: [ [ "A", "B" ], "C" ] }
    
    查詢語法: db.inventoryEQ.find({tags: {$eq:['A','B']}})
    查詢結果: 
    {_id: 3, item: { name: 'ij', code: '456' }, qty: 25, tags: [ 'A', 'B' ]},
    {_id: 5, item: { name: 'mn', code: '000' }, qty: 20, tags: [ [ 'A', 'B' ], 'C' ]}
```
* 指定查詢運算子 - $gt, $gte
```text
$gt是大於的意思, $gte是大於等於的意思
格式: { field: { $gt: value } }
ex:
    假設有資料 
   {
      "item": "nuts", "quantity": 30,
      "carrier": { "name": "Shipit", "fee": 3 }
   },
   {
      "item": "bolts", "quantity": 50,
      "carrier": { "name": "Shipit", "fee": 4 }
   },
   {
      "item": "washers", "quantity": 10,
      "carrier": { "name": "Shipit", "fee": 1 }
   }
   查詢語法: db.inventory.find( { quantity: { $gt: 20 } } )
   結果: 
   {
    _id: ObjectId("6299b62f7f1a29a39928918d"),
    item: 'nuts',
    quantity: 30,
    carrier: { name: 'Shipit', fee: 3 }
  },
  {
    _id: ObjectId("6299b62f7f1a29a39928918e"),
    item: 'bolts',
    quantity: 50,
    carrier: { name: 'Shipit', fee: 4 }
  }
```
* 指定查詢運算子 - $in
```text
$in代表查詢條件中任一符合即滿足條件
格式: { field: { $in: [<value1>, <value2>, ... <valueN> ] } }
效能考量: 使用$in運算子時建議參數個數不超過10個, 否則可能會有效能問題
ex:
    假設有資料
    { "item": "Pens", "quantity": 350, "tags": [ "school", "office" ] },
   { "item": "Erasers", "quantity": 15, "tags": [ "school", "home" ] },
   { "item": "Maps", "tags": [ "office", "storage" ] },
   { "item": "Books", "quantity": 5, "tags": [ "school", "storage", "home" ] }
   查詢語法: db.inventory.find( { quantity: { $in: [ 5, 15 ] } })
   結果:
   {
    _id: ObjectId("6299b88b7f1a29a399289191"),
    item: 'Erasers',
    quantity: 15,
    tags: [ 'school', 'home' ]
  },
  {
    _id: ObjectId("6299b88b7f1a29a399289193"),
    item: 'Books',
    quantity: 5,
    tags: [ 'school', 'storage', 'home' ]
  }
$in也譨使用正規表示式查詢
    如上例
    查詢語法: db.inventory.find( { tags: { $in: [/^s/] } })
    結果:
    {
    _id: ObjectId("6299b88b7f1a29a399289190"),
    item: 'Pens',
    quantity: 350,
    tags: [ 'school', 'office' ]
  },
  {
    _id: ObjectId("6299b88b7f1a29a399289191"),
    item: 'Erasers',
    quantity: 15,
    tags: [ 'school', 'home' ]
  },
  {
    _id: ObjectId("6299b88b7f1a29a399289192"),
    item: 'Maps',
    tags: [ 'office', 'storage' ]
  },
  {
    _id: ObjectId("6299b88b7f1a29a399289193"),
    item: 'Books',
    quantity: 5,
    tags: [ 'school', 'storage', 'home' ]
  }
```
* $lt、$lte 分別代表小於、小於等於, 用法如同$gt、$gte
* $ne 代表不等於的意思, 用法如同$eq
* $nin 代表不包含的意思用法如同$in
<br> <br>
* 邏輯查詢運算子 - $and
```text
格式: { $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }
陣列內的條件都符合才滿足條件
ex:
    條件：價格不等於1.99 且 存在價格field
    語法: db.inventory.find( { $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] } )
    語法: db.inventory.find( { price: { $ne: 1.99, $exists: true } } )
    
    條件: qty小於10 或 大於50 且 有打折 或 價格小於5元
    語法: 
    db.inventory.find( {
    $and: [
        { $or: [ { qty: { $lt : 10 } }, { qty : { $gt: 50 } } ] },
        { $or: [ { sale: true }, { price : { $lt : 5 } } ] }
    ]
} )
```
* 邏輯查詢運算子 - $not
```text
格式: { field: { $not: { <operator-expression> } } }
ex: 
    條件: 價格小於等於1.99或field不存在
    語法: db.inventory.find( { price: { $not: { $gt: 1.99 } } } )
也可用於正規表示式
如: db.inventory.find( { item: { $not: /^p.*/ } } )
```
* 邏輯查詢運算子 - $nor
```text
格式: { $nor: [ { <expression1> }, { <expression2> }, ...  { <expressionN> } ] }
ex:
    語法: db.inventory.find( { $nor: [ { price: 1.99 }, { sale: true } ]  } )
    條件: 價格不等於1.99且sale不等於true、價格不等於1.99但不存在sale filed
          不存在price field但sale不為true、price sale field 都沒有
```
* 邏輯查詢運算子 - $or
```text
格式: { $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
ex:
    條件: quantity小於20 或價格等於10元
    語法: db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )

index規則: 使用$or運算子時mongodb會執行集合掃描, 除非所有expression都有支援index,
           且每個expression可分別使用各自的index.
ex:
    下列語法要使用index需要單獨為field建立index, 而不是聯合索引
    db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )

$or and text Queries: 如果$or包含$text查詢則所有expression都需要支援index, 因為$text查詢一定要用index, 否則報錯

$or and GeoSpatial Queries:  $or supports geospatial clauses with the following exception for the near clause (near clause includes $nearSphere and $near). $or cannot contain a near clause with any other clause.

$or and Sort Operations: 當使用sort()執行$or時可以使用到index, 先前的版本不支援. (現行版本為5.0)

$or versus $in: 假如可以, 永遠優先使用$in
```
* 元素查詢運算子 - $exists
```text
格式: { field: { $exists: <boolean> } }
當值為true則會查出所有存在這個字段的資料 (包含null)
```
[Use a Sparse Index to Improve $exists Performance (待補)](https://www.mongodb.com/docs/manual/reference/operator/query/exists/#use-a-sparse-index-to-improve--exists-performance)

* 元素查詢運算子 - $type
```text
格式: { field: { $type: <BSON type> } } or { field: { $type: [ <BSON type1> , <BSON type2>, ... ] } }
用於查詢字段是否屬於指定的bson type, 如果需查詢的字段是陣列則陣列元素中有至少一個符合就會被查出

如下情況3.6以後的版本 (針對字段類型是array)都能查出, 但3.6前的版本(針對字段內的元素是array)只能查出第一筆
{ "data" : [ "values", [ "values" ] ] }
{ "data" : [ "values" ] }
查詢條件: find( {"data" : { $type : "array" } } )

4.2版本後無法將 $type: 0 作為 exists:false 的同義
```
$type運算子支援number 或 Alias 如下表
![](https://github.com/HsiaoWeiYun/notes/blob/master/mongodb/img/bson_type.png?raw=true)

* 陣列查訊運算子 - $all
```text
格式: { <field>: { $all: [ <value1> , <value2> ... ] } }
下兩句等價:
{ tags: { $all: [ "ssl" , "security" ] } }
{ $and: [ { tags: "ssl" }, { tags: "security" } ] }
```

* 陣列查訊運算子 - $elemMatch
```text
格式: { <field>: { $elemMatch: { <query1>, <query2>, ... } } }
此運算子會把該陣列任一元素滿足所有條件的資料找出
ex
    假設有資料
    { _id: 1, results: [ 82, 85, 88 ] }
    { _id: 2, results: [ 75, 88, 89 ] }
    查詢語法為
    db.scores.find({ results: { $elemMatch: { $gte: 80, $lt: 85 } } })
    查出任一元素滿足條件在80~85之間, 回傳下列資料
    { "_id" : 1, "results" : [ 82, 85, 88 ] }

Array of Embedded Documents
ex
    假設有資料
    db.survey.insertMany( [
        { "_id": 1, "results": [ { "product": "abc", "score": 10 },
                            { "product": "xyz", "score": 5 } ] },
        { "_id": 2, "results": [ { "product": "abc", "score": 8 },
                            { "product": "xyz", "score": 7 } ] },
        { "_id": 3, "results": [ { "product": "abc", "score": 7 },
                            { "product": "xyz", "score": 8 } ] },
        { "_id": 4, "results": [ { "product": "abc", "score": 7 },
                            { "product": "def", "score": 8 } ] }
    ] )
    查詢語法
    db.survey.find(
        { results: { $elemMatch: { product: "xyz", score: { $gte: 8 } } } }
    )
    查出資料為
    { "_id" : 3, "results" : [ { "product" : "abc", "score" : 7 },
                           { "product" : "xyz", "score" : 8 } ] }

```

* Use $all with $elemMatch
```text
ex
  db.inventory.find( {
                     qty: { $all: [
                                    { "$elemMatch" : { size: "M", num: { $gt: 50} } },
                                    { "$elemMatch" : { num : 100, color: "green" } }
                                  ] }
                   } )  
```

* Query on Embedded/Nested Documents
```text
ex
    假設有下列資料
    db.inventory.insertMany( [
       { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
       { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
       { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
       { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
       { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
    ]);
    Document需完全符合的狀況下 (順序無所謂)
    db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } ) 
                或
    db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )
    
    針對某嵌套字段的情況下可使用"點符號"
    db.inventory.find( { "size.uom": "in" } )
                或
    db.inventory.find( { "size.h": { $lt: 15 } } )
                或
    db.inventory.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } )
```

* 指定Query後想回傳的資料欄位 (projection)
```text
ex
    假設有下列資料
    db.inventory.insertMany( [
      { item: "journal", status: "A", size: { h: 14, w: 21, uom: "cm" }, instock: [ { warehouse: "A", qty: 5 } ] },
      { item: "notebook", status: "A",  size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "C", qty: 5 } ] },
      { item: "paper", status: "D", size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "A", qty: 60 } ] },
      { item: "planner", status: "D", size: { h: 22.85, w: 30, uom: "cm" }, instock: [ { warehouse: "A", qty: 40 } ] },
      { item: "postcard", status: "A", size: { h: 10, w: 15.25, uom: "cm" }, instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
    ]);
    
    回傳所有欄位
    db.inventory.find( { status: "A" } )
    
    回傳_id(預設), item, status
    db.inventory.find( { status: "A" }, { item: 1, status: 1 } )
    
    上列條件但不想回傳_id (除_id外其餘皆不可在'包含某字段的條件中排除字段')
    db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )
    
    回傳全部資料但唯獨排除特定字段
    db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )
    
    指定回傳的字段屬於嵌入字段一樣使用"點符號"表示
    db.inventory.find(
       { status: "A" },
       { item: 1, status: 1, "size.uom": 1 }
    )
```