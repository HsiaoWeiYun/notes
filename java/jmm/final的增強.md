### final的增強

#### final的重排需規則非為兩大類
* final變數的寫
  * JMM禁止編譯器把final變數的寫重排序到constructor之外
  * 編譯器會在final變數寫入之後constructor return之前插入一個StoreStore屏障. (這個屏障禁止把final變數重排序到constructor之外)
* final變數的讀
  * 確保final變數再讀之前一定已經初始化過了, 所以會禁止這類的重排序

簡而言之, JSR-133 增強了final語意, 確保final 變數再多執行緒的條件下能正確被初始化, 禁止了相關的重排序 <br>
多執行緒的情況下不用加鎖也能正確讀取到被初始化過後的數值.