### Transaction - ACID <br><br>

* 原子性 (Atomicity): <br>
指的是transaction中包含的所有操作只能是全部都做, 不然就是全部失敗, 保證資料庫是一致的. <br>
ex: A向B轉100元, A帳戶要先扣100元, B帳戶接著增加100元, 這兩個動作要就是都提交, 要就是都回滾, 不能發聲某一動作單獨成功或失敗的狀況. <br> <br>
  
* 一致性 (Consistency): <br>
指的是資料庫在transaction操作前和處理後都需要滿足業務規則約束. <br>
A and B 帳戶的總金額在轉帳前與轉帳後需要一致, 如有不一致必須是短暫的, 且只有在transaction提交前才會出現. <br> <br>
  
* 隔離性 (Isolation): <br>
允許多個transaction同時對資料庫進行讀寫的能力, 可以防止多個transaction交叉執行而導致資料的不一致. <br>
ex: A and B 之間轉帳, C也同時轉帳給A, AB的一致性需得到滿足, 故在AB執行轉帳過程中其他transaction不能修改當前相關的數值. <br> <br>
  
* 持久性 (Durability): <br>
transaction結束後對資料的修改是永久的, 即使發生故障也不會丟失.

  