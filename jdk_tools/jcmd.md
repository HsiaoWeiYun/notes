### jcmd (可發送豐富的診斷指令給JVM)

* 用法: jcmd [pid | main-class] command...|PerfCounter.print| -f filename

* 參數描述
  * pid: 接受診斷的jvm process id (常用)
  * main class: 接受診斷的完整main class, 若有多個process使用相同名稱的main則診斷會一起傳入 (不常用)
  * command: 子命令 (常用)
  * Perfcounter.print: 顯示性能計數器 (不常用)
  * -f file: 從檔案讀取指令, 每隔命令需單獨一行, #開頭表註解 (不常用)
  * -l: jvm process 列表 (常用)
  * -h: 常看幫助 (常用)



#### jcmd -l
* 用途: 顯示jvm process id 與 main 名 或顯示 jar包名


#### jcmd <pid> help (常用)
* 用途: 找出這個jvm process 所支援的子命令

#### jcmd <pid> help command (常用)
* 用途: 找出子命令支援的選項

#### jcmd <pid> GC.heap_info (常用)
* 用途: 通用heap 資訊, 也包含 metaspace
* 引響程度: 中

#### jcmd <pid> PerfCounter.print
* 用途: 查看性能統計訊息

#### jcmd <pid> GC.class_histogram  (常用)
* 用途: 查看每個class 實體個數以及佔用容量
* 引響程度: 高 

#### jcmd <pid> Thread.print  (常用)
* 用途: 查看Thread stacktraces
* 引響程度: 中 (要看有多少Thread)

#### jcmd <pid> GC.heap_dump FILE_PATH  (常用)
* 用途: 產生HPROF格式的java heap dump
* 引響程度: 高 (要看Heap有多少記憶體)

#### jcmd <pid> VM.system_properties
* 用途: 查看jvm屬性資訊
* 引響程度: 低

#### jcmd <pid> VM.flags
* 用途: 查看jvm參數 (加上-all可列出所有參數)
* 引響程度: 低

#### jcmd <pid> VM.command_line
* 用途: 查看java 啟動參數
* 引響程度: 低

#### jcmd <pid> GC.run_finalization
* 用途: 執行一次java.lang.System.runFinalization()
* 引響程度: 中

#### jcmd <pid> GC.run
* 用途: 執行一次full gc
* 引響程度: 中

#### jcmd <pid> VM.version
* 用途: 查看版本
* 引響程度: 低

#### jcmd <pid> VM.native_memory [summary | detail | baseline | summary.diff | detail.diff | shutdown] [scale= KB | MB | GB] (特殊)
* 用途: 查看jvm process 的 Native Memory Tracking (NMT), -XX:NativeMemoryTracking 可開啟帶來5~10%效能損耗
* 引響程度: 中