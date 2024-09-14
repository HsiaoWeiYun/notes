### CAS (Compare And Swap)

#### 如何保證原子
常見的做法就是加鎖, 常見的做法有兩種
1. [synchronized](synchronized的原理與應用.md): 可以說是悲觀鎖, 只允許一條thread進入臨界區間
2. CAS: 樂觀鎖, 執行時會先假設不會有衝突所以不會加鎖, 如果有衝突就重試, 直到成功為止.

ps. 悲觀鎖多用於寫多讀少的場景; 樂觀鎖多用於讀多寫少的場景, 避免頻繁自旋消耗大量cpu

#### 什麼是CAS
在CAS實作中有三個變數, 分別是 V、E、N
* V: 待更新的變數
* E: 預期值 (舊值)
* N: 新值

過程如下: <br>
判斷V**是否等於**E, 如果等於就把V設定為N; 如果不等於就放棄更新
1. 假設有一個多執行續的共享變數i, i原本等於5, A thread 想變為6
2. 將i (V)與5 (E)進行比對, 結果相等, 代表並沒有其他thread競爭, 則把i設定為6 (N), CAS成功
3. 將i (V)與5 (E)進行比對, 結果不相等, i已經變為2了, 則放棄此次更新什麼也不做, 此次CAS失敗, i = 2

ps. 有可能發生正準備要更新i的時候i的值被改變的情況嗎?  答案是不會的, 因為CAS是原子操作, 是利用cpu 指令保證原子性


#### CAS原理
在java中有一個**unsafe**類別, 裡面有一些native的CAS方法, 由C++負責實現
```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

ps. linux x86下主要是透過**cmpxchgl**這個指令在CPU上完成CAS操作, 不過多cpu的情況會需要使用**lock**指令加鎖, 不同作業系統與處理器的實現方式都不一樣.

#### CAS的實際使用
以AtomicInteger.getAndAdd(int delta) 為例
```java

// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();

public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
```
unsafe 的 getAndAddInt 方法
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```
分析:
* Object var1 代表待操作的物件
* long var2 代表var1的偏移量, 可以通過Unsafe.objectFieldOffset 取得
* int var4 想要增加的值

執行過程: 
1. do ... while 內 先用 getIntVolatile 取得當前物件指定字段, 存入var5, getIntVolatile 可以保證操作可見性以及禁止指令重排 (因為是[volatile](../jmm/volatile的特性.md))
2. 執行compareAndSwapInt(var1, var2, var5, var5 + var4) 進行CAS操作, 如果物件var1在位置var2處的值等於var5則設定新值var5 + var4, 並返回true否則返回false
3. 如果為true則while loop 結束, CAS成功, 成功更新為var5 + var4
4. 如果為false則繼續嘗試, 重新取得var5並繼續CAS操作

#### CAS的三大問題
1. ABA問題: 就是一個值從A變成B然後又變成A, 這時候CAS檢查不出來數值已經被更新了兩次 <br>
   解法: 在數值前追加版本號碼或時間戳, ex: AtomicStampedReference, 這個類別的compareAndSet的作用是先檢查當前引用是否等於預期引用且當前標誌是否等於預期標誌, 若都相等才執行CAS <br>
```java
public boolean compareAndSet(V   expectedReference,
                              V   newReference,
                              int expectedStamp,
                              int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
          casPair(current, Pair.of(newReference, newStamp)));
}
```
流程:
* Pair<V> current = pair; 取得當前pair, 內含引用與標記
* 接下來的return做了幾個檢查
  * expectedReference == current.reference && expectedStamp == current.stamp: 檢查當前引用與標記是否與期望的相等, 有任何不同就false
  * 如果通過上述檢查, 就檢查新的引用與和標記是否與現用的相同, 如果相同就直接返回沒必要更新
  * 如果不同就執行casPair來更新pair, 用Pair.of(newReference, newStamp)取代舊的Pair



2. 長時間自旋: 長時間的自旋會消耗大量cpu, 解決方法是讓JVM支援使用**cpu pause指令**, CAS失敗開始自旋時讓cpu小睡一段時間再接續自旋, 降低操作頻率 (也可解決因記憶體順序衝突導致的cpu流水線重排問題).


3. 多個變數的原子操作: CAS一次只能操作一個變數, 當有多個變數需要原子操作時就無法保證原子性
   1. 解法1: 使用鎖, 將需要原子的操作包在臨界區間內, 這樣可以保證只有當前thread可以操作
   2. 解法2: 使用AtomicReference這類的物件, 將多個變數封裝成一個物件內進行CAS操作