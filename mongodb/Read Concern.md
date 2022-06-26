### Read Concern

* 什麼是Read Concern: 一種控制replica sets and replica set shards '讀'操作的資料一致性與隔離性參數 <br>

ps. 從Mongodb4.4開始replica sets and sharded clusters支援全局預設read concern <br>

* Read Concern Levels: local、available、majority、linearizable、snapshot <br> <br>

ps. rollback指的是當資料寫入primary但未同步到secondary時, 此時primary當機secondary變為primary則資料遺失

#### local
可從mongo實體上讀取最新的資料, 不過這個資料並不保證已經寫到大部分的實體內, 也就是說有rollback的風險. <br>
<br>
MongoDb 4.4 開始可以在交易內建立Collections and Indexes, 如果是顯式創建則交易一定要read concern一定要local level <br> <br>

Example 寫入w-0 到 有三個節點的Rs <br>
為了方便演示簡單假設下列條件
1. 寫入w-0之前的寫入都已經成功寫入到所有節點
2. w-pre是w-0之前的寫入資料
3. w-0之後沒有其他寫入

![](https://github.com/HsiaoWeiYun/notes/blob/master/mongodb/img/read_concern_example.png?raw=true)

|  Time   | 事件  | 最近一次寫入 | 最後一次 'majority' 寫入 | 
|  ----  | ----  |  ----  | ----  |
| t0  | p寫入 w-0 | **p: w-0** s1: w-pre s2: w-pre | all: w-pre 
| t1 | s1寫入w-0 | p: w-0 **s1: w-0** s2: w-pre | all: w-pre
| t2 | s2寫入w-0 | p: w-0 s1: w-0 **s2: w-0** | all: w-pre
| t3 | s1 通知p已成功複製資料 | p: w-0 s1: w-0 s2: w-0 | **p: w-0** s1: w-pre s2: w-pre
| t4 | s2 通知p已成功複製資料 | p: w-0 s1: w-0 s2: w-0 | p: w-0 s1: w-pre s2: w-pre
| t5 | s1收到通知,更新majority snapshot 為w-0| p: w-0 s1: w-0 s2: w-0 | p: w-0 **s1: w-0** s2: w-pre
| t6 | s2收到通知,更新majority snapshot 為w-0| p: w-0 s1: w-0 s2: w-0 | p: w-0 s1: w-0 **s2: w-0**

總結: <br>

|  讀取節點   | 時間  | 資料狀態 | 
|  ----  | ----  |  ----  |
| p | t0 之後| w-0
| s1 | t1 之前| w-pre
| s1 | t1 之後| w-0
| s2 | t2 之前| w-pre
| s2 | t2 之後| w-0

#### available
主要用於sharded cluster上, 在rs上與local是等價的, <br>
在sharded cluster上有比較高的分區容錯性, 查詢時並不會向主分片或 config server 聯繫更新[metadata](https://www.mongodb.com/docs/manual/core/sharded-cluster-config-servers/) . <br><br>

與local的差異主要考慮以下情況:
1. 一個chunk x 正在從shard1向shard2遷移
2. 肘個遷移過程中chunk x 中的部分資料會在shard1與shard2同時存在, 但shard1仍舊是chunk x的負責方
3. 所有對chunk x的讀寫仍舊是進入shard1
4. config 中紀錄的資訊chunkx 依然屬於shard1
5. 此時若讀取shard2若
   * read concern - local: 只讀取shard2負責的資料 (不包含x)
   * read concern - available: 有什麼資料就讀什麼 (包含x)

不可與'causally consistent sessions and transactions' (待補) 一起用

#### majority
保證讀取到的資料已同步至大多數節點. (保證不會被rollback) <br> <br>
在multi-document transactions的操作中只有當transaction 提交時使用 write concern - majority, <br>
read concern - majority 才能提供其保證. (交易內讀取的資料已被同步至大多數節點的事實) <br> <br>

每個rs節點都維護一個majority-commit point快照, 快照直到沒人使用才會被刪除. <br>
如果使用primary-secondary-arbiter (PSA)架構可能會有下列問題:
* 當secondary不可用或lagging時會有效能問題
* 如果你使用majority為全局設定且write concern小於大多數則查詢時有可能會回傳未同步到大多數節點的資料

Example 寫入w-0 到 有三個節點的Rs <br>
為了方便演示簡單假設下列條件
1. 寫入w-0之前的寫入都已經成功寫入到所有節點
2. w-pre是w-0之前的寫入資料
3. w-0之後沒有其他寫入

![](https://github.com/HsiaoWeiYun/notes/blob/master/mongodb/img/read_concern_example.png?raw=true)

|  Time   | 事件  | 最近一次寫入 | 最後一次 'majority' 寫入 | 
|  ----  | ----  |  ----  | ----  |
| t0  | p寫入 w-0 | **p: w-0** s1: w-pre s2: w-pre | all: w-pre
| t1 | s1寫入w-0 | p: w-0 **s1: w-0** s2: w-pre | all: w-pre
| t2 | s2寫入w-0 | p: w-0 s1: w-0 **s2: w-0** | all: w-pre
| t3 | s1 通知p已成功複製資料 | p: w-0 s1: w-0 s2: w-0 | **p: w-0** s1: w-pre s2: w-pre
| t4 | s2 通知p已成功複製資料 | p: w-0 s1: w-0 s2: w-0 | p: w-0 s1: w-pre s2: w-pre
| t5 | s1收到通知,更新majority snapshot 為w-0| p: w-0 s1: w-0 s2: w-0 | p: w-0 **s1: w-0** s2: w-pre
| t6 | s2收到通知,更新majority snapshot 為w-0| p: w-0 s1: w-0 s2: w-0 | p: w-0 s1: w-0 **s2: w-0**

<br>

總結: <br>

|  讀取節點   | 時間  | 資料狀態 | 
|  ----  | ----  |  ----  |
| p | t3 之前| w-pre
| p | t3 之後| w-0
| s1 | t5 之前| w-pre
| s1 | t5 之後| w-0
| s2 | t6 之前| w-pre
| s2 | t6 之後| w-0

Read concern "majority" is available for the WiredTiger storage engine. <br>

#### linearizable (只適用於single document)
查詢返回的資料是已經同步到大多數節點的. 查詢結果返回前會等待直到所有節點都寫入才會返回. (與majority不同)
如果在讀取期間多數節點崩潰或重啟, 如果[writeConcernMajorityJournalDefault](https://www.mongodb.com/docs/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) 被設為true則會回傳已被持久化的資料 (已寫入硬碟) <br>
ps. writeConcernMajorityJournalDefault 被設為false時若資料寫入時指定write concern - majority, 寫入操作在確認寫入日誌 (on-disk journal)之前就會ack了, 也就是說此時若發生崩潰或重啟資料會消失(rollback). <br> <br>

TIP. 使用read concern - linearizable 時一定要與 maxTimeMS 參數一起使用, 為了避免多數節點不可用導致無限阻塞 <br>
由此可見效能上linearizable一定比majority要慢得多的. (為了在分散式架構下確保資料一致性所做的犧牲)
<br>

#### snapshot
read concern - snapshot 可使用於  multi-document transactions, <br>
且從MongoDb 5.0開始也可用於multi-document transactions之外的某些操作.
* 如果交易不是因果一致會話([causally consistent session](https://www.mongodb.com/docs/manual/core/read-isolation-consistency-recency/#std-label-sessions)) 的一部分, 交易提交使用 write concern - majority <br>
則保證從majority-committed快照讀取資料.
* 如果交易是因果一致會話的一部分, 交易提交使用 write concern - majority <br>
則保證從majority-committed快照讀取資料, 改快照提供交易開始前的因果一致性.
* atClusterTime: multi-document transactions之外的讀操作(read concern - snapshot)支援atClusterTime參數. <br>
atClusterTime與許你指定一個timestamp, 滿足讀取指定時間可用的快照資料. <br>
```text
db.runCommand( {
    find: "restaurants",
    filter: { _id: 5 },
    readConcern: {
        level: "snapshot",
        atClusterTime: Timestamp(1613577600, 1)
    },
} )
```
atClusterTime Considerations and Behavior:
* atClusterTime 與 [minSnapshotHistoryWindowInSeconds](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.minSnapshotHistoryWindowInSeconds) 相關, 改參數代表儲存引擎保留歷史快照的最小窗口(秒) <br>
如果指定的atClusterTime大於該窗口則返回錯誤.
* 如果對delayed replica set member 執行snapshot讀操作則可能得到舊的資料
* It is not possible to specify atClusterTime for "snapshot" inside of causally consistent sessions.