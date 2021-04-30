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
待續....

  




