### MySQL設定參數說明 <br><br>

* MySQL啟動讀取參數文件優先順序 (如果想指定參數文件啟動時加上--default-file參數): <br>
  /etc/my.cnf --> /etc/mysql/my.cnf --> /usr/local/mysql/my.cnf --> ~/my.cnf <br><br>
  
#### my.cnf 參數類型:
1. client section: 設定MySQL Client 的參數
2. server section: 設定MySQL Server 的參數 (下面一一說明) <br>

* innodb_buffer_pool_size: <br>
該buffer位於主記憶體中, InnoDB用來cache被訪問過的表與index文件, 讓常被訪問的資料可以直接在記憶體中被處理, 提升效能. <br>
ps. MySQL Server 只跑一個應用的前提下建議可以設置物理記憶體的50~80%. (5.7支持線上修改, 一開始不必設置太大) <br><br>

* innodb_buffer_pool_instances: <br>
默認為1, MySQL5.6.6之後可以設置為多個, 可以想成把innodb_buffer_pool劃分為多個實例, 提高並行性, 避免記憶體競爭的問題. <br><br>
ps. 當innodb_buffer_pool_size大於1G時該值才生效 <br>
ps. **show engine innodb status (待補)** --> 查看過去某段時間innodb狀態 <br><br>
  
* innodb_buffer_pool_load_at_startup: 啟動時將熱資料重新載入buffer中
* innodb_buffer_pool_dump_at_shutdown: 關閉時將bufferdump出來, 存在ib_buffer_pool 文件內 (參數 innodb_buffer_pool_filename) <br>
ps. 試想, 若DB當機了恢復時只能再次從硬碟讀取資料到buffer中, 若量很大的話會造成問題, 這時就可以靠上面這兩個參數了. <br><br>

* innodb_data_file_path: 系統表空間路徑與大小, 建議設為1G. <br>

* innodb_thread_concurrency: innodb 最大執行緒數量, 默認值為0, 代表不受限制. (待補: 執行緒效能調整) <br><br>

* interactive_timeout: 默認為8小時, MySQL關閉交互式連線前等待的活動時間. <br>
* wait_timeout: 默認為8小時, MySQL關閉非交互式連線前等待的活動時間. <br>
  (兩個參數要一起調整, 且值一至, 要避免過大的連接時間, 建議調整為300 ~ 600 秒) <br>
  (待補: MySQL Timeout 參數整理) <br><br>
 
* innodb_flush_method: 這個參數引響innodb budder pool 與redo log 的刷寫方式, O_SYNC (寫資料時先寫到os buffer即算完成)、 <br> 
  O_DSYNC (寫資料文件方式與O_SYNC一樣, 使用O_SYNC方式刷寫redo log)、 <br> 
  O_DIRECT (直接將資料往硬碟寫, 不經過os buffer), 哪個效能好要看使用狀況. (建議是O_DIRECT or O_SYNC) <br>
  (待補: Linux O_DIRECT  O_SYNC 區別) <br> <br> 
  
* transaction_isolation: 分別有READ-UNCOMMITTED(RU)、READ-COMMITTED(RC)、REPEATABLE-READ (RR, 預設)、SERIALIZABEL <br>
  (待補: 交易隔離層級) <br> <br>

* innodb_files: innodb可同時打開的.ibd文件個數, 最小為10, 預設為300, 建議調整為65535. <br> <br> 

* innodb_log_buffer_size: log buffer size, innodb改變資料時會把這次改動的紀錄先寫到log buffer中. <br> 
  (可以透過show global status like '%innodb_log_waits%' 查看, 如果innodb_log_waits(等待log buffer刷出的次數)大於0且一直增長, 可以增大 innodb_log_buffer_size, 取值範圍16M ~ 64M) <br>
* innodb_log_file_size: 是指redo log 的大小, 不宜太大太小, 太大db恢復時需要較長時間, 太小造成redo log 頻繁切換造成無用io. <br> <br>

* max_connections: 代表MySQL資料庫最大連線數, 預設151. (與innodb_thread_concurrency參數有關, 若該值為0代表thread數量無上限, 所以連接數大時有可能疲於context switch, 該值建議邏輯cpu核心數*2) <br> <br>

* show_query_log: 慢查詢log開關, 1等於開啟. <br>
* long_query_time: 慢查詢的定義時間, 超過就會紀錄在慢查詢內, 單位是秒. <br>
* log_queries_not_using_indexes: 如過運行的SQL沒有用index, 則同樣會紀錄到慢查詢內. <br><br>

* binlog_format: binlog格式, statement、row、mixed 三種. 生產用row更安全, 不會出現主從丟數據的情況. <br><br>

* lower_case_table_names: table name 是否區分大小寫, 預設為0, 0代表區分大小寫, 1代表不區分以小寫儲存. <br>
待續 ...





  




