### 常見SQL Mode <br><br>

* ONLY_FULL_GROUP_BY: 對於 group by 這種聚合操作, 如果select 出來的列沒有出現在group by 內則SQL不合法. <br><br>

* NO_AUTO_VALUE_ON_ZERO: 這個參數與auto increment欄位有關, 一般來說auto increment 欄位插入0 or NULL代表自動產生下一個自增值, 設此參數後會變更這個行為. <br><br>

* STRICT_TRANS_TABLES: 這個參數代表支援交易的表為嚴格模式 (1. 不支援對not null 插入null 2. 不支援auto increment 欄位插入'' 3. 不支援text欄位有預設值). <br><br>

* NO_ZERO_IN_DATE: 此參數引響日期不能為0 (年允許)否則違反的參數會變成 0000-00-00 , 並產生一條警告. 若為嚴格模式的話會直接報錯, 不會警告. <br> 
* NO_ZERO_DATE: 此參數引響日期格式不允許為0000-00-00, 違反會跳警告, 若為嚴格模式會直接報錯. <br>
**上述兩個參數雖並非嚴格模式的一部分但通常搭配嚴格模式使用.** <br><br>
  
* ERROR_FOR_DIVISION_BY_ZERO: 在insert or update 的過程中若資料被0除則產生錯誤而非警告. <br><br>

* NO_AUTO_CREATE_USER: 禁止grant創建密碼為空的用戶 <br><br>

* NO_ENGINE_SUBSTITUTION: 如果需要的引擎被禁用或者尚未編譯則拋出錯誤. <br><br>

* PIPES_AS_CONCAT: 把"||"視為字串連接而非運算  <br><br>

* ANSI_QUOTES: 不能用雙引號引號字串 <br><br>


ps. 可以直接 set session sql_mode = ' ... '  去實驗
