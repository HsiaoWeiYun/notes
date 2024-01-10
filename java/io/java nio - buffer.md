### java nio - buffer
java nio 的buffer可同時用於讀與寫, 其使用上可以看成專門用來交換資料使用, <br>
NIO處理過程的核心就是把資料在buffer中搬進搬出, 與一般BufferedInputStream的差異在於 <br>
BufferedInputStream沒辦法接觸到其buffer, 且nio buffer 將其meta data封裝載物件中, 使用上更靈活 <br>

java.nio.Buffer 是Java NIO buffer的高階抽象這裡定義了很多共同的概念, 比如說: position、limit、capacity、flipping、marking以及 rewinding <br>

java.nio.Buffer 也有很多對應基本型態的子類別
* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

因為所謂資料其本質都是binary, 所以ByteBuffer算是裡面比較重要的, 其餘的都是根據不同的基本型態提供方便的介面去操作而已 <br>

#### Creating Buffers
兩種方式去建構, 透過第二種去建構的操作array等同於變更buffer內的資料
1. 指定容量: ByteBuffer.allocate(6);
2. 提供事先建立好的陣列: ByteBuffer.wrap(backingArray);

#### 三種 Buffer: Direct、Non-Direct、MMAP
* Non-Direct, 透過上面的兩種方式建立的都是屬於這類, 這類buffer其記憶體是由JVM去控管, 實體為 HeapByteBuffer
* Direct, 使用ByteBuffer.allocateDirect(6);建立, 這類buffer屬於Heap之外, 可以更快速的讓native程式去存取 (使用c語言的malloc分配堆外記憶體), 實體為DirectByteBuffer
* MappedByteBuffer, FileChannel.map()返回的就屬這類的buffer, 同樣也是Heap之外實際上是mmap()的封裝, 利用虛擬記憶體技術操作內核buffer (詳情參考[kernel buffer、mmap、virtual memory相關知識](zero-copy.md))

要判斷一個buffer是否為堆內可呼叫array()嘗試取得所持有的'backing array', 反之會發生UnsupportedOperationException <br>

#### Buffer State
* Capacity: 代表一個buffer的容量, buffer一建立後Capacity就不會再改變
* Limit: 是用來判斷第一個**不能**被讀\寫的資料, 0與limit之間的資料是可以被讀取到的, limit與 capacity之間的資料則是垃圾
* Position: 是用來判斷下一個資料是否可以讀\寫, 當資料讀寫時position會遞增
* Mark: 就是一個會被記憶住的位置, 呼叫mark()時會將目前position的值給mark, 呼叫reset()時會把mark的值給position以利後續操作

另外還有一個remaining()方法可以得知剩下可以消耗的資料數量 (limit - position), 也可以呼叫hasRemaining()得知是否還有資料可以消耗 <br>


#### Data Writing、Reading
buffer 讀寫資料的範例如下, 在讀寫時position、limit扮演關鍵的角色, 隨著持續讀\寫 position會被遞增

```java
    @SneakyThrows
    public static void main(String[] args){
        ByteBuffer byteBuffer = ByteBuffer.allocate(6);

        byteBuffer.put((byte) 0x00);
        System.out.printf("writing, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.put((byte) 0x01);
        System.out.printf("writing, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.put((byte) 0x02);
        System.out.printf("writing, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.put((byte) 0x03);
        System.out.printf("writing, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.put((byte) 0x04);
        System.out.printf("writing, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        System.out.printf("before flip, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.flip();
        System.out.printf("after flip, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        while (byteBuffer.hasRemaining()){
            System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));
            System.out.printf("reading, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        }

        byteBuffer.clear();
        System.out.printf("after clear, position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
    }
```
輸出為
```text
writing, position: 1, limit: 6, capacity: 6
writing, position: 2, limit: 6, capacity: 6
writing, position: 3, limit: 6, capacity: 6
writing, position: 4, limit: 6, capacity: 6
writing, position: 5, limit: 6, capacity: 6
before flip, position: 5, limit: 6, capacity: 6
after flip, position: 0, limit: 5, capacity: 6
0x00
reading, position: 1, limit: 5, capacity: 6
0x01
reading, position: 2, limit: 5, capacity: 6
0x02
reading, position: 3, limit: 5, capacity: 6
0x03
reading, position: 4, limit: 5, capacity: 6
0x04
reading, position: 5, limit: 5, capacity: 6
after clear, position: 0, limit: 6, capacity: 6
```

#### Buffer Life Cycle: Fill、Flip、Drain and Clear
nio buffer基本上是一個可以用來做資料交換的物件, 可讀寫的情況下具有兩種模式
1. Filling Mode: 寫模式, 由一個生產者寫入buffer
2. Draining Mode: 讀模式, 由一個消費者讀取buffer資料

一般來說, buffer一開始為空, 接著由生產者寫入資料, 這時是Filling Mode, <br>
寫入完畢後呼叫flip()切換為讀模式, 由消費者接著讀取, 呼叫clear()回復至初始狀態, 範例如上 <br>

#### Compact the Buffer
有些情況下, 可能你只會從buffer讀出部分資料, 接著再繼續往裡面填充, <br>
在這種情況下會希望buffer剩餘的資料可以往前挪移, 這時就會需要compact() (可以想像成一個FIFO的Queue) <br>
呼叫compact()時內部操作如下
1. 把剩餘資料複製到開頭
2. position設定為剩餘的數量
3. limit設定為capacity
4. 清除mark

ps. 呼叫compact()後如果想繼續讀需要先flip

```java
    @SneakyThrows
    public static void main(String[] args){
        ByteBuffer byteBuffer = ByteBuffer.allocate(6);
        byteBuffer.put((byte) 0x00);
        byteBuffer.put((byte) 0x01);
        byteBuffer.put((byte) 0x02);
        byteBuffer.put((byte) 0x03);
        byteBuffer.put((byte) 0x04);
        System.out.printf("before get position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.flip();
        byteBuffer.get();
        byteBuffer.get();
        byteBuffer.get();
        System.out.printf("after get position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        byteBuffer.compact();
        System.out.printf("after compact position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        byteBuffer.flip();
        System.out.printf("after flip position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));
        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));
    }
```
輸出為
```text
before get position: 5, limit: 6, capacity: 6
after get position: 3, limit: 5, capacity: 6
after compact position: 2, limit: 6, capacity: 6
after flip position: 0, limit: 2, capacity: 6
0x03
0x04
```


#### Marking the Buffer
buffer在讀取中有讓你回到之前位置的手段, 可以呼叫mark()記憶位置, 呼叫reset回到之前記憶的位置, <br>
mark: 設定mark為目前position
reset: 將position設定為mark
clear: 實際並不清理資料, 只不過將這些標記回復初始狀態
InvalidMarkException: 如果沒有呼叫過mark()就呼叫reset(), 此時mark為-1會拋出InvalidMarkException

```java
    @SneakyThrows
    public static void main(String[] args) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(6);
        byteBuffer.put((byte) 0x00);
        byteBuffer.put((byte) 0x01);
        byteBuffer.put((byte) 0x02);
        byteBuffer.put((byte) 0x03);
        byteBuffer.put((byte) 0x04);

        byteBuffer.flip();

        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));
        System.out.printf("mark position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        byteBuffer.mark();
        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));
        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));

        byteBuffer.reset();
        System.out.printf("reset after flip position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
        System.out.println("0x" + HexBin.encode(new byte[]{byteBuffer.get()}));

    }
```

輸出為

```text
0x00
mark position: 1, limit: 5, capacity: 6
0x01
0x02
reset after flip position: 1, limit: 5, capacity: 6
0x01
```

#### Rewind Buffer
如果想要重新讀取buffer可以呼叫rewind(), 會把position變為0, limit不變, 但mark變為-1 <br>

```java
    @SneakyThrows
    public static void main(String[] args) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(6);
        byteBuffer.put((byte) 0x00);
        byteBuffer.put((byte) 0x01);
        byteBuffer.put((byte) 0x02);
        byteBuffer.put((byte) 0x03);
        byteBuffer.put((byte) 0x04);

        byteBuffer.flip();

        byteBuffer.get();
        byteBuffer.get();
        byteBuffer.get();

        System.out.printf("before rewind position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());

        byteBuffer.rewind();

        System.out.printf("after rewind position: %d, limit: %d, capacity: %d%n", byteBuffer.position(), byteBuffer.limit(), byteBuffer.capacity());
    }
```
輸出為
```text
before rewind position: 3, limit: 5, capacity: 6
after rewind position: 0, limit: 5, capacity: 6
```