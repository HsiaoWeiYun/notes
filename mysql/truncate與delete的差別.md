### truncate與delete的差別 <br><br>

truncate屬於DDL語法, delete屬於DML語法, DDL在交易中無法回滾, 且truncate會清除自增屬性. <br> <br>

(刪除未清除自增屬性)
![](./img/test_delete.png)
<br> <br>
(已清除自增屬性)
![](./img/test_truncate.png)
<br> <br>
(DDL不支援交易)
![](./img/test_truncate_in_transaction.png)