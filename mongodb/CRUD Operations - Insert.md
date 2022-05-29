### CRUD Operations - Insert

#### insertOne
```text
db.user.insertOne( { name: "vv123", age: 35 });
db.user.find({});
```

#### insertMany
```text
db.user.insertMany([
 { name: "gg1", age: 18 },
 { name: "gg3", age: 20 } 
]);
```

#### Insert Behavior:
如果collection為空新增資料時會自動建立 <br>
* _id: 每個document儲存時都會需要一個唯一的 **_id** , 如果每有指定MongoDB driver 會自動產生一個.
* Atomicity (原子性): 單一doc寫入時是具有原子性的, 更多的資訊待 Atomicity and Transactions 章節討論.
* Write Acknowledgement: 待 Write Concern 章節討論.
