### jmap (可針對指定的process印出記憶體等資訊 或者產生dump檔案分析)

* 用法: jmap [options] pid
* 參數描述
  * clstats: 印出class loader分析, 細節參考 clstats column
  * finalizerinfo: 印出正在等待finalization的物件
  * histo[:live]: 印出 java object histogram
  * dump:<dump-options>: dump java heap


* clstats column
  * Index: 唯一性編號
  * Super: 父類編號
  * InstBytes: 每個實體的大小
  * KlassBytes: 每個class的大小
  * annotations: annotation 大小
  * CpAll: Combined size of the constants, tags, cache, and operands per class
  * MethodCount: 方法數量
  * Bytecodes: 用於byte code 的大小
  * MethodAll: Combined size of the bytes per method, CONSTMETHOD, stack map, and method data
  * ROAll: ***可*** 放入唯獨記憶體內的class metadata 大小
  * RWAll: ***必須***放入讀寫記憶體內的class metadata 大小
  * Total: Sum of ROAll + RWAll
  * ClassName: Name of the loaded class

    
* histo[:live]: 針對存活的物件做直方圖統計


* dump-options:
  * live: dump 存活物件, 不指定則整個heap都會被dump出來
  * format=b: dump格式為hprof binary format
  * file=filename: dump file name

Example: jmap -dump:live,format=b,file=heap.bin pid