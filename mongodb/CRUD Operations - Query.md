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

尚未完成....