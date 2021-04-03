### MySQL系統結構

MySQL基本的系統結構 (如下圖) 由上到下大致分為4層, 網路連接層/服務層/引擎層/系統檔案層<br>

![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/mysql_system_structure.png?raw=true)
<br>

* 網路連接層<br>
&emsp;&emsp;主要負責連接管理、授權認證、執行緒處理 ([One-Thread-Per-Connection](https://www.jianshu.com/p/ad350122d045))這三部分.<br>
當Client連到Server時server會對其進行認證, 可能是帳號密碼也可能是SSL憑證, 接著還會驗證是否具有執行這項操作的權限.<br>

* 服務層<br>
&emsp;&emsp;MySQL核心服務都在這層, 下圖簡單分析SQL語法在服務層的流程<br><br>
![](https://github.com/HsiaoWeiYun/notes/blob/master/mysql/img/mysql_system_structure_service.png?raw=true)
<br>
Query Cache: 解析SQL之前會先檢查Cache, 若可以找到對應的查詢就不必進行解析與優化, 直接返回快取中的結果. <font color='red'>(待補: 如何關閉Query Cache)</font> <br><br>

解析器與前處理器: 解析SQL並建立解析樹, 這部分主要驗證語法是否正確, 預處理會根據MySQL的規則進一步檢查解析樹是否合法, 查詢的資料表與欄位是否存在.<br><br>

查詢優化器: 產出查詢計畫, 一般來說一條查詢語句有很多種執行方式, 最後都會返回相同的結果. 優化器的作用就是找出最好的執行計畫. <font color='red'>(待補: MySQL優化器)</font><br><br>

查詢引擎: 完成解析與優化後, MySQL會生成對應的執行計畫, 查詢執行引擎根據執行計畫呼叫儲存引擎介面得出結果.<br><br>

* 儲存引擎<br>
&emsp;&emsp;MySQL資料庫與其他分支版本主要的引擎有 InnoDB、MyISAM、Memory、blackhole、TokuDB以及MariaDB columnstore. <br><br>

下列是引擎特性對比:<br>
引擎名稱 | 特點 | 應用場景
------|------|------
InnoDB|支援交易、行鎖、MVCC多版本控制|OLTP場景
MyISAM|不支援交易、表鎖|OLTP場景, 生產環境盡量少用 (版本8.0廢棄)
Memory|表中資料都存放在記憶體內, 支援Hash and Btree Index, 讀取速度快|應用對於安全性要求不高的條件下
TokuDB|Percona公司所有. 支援交易與壓縮功能, 高速寫入功能(比InnoDB快9倍),online DDL, 不產生index碎片|用於海量資料場景

<br>
InnoDB vs MyISAM<br>

差異 | InnoDB | MyISAM
-----|------|------
交易|支援|不支援
鎖粒度|行鎖|表鎖
多執行緒|高|低
結構與機制|資料與index都儲存在.Ibp文件裡, 且都cache在memory內|資料文件副檔名為.MYD, index文件副檔名為.MYI, 只cache index文件

<br><font color='red'>待補: InnoDB結構</font><br>

* 系統檔案層<br>
&emsp;&emsp;這層主要是將資料庫的資料儲存在檔案系統上，並完成與引擎間的互動. 下面列出MyISAM 與 InnoDB 物理檔案結構.<br><br>

MyISAM:<br>
* .frm: 與table相關的meta data都放置於此, 包含表結構定義資訊.<br>
* .MYD: 專放MyISAM表資料<br>
* .MYI: 專放MyISAM Index資料<br>

InnoDB:<br>
* .frm: table meta data都放置於此, 包含表結構定義資訊.<br>
* .iba: 專放InnoDB獨享表資料<br>
* .ibdata: 專放InnoDB共享表資料<br>

ibdata 與 iba 的差異: 兩種檔案都是存放innoDB資料, 因為innoDB可藉由設定決定使用"共享表空間" or "獨享表空間". 獨享表空間儲存方式使用.ibd檔案，並且每個表一個ibd檔案。共享表空間儲存方式採用.ibdata檔案，所有的表共同使用一個ibdata檔案，即所有的資料檔案都存在一個檔案中。決定使用哪種表的儲存方式可以通過mysql的配置檔案中 innodb_file_per_table選項來指定。InnoDB預設使用的是獨享表的儲存方式，<font color='red'>這種方式的好處是當資料庫產生大量檔案碎片的時，整理磁碟碎片對線上執行環境的影響較小。</font>
