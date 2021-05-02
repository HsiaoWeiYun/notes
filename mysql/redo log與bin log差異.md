###redo log與bin log差異 <br><br>

1. 紀錄內容不同 <br>
binlog紀錄所有數據改變資訊. <br>
redo log紀錄所有InnoDB表的變化. <br> 
2. 紀錄時間點不同 <br>
binlog紀錄commit完畢後的DML、DDL <br>
redo log紀錄事務開始之後的DML、DDL <br>
3. 文件寫入方式不同 <br>
binlog在寫滿或db重啟後我生成新的binlog文件 <br>
redo log為循環寫入, 最後一個文件寫滿後會重新寫第一個文件 <br>
4. 作用不同 <br>
binlog可用於主從同步、資料恢復 <br>
redo log用於當機後的資料恢復 <br>