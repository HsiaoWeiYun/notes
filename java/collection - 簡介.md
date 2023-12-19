## Java Collection 簡介


#### List、Set、Queue、Map 的區別
* List
  * 有序
  * 可重複
  * 隨機訪問效能高
* Set
  * 元素不可重複
* Queue
  * 先進先出
  * 有序
  * 可重複
* Map
  * 可以使用Key找到Value
  * Key無序、不可重複
  * value無序、可重複
  * 每個key最多mapping到一個value

#### 底層資料結構

* List
  * ArrayList: array
  * Vector: array
  * LinkedList: doubly linked list

* Set
  * HashSet: 底層用HashMap實作
  * LinkedHashSet: 底層用LinkedHashSet實作 (維護了插入的順序)
  * TreeSet: 紅黑樹 (利用元素的排序維護順序, 不允許null)

* Queue
  * PriorityQueue: array實做binary heap
  * ArrayQueue: array+雙指針

* Map
  * HashMap: Java 8 之前是由array + linked-list組成的, java 8 後改為當鏈長度大於8時轉換為紅黑樹
  * LinkedHashMap: 繼承自HashMap以此基礎上增加doubly linked list結構, 此結構可以維護key的插入順序
  * Hashtable: array (本體) + linked-list (解決衝突)組成
  * TreeMap: 紅黑樹

<br>

#### List
<hr>

#### ArrayList 與 Array 的差別
* ArrayList會根據實際儲存的元素動態擴容, 而Array建立後就不能改變容量了
* ArrayList允許使用泛型來確保型別安全, Array則不行
* ArrayList只能儲存物件, 而Array沒有這個限制基本類型也能存

#### ArrayList 與 Vector 的差別
* ArrayList底層使用Array實作, 適用於頻率較高的查找工作, non-threadsafe
* Vector是List的古老實現, 使用Array實作, threadsafe

#### Vector 與 Stack 的差別
* Vector 與 Stack 都使用synchronized達到threadsafe
* Stack繼承Vector, 順序為FILO, Vector是一個列表

#### ArrayList新增與刪除的時間複雜度
* 頭部插入: O(n), 因需要將頭後面的元素往後移動一個位置
* 尾部插入: 為達容量上限時為O(1), 只要往尾部新增一個元素即可, 若容量不足需將現有元素複製到新的array然後再向尾部插入元素, O(n)
* 指定位置插入: 指定位置之後的元素需要向後移動一個位置然後才將元素放入指定位置, O(n)
* 頭部刪除: 同頭部插入的概念, O(n)
* 尾部刪除: 直接移除尾部元素, O(1)
* 指定位置刪除: 同指定位置插入概念, O(n)

#### Linked-List新增與刪除的時間複雜度
* 頭部插入刪除: 移動頭的指針即可, O(1)
* 尾部插入刪除: 移動尾的指針即可, O(1)
* 指定位置插入刪除: 先從頭部依序訪問到指定位置在移動對應指針指向, O(n)

#### RandomAccess interface
* 該interface是一個標記型interface, 代表實現的物件支援隨機訪問 (隨機訪問效能高, 可透過索引快速訪問)
* ArrayList有實現RandomAccess, LinkedList沒有 (因為是不連續的記憶體位址只能透過指針訪問)

<br>

#### Queue
<hr>

#### Queue 與 Deque的區別
Queue是單向列隊, 只能從一端插入元素另一端刪除元素, 遵守FIFO
Queue實現了Collection interface, 會因為容量問題而導致的操作失敗處理方式不同而分為兩類方法, 一種是操作失敗後會拋出異常一種是返回false

| 方法描述 | 拋出異常      | 返回true/false |
|------|-----------|--------------|
| 插入隊尾 | add(E e)  | offer(E e)   |
| 刪除隊首 | remove    | poll()       |
| 查詢隊首 | element() | peek         |

Deque是雙向列隊, 在列隊的兩端均可新增刪除, 同樣因容量限制失敗後的處理方式分為兩類 <br>
也有提供push() pop() 方法, 模擬Stack

| 方法描述 | 拋出異常          | 返回true/false    |
|------|---------------|-----------------|
| 插入隊首 | addFirst(E e) | offerFirst(E e) |
| 插入隊尾 | addLast(E e)  | offerLast(E e)  |
| 刪除隊首 | removeFirst() | pollFirst()     |
| 刪除隊尾 | removeLast()  | pollLast()      |
| 查詢隊首 | getFirst()    | peekFirst()     |
| 查詢隊尾 | getLast()     | peekLast        |

#### PriorityQueue
PriorityQueue 是在java 1.5後新增的, 與Queue的差異在於列隊順序是與優先級相關, 優先級高的先出隊
* PriorityQueue使用binary heap 實現
* PriorityQueue因基於binary heap故新增與刪除落在O(log n )
* non-threadsafe, 不支援null and non-comparable 的物件

#### BlockingQueue
BlockingQueue 是一個 interface, 特點是當列隊沒有元素時會一直阻塞直到有元素, 若列隊已滿則等到有空間才有辦法放入新元素 <br>
一般用於生產者 - 消費者 模型

BlockingQueue 有幾下幾種實作
* ArrayBlockingQueue: 用array實現的阻塞列隊, 在建立時需指定大小, 支持公平與非公平鎖的訪問機制
* LinkedBlockingQueue: 用linked-list實做, 也可以在建立時指定大小(有界), 不指定則為無界 (Integer.MAX_VALUE), 支持公平與非公平鎖的訪問機制
* PriorityBlockingQueue: 支援優先級的阻塞列隊, 原蘇必須實作Comparable或指定Comparator, 不支援null
* SynchronousQueue: 同步列隊是一種不儲存元素的組塞列隊, 每個插入操作都必須等待對應的刪除操作, 反之刪除操作也須等待對應的插入操作, 通常用於thread之間的資料傳遞
* DelayQueue: 延遲列隊, 元素只有到了指定延遲時間才能夠從列隊移出

#### ArrayBlockingQueue 與 LinkedBlockingQueue 有什麼區別
* ArrayBlockingQueue基於array LinkedBlockingQueue基於linked-list
* ArrayBlockingQueue為有界, LinkedBlockingQueue可以為有界或無界
* ArrayBlockingQueue為共用鎖也就是生產者與消費者共用一把鎖, 而LinkedBlockingQueue是拆開的分為putLock、takeLock, 這樣可以防止生產者與消費者之間的鎖競爭
* ArrayBlockingQueue需提前分配記憶體而 LinkedBlockingQueue不用
