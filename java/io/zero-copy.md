### Zero-Copy

想要理解什麼是zero-copy必須修知道下面幾個相關知識
1. IO呼叫流程
2. DMA
3. Virtual Memory
4. MMap
5. sendfile

#### IO呼叫流程
具體流程可以參考[五種IO模型](五種io模型.md) <br>
簡單來說一個IO的流程為 (BIO舉例): 
1. 首先app對作業系統發出system call 請求 (要求讀取檔案)
2. cpu 將user mode 切為 kernel mode (content switch 1次)
3. 等待硬碟將資料複製到kernel buffer (複製 1次)
4. 複製完畢後cpu將資料從kernel buffer 複製到 user buffer (複製 2次)
5. cpu 將kernel mode 切回user mode (content switch 2次)

一次io花了2次cpu content switch, 2次 cpu 複製

#### DMA
* 什麼是DMA: DMA全名為 <font color=#FF0000> Direct Memory Access (直接訪問器) </font> 為主機板上的一塊獨立晶片
* 用途: DMA的作用很單純, 就是允許外部設備直接與buffer之間進行IO資料傳輸, 過程不需要cpu參與, 達到cpu使用率變高的目的

#### Virtual Memory
早期的機器記憶體都很小, 所以開不了太多的process, 為了解決的個困擾就有了虛擬記憶體的技術 <br>
process 都有一塊虛擬的記憶體空間, 這些虛擬的記憶體空間會映射到實體的記憶體位址, 這就導致了會有不同的process共用一塊實體記憶體位置 <br>
採取的作法就是動態的將記憶體上的資料換入換出 (硬碟swap區域) <br>
舉例: 實體記憶體1g; 虛擬記憶體2g, 超出的部分利用LRU or LFU 算法將不常訪問的空間換入到硬碟swap, 需要訪問的時候再轉移回來即可

#### Zero Copy
零拷貝不是沒有拷貝的意思, 而是**減少cpu context switch** 以及**cpu拷貝次數**

#### 後續說明範例
假設一個文件下載的IO過程, 從硬碟上讀取檔案接著透過網路傳送給用戶 <br>
1. 應用程式呼叫system call, 調用read方法, cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 1次) </font>
2. DMA將資料從硬碟寫入kernel buffer <font color=#FF0000> (DMA 複製 1次) </font>
3. cpu 將資料從kernel buffer 複製到 user buffer <font color=#FF0000> (cpu 複製 1次) </font>, cpu context switch 將kernel mode 轉為user mode <font color=#FF0000> (content switch 2次) </font>
4. read方法返回, 接著調用write發動io請求, cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 3次) </font>
5. cpu 將user buffer的資料複製到socket buffer <font color=#FF0000> (cpu 複製 2次) </font>, 再將socket buffer資料複製給網卡 <font color=#FF0000> (DMA 複製 2次) </font>
6. cpu context switch 將kernel mode轉為 user mode <font color=#FF0000> (content switch 4次) </font>
7. write方法轉回

可以看出歷經了**4次context switch** 、 **2次cpu複製** 以及 DMA複製2次

#### Zero-Copy - mmap + write
上面提到虛擬記憶體的部分, 代表著可以將kernel buffer 映射到 user buffer, 這樣就可以避免這一段流程的複製 <br>
(意思是多個虛擬地址指向同個實體地址) <br>

[void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);](https://man7.org/linux/man-pages/man2/mmap.2.html)
* addr: 虛擬記憶體位置
* length: 映射長度
* prot: 映射記憶體的保護模式
* flags: 映射類型
* fd: 映射的文件fd
* offset: 文件偏移量

#### mmap + write 實現流程如下
1. 應用程式利用mmap方法向作業系統請求IO操作
2. cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 1次) </font>
3. CPU利用DMA把資料從硬碟複製到kernel buffer <font color=#FF0000> (DMA 複製 1次) </font>
4. cpu context switch 將kernel mode轉為 user mode <font color=#FF0000> (content switch 2次) </font>
5. mmap 方法返回, 應用程式透過write向作業系統發起io請求
6. cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 3次) </font>
7. cpu將kernel buffer 的資料複製到 socket buffer <font color=#FF0000> (cpu 複製 1次) </font>
8. cpu利用DMA把socket buffer資料複製到網卡上 <font color=#FF0000> (DMA 複製 2次) </font>
9. cpu context switch 將kernel mode轉為 user mode <font color=#FF0000> (content switch 4次) </font>
10. write method 返回

可以發現 mmap + write 的實作, content switch 還是發生了4次, 但是 **cpu複製減少到了1次**, DMA複製還是有2次

#### Zero-Copy - sendfile
linux kernel 2.1 之後才出現, [ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);](https://man7.org/linux/man-pages/man2/sendfile.2.html)
* out_fd: 待寫入的文件fd
* in_fd: 待讀取的文件fd
* offset: 指定偏移量, 從哪邊開始讀
* count: count is the number of bytes to copy between the file
  descriptors.
可以看出sendfile是在作業系統內核操作的, 可以避免用戶空間與內核空間的複製操作, 實現Zero-Copy

#### sendfile實現流程如下
1. 應用程式發起sendfile調用, cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 1次) </font>
2. DMA將資料從硬碟複製到kernel buffer <font color=#FF0000> (DMA 複製 1次) </font>
3. cpu將kernel buffer資料複製到socket buffer <font color=#FF0000> (cpu 複製 1次) </font>
4. DMA將socket buffer資料複製到網卡上 <font color=#FF0000> (DMA 複製 2次) </font>
5. cpu context switch 將kernel mode轉為 user mode <font color=#FF0000> (content switch 2次) </font>
6. sendfile返回
可以發現sendfile實現將**context switch將低到剩下2次**, **cpu 複製只剩下1次**, 不可避免的DMA複製有2次

感覺可以把剩下的一次Cpu複製也省下來, 答案就是 sendfile + Scatter-Gather DMA <br>
也就是讓DMA具有直接將buffer的資料複製到網卡內的能力, 省下最後一次的cpu複製

#### sendfile + Scatter-Gather DMA 實現流程如下
1. 應用程式發起sendfile調用, cpu context switch 將user mode 轉為 kernel mode <font color=#FF0000> (content switch 1次) </font>
2. DMA將資料從硬碟複製到kernel buffer <font color=#FF0000> (DMA 複製 1次) </font>
3. Cpu收集kernel buffer fd相關資料 (內存地址、偏移量等等)傳送到socket buffer.
4. DMA根據socket buffer內的信息將kernel buffer的資料直接複製到網卡 <font color=#FF0000> (DMA 複製 2次) </font>
5. cpu context switch 將kernel mode轉為 user mode <font color=#FF0000> (content switch 2次) </font>
6. sendfile返回
可以發現僅發生了2次context switch、DMA複製兩次, Cpu複製一次都沒有, 完美達到Zero-Copy
   (DMA複製無所謂, Cpu讓其去做事後就可以切換去做其他事情, 等DMA發送中斷事件即可)