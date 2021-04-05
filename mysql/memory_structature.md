### MySQL InnoDB 記憶體結構<br><br>

![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/innodb_memory_structature.png?raw=true)

<br><br>

* Buffer Pool: <br>
&emsp;&emsp;InnoDB引擎是基於硬碟儲存的, 其中使用page的方式進行管理, 但由於CPU與硬碟間的速度實在差異過大, 所以使用buffer來提高性能.<br>
讀取操作發生時, 首先先把從硬碟讀出的page放入buffer內, 下次在讀取相同page時會先判斷是否在buffer內, 若存在就直接讀取該page.<br>
對於修改操作則是先修改buffer內的page, 之後再以一定頻率刷新到硬碟上, 所以buffer pool size 直接影響到資料庫整體效能, 通過innodb_buffer_pool_size來設定. <br>
架構上來看buffer類型有: index page、data page、undo page、insert buffer page、adaptive hash index、lock info、data dictionary.<br>

* data [page](./表空間.md)、index [page](./表空間.md) :<br>
這個就不贅述了, 指的就是資料與index.<br>

* Double Write: <br>
&emsp;&emsp;主要解決IO寫入時頁毀損的問題. 舉例: page預設16k, 儲存系統IO最小單位4k, 硬碟io最小單位512bytes, 當DB正在將dirty page刷回硬碟時寫了2k忽然斷電, 這時導致前2k的資料是新的後14k是舊的, 這時這個page就是不完整的, 是一個毀損的page. <br><br>

![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/partial_page_write_failure.png?raw=true)

兩次寫提高了innodb的可靠性, 解決partial page write的問題, 由兩個部分組成, 一部分是記憶體中的 doublewrite buffer, 大小為2M, 另一部分是硬碟上[共享表空間](./表空間.md) (ibdata1)連續的128 page (也就是2個extent), 大小也是2M. <br>
當觸發buffer pool 髒頁刷新時不直接寫入硬碟, 而是先複製到記憶體中 (doublewrite buffer), 接下來分兩次從doublewrite buffer寫入共享表空間 (1次1M) (連續儲存, 順序寫, 效能高). 接著才真正寫入單獨表空間中. (隨機寫), 如果在寫入時發生停電, 在恢復時可從共享表空間的doublewrite找到該頁的副本, 執行恢復操作.<br><br>

![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/double_write_work_flow.png?raw=true)

<br><br>

* Insert Buffer: <br>
&emsp;&emsp;也叫做change buffer, 為了解決非聚簇索引 (待補)插入造成的問題. 一般聚簇索引 (ex: auto_increment) 新增資料時, 因為索引是保證遞增且唯一的, 所以按照順序存放就可以了, 並不需要去讀取其他page, 但對於非聚簇索引而言不保證是遞增的還具有一定的隨機性, 這是新增一條數據會嘗試讀取其他page (隨機io), 且當一個頁已滿或接近滿時會導致分頁的狀況造成額外資料複製的開銷. <br>
為了解上述問題, 當非聚簇索引新增資料時並不是每一次直接新增到索引內, 而是先放到 insert buffer後就當作索引已經新增到葉子節點中 (實際上沒有新增到B+ tree內), 接著再已一定頻率將 insert buffer 與 非聚簇索引合併, 這樣就大大提高了新增的效能.<br><br>

* Redo Log Buffer: <br>
&emsp;&emsp;當Buffer Pool 有髒頁時 (page比硬碟的資料還要新), 這時就需要將page data 刷到硬碟中, 但假如資料一有變動就要刷新開銷會很大, 所以InnoDB採用了"Write Ahead Log"機制 (待補), 即當交易提交後會先寫redo log buffer, 髒頁再擇時刷到硬碟中. 如果發生當機就利用redo log進行資料恢復.<br>

![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/redo_log_buffer_work_flow.png?raw=true)

<br><br>
Redo Log Buffer通常不用太大 (預設8M), 因為每秒都會將Redo Log Buffer的資料刷到硬碟中, 除此之外每次交易提交時也會刷, 通過"Force Log at Commit"機制實現交易持久化, 也就是說交易提交時會將Redo Log Buffer寫到 reo log file 內 (fsync), 交易才算完成.<br>
可以通過 innodb_flush_log_at_trx_commit 設定來控制這塊機制, 預設為1代表每提交一次就要執行一次fsync (效能最差但最安全), 0表示交易提交不呼叫fsync, 改由 main thread去操作, 2表示提交時只寫入os file system buffer, 不呼叫fsync (靠os自己同步的機制刷到硬碟內).<br><br>

* Adaptive Hash Index: <br>
InnoDB會持續監控表index查詢, 若發現建立 Hash Index 可以帶來效能提升, 就會開始建立.<br><br>

* lock info: <br>
InnoDB 會對行以及各種資源上鎖, 允許對資源併發訪問, 這邊就是存放鎖 (待補)的資訊.<br><br>

* Data Dictionary:<br>
包含表結構、欄位、view、trigger等訊息