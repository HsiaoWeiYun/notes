### Transaction - 語法 <br><br>

#### 顯式交易
交易語法是由 begin or start transaction 開始的, 或者把 autocommit 關掉 (set autocommit = 0) <br>
交易結束語法通常使用 commit or rollback "顯式"結束. <br><br>

![](./img/explicit_transaction_demo.png)

<br> <br>

#### 隱式交易
隱式提交可以是1.DDL語法 2.再次輸入begin or start transaction. <br>
隱式回滾可以退出session、連接超時or關機. <br> <br>

隱式提交
![](./img/implicit_transaction_commit.png)
<br> <br>
隱式回滾
![](./img/implicit_transaction_rollback.png)
