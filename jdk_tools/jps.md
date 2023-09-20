### jps (JVM Process Status Tool)

* 用途: 找出LVMID (Local Virtual Machine Identifier), 也可找出啟動參數、main class 等資訊

* 命令格式: jps \[options] \[LVMID] <br><br>

Options

| options | 描述                          |
|---------|-----------------------------|
| -q      | 只輸出LVMID這個資訊                |
| -m      | 只輸出傳遞給main方法的參數             |
| -l      | 輸出main方法的完整名稱, 或者jar包名稱     |
| -v      | 顯示傳遞給jvm的參數                 |
| -V      | 只顯示LVMID 與 main方法 or jar包名稱 |

