### binlog <br>
binlog用於記錄資料庫寫入操作訊息, 已檔案形式儲存在硬碟中. <br>
binlog是通過追加的方式寫入的, 通過max_binlog_size參數可以設定binlog檔案大小, 當檔案超過大小就會生成新的binlog檔案. <br><br>

* 使用場景
1. 主從複製: 在Master將binlog發送到各Slave端, Slave端重放binlog達到資料一至. 
2. 數據恢復: 通過**mysqlbinlog (待補)** 工具來恢復資料
<br>
* binlog刷硬碟時機 <br>
對於innodb來說, 只有在事務提交時才會紀錄binlog, 通過sync_binlog參數來控制刷硬碟時機. <br>
0: 不強制, 讓FileSystem自行決定何時寫入硬碟 <br>
1: 每commit一次就寫一次硬碟 <br>
n: 每commitn次就寫一次硬碟 <br><br>

* binlog格式 <br>
STATMENT: 基於SQL的複製 (statement-base replication, SBR), SQL語法紀錄在binlog中. <br>
&nbsp; &nbsp; &nbsp;  優: 不需紀錄每個資料的變化, 減少binlog大小. <br>
&nbsp; &nbsp; &nbsp;  缺: 某些狀況主從不同步. ex: sysdate slepp 等 <br>
ROW: 基於行的複製 (row-based replication, RBR), 紀錄哪條資料被改了 <br>
&nbsp; &nbsp; &nbsp;  優: 不會有主從不同步的問題 <br>
&nbsp; &nbsp; &nbsp;  缺: 會產生大量binlog <br>
MIX: 基於STATMENT和ROW兩種混合複製 (mixed-based replication, MBR), 一般的資料用STATMENT, 有主從問題的用ROW.