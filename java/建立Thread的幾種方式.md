### 建立Thread的幾種方式


#### Process 是什麼
是對運行中應用程式的封裝, 是作業系統分配資源與調度的基本單元, process間不共享記憶體

#### Thread 是什麼
是Process的子任務, 一個process在執行中可以產生很多thread, 一個process內的thread彼此共享記憶體資源 <br>
每個thread都有自己的Stack、Native Stack、Program Counter, 比process更加輕量級

#### Process 與 Thread 之間的差異
1. 每個Process有自己的記憶體、資源, Thread可彼此之間共享
2. 建立Process比Thread負擔要更重, 因為Process有獨立資源, Thread為共享
3. Process之間要互相溝通會需要額外的機制 (IPC), Thread之間互相溝通相對簡單


#### java 建立 Thread的三種方式
* 繼承Thread, 複寫run() <br>
不建議, 因java不支援多重繼承, 彈性差

* 實現Runnable interface <br>
建議, 通常用於不需要取得回傳結果時使用

* 實現Callable interface <br>
建議, 通常用於需要取得回傳結果, 需要在未來判斷任務執行狀態時使用


```java
    @SneakyThrows
    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("t1: " + i);
                }
            }
        };

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("t2: " + i);
                }
            }
        });


        ExecutorService executorService = Executors.newSingleThreadExecutor();
        Future<String> future = executorService.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                TimeUnit.SECONDS.sleep(1);
                return "callable: " + System.currentTimeMillis();
            }
        });


        t1.start();
        t2.start();
        System.out.println(future.get());
        executorService.shutdown();
    }
```
#### Future
Future位於java.util.concurrent下, 是一個interface
```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```
一共宣告了五種方法
1. cancel: 用於取消任務, 取消成功返回true, 取消失敗返回false, mayInterruptIfRunning 代表是否取消正在執行的任務
   1. return true: 真的有任務被你取消了, 已經完成的不屬於這類
   2. return false: 任務並沒有被取消, 已完成、執行中且mayInterruptIfRunning=true 都屬這類
2. isCancelled: 是否被取消成功, 如果任務在完成前被取消則返回true
3. isDone: 是否完成
4. get(): 獲取執行結果
5. get(long timeout, TimeUnit unit): 若在指定時間內沒有成功獲取結果, 則返回null

提供了三種功能
1. 判斷任務是否完成
2. 中斷任務
3. 獲取執行結果

#### FutureTask
為Future的實現
```txt
public class FutureTask<V> implements RunnableFuture<V>
```
```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```
2種建構子
```java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}

```