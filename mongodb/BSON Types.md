### BSON Types

* BSON是一個2進制可序列化格式用於在mongodb儲存documents以及遠端呼叫
* [BSON 詳細規格](Zhttps://bsonspec.org/)
* BSON Type 有兩種表示法 數字與字串, 如下表
  ![](https://github.com/HsiaoWeiYun/notes/blob/master/mongodb/img/bson_type.png?raw=true)
```text
The $type operator supports using these values to query fields by their BSON type. $type also supports the number alias, which matches the integer, decimal, double, and long BSON types.
The $type aggregation operator returns the BSON type of its argument.
The $isNumber aggregation operator returns true if its argument is a BSON integer, decimal, double, or long. New in version 4.4
```
* ObjectId: 有小、唯一、生成速度快、有序等特性. [(See also)](https://www.mongodb.com/docs/manual/reference/method/ObjectId/#mongodb-method-ObjectId)
```text
12bytes包含: 4bytes timestamp(Unix epoch)、每個process產生的5bytes 隨機數、3byte自增計數 <br>
如果一個整數被用來建立ObjectId則整數會取代timestamp的部分
在MongoDB中每個doc中都會需要一個唯一的_id作為主鍵, 如果沒有指定MongoDB driver會自動產生ObjectId作為_id
(upsert:true 的操作也同樣適用)
```
* String: BSON string 規格是UTF-8. 所以通常每種語言的driver會在序列化階段將字串轉成utf-8方便儲存BSON
* Timestamps: BSON有自己的時間儲存方式(64bits), 這跟我們一般Date type 無關
```text
前32bits為Unix epoch
後32bits為給定秒內的遞增序數
```
* Date: BSON使用64bit儲存Unix epoch (milliseconds) <br>
example
```js
var mydate1 = new Date()
var mydate2 = ISODate()
mydate1.toString()
mydate1.getMonth()
```