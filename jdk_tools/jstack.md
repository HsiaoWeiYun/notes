### jstack (針對java process生成thread快照)

* 用法: jstack [ options ] pid (pid可透過jps指令查詢)
* options
  * -F: 當無法應答請求時強制產生快照
  * -l: 額包含額外鎖的資訊
  * -m: 如果有呼叫本地方法也會包含c/c++ stack trace

* 範例: jstack -F -l pid >> thread_dump.txt

#### 實際操作, 無窮迴圈為例

1. 先找出java process (jps)
2. 找出該process消耗最多cpu資源的thread (top -Hp pid)
3. 將pid轉為16進制 (printf "%x\n" pid)
4. 執行thread dump (jstack -F -l pid >> thread_dump.txt)
5. 針對文件內容搜尋16進制的thread id 找到stack trace
6. java 範例

```java
 private Object lock = new Object();

    public class Task implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                runrun();
            }
        }

        private void runrun(){
            int i = 0;
            while (true){
                i++;
            }
        }

    }

    private Executor executor = Executors.newFixedThreadPool(6);

    @GetMapping("/loop")
    public String loop() throws InterruptedException {

        Task t1 = new Task();
        Task t2 = new Task();

        executor.execute(t1);
        executor.execute(t2);


        return "success";
    }
```

* Lock State
  * locked: 代表使用synchronized申請鎖成功
  * waiting to lock: 代表申請鎖未成功, 進入等待區
  * waiting on: 代表申請鎖成功後執行wait()後進入等待區