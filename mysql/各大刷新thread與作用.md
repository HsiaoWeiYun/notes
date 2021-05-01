### 各大刷新Thread與作用 <br><br>

* master thread: 優先權最高, 內部有四個Loop分別為main loop、backgroup loop、flusg loop、subspend loop. <br>
main loop又包含每1s與每10s的操作. <br>
每1s操作: <br>
1. log buffer flush到硬碟
2. 刷新髒頁至硬碟
3. merge change buffer
4. 產生checkpoint
5. 清除不需要的table cache
6. 切換至background loop <br><br>

每10s操作: <br>
1. log buffer flush到硬碟
2. merge change buffer
3. 刷新髒頁至硬碟
4. 刪除不需要的undo page
5. 產生checkpoint <br><br>

* 四大I/O Thread: <br>
1. redo log thread: 負責把buffer刷新到redo log文件.
2. change buffer thread: 負責把change buffer的內容刷到硬碟.
3. read/write thread, 默認都是4個, 使用高轉速硬碟可以加大此值 (innodb_read_io_threads、innodb_write_io_threads ). <br><br>

* 其他
1. page cleaner thread: 負責髒頁刷新的thread
2. purge thread: 刪除不需要的undo page
3. error monitor thread: 負責監控database錯誤
4. lock monitor thread: 負責監控鎖