### java nio - Selector

Selector是java nio的關鍵元件, 他允許一個Thread處理多個Channel. <br>

#### Selector主要運作原理
通過不斷輪詢註冊在Selector上的Channel, 當事件發生時 (比如說Channel上有新的連接或新的讀寫事件)Channel就緒, <br>
這時就會被Selector輪詢出來, Selector會將相關的Channel加入就緒集合內, 透過 'SelectionKey' 可以獲取就緒集合 <br>
接著再對這些就緒的Channel做相對應的IO操作. <br>
一個多路復用的Selector可以同時輪詢多個Channel, JDK使用'epoll()'代替傳統'select', 所以並沒有最大連接上限 (1024/2048) <br>
代表只要一個Selector就可以負責成千上萬的連接. <br>

#### Selector可以監聽下面四種事件
1. SelectionKey.OP_ACCEPT: 代表通道接受連接 
2. SelectionKey.OP_CONNECT: 代表通道完成連接
3. SelectionKey.OP_READ: 代表通道準備好讀取, 代表有資料可供讀取
4. SelectionKey.OP_WRITE: 代表通道準備好接受寫入資料

下面是Selector網路讀寫範例
```java
    @SneakyThrows
    public static void main(String[] args){
        try(ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            Selector selector = Selector.open()){

            serverSocketChannel.configureBlocking(false);
            serverSocketChannel.socket().bind(new InetSocketAddress(8888));

            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {
                //selector.select(); 監控所有已註冊channel, 當任一channel準備就緒才會返回, 並加入selectedKeys
                int readyChannels = selector.select();
                if (readyChannels == 0) {
                    continue;
                }

                System.out.println("");
                System.out.println("readyChannels: " + readyChannels);

                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

                while (keyIterator.hasNext()) {
                    SelectionKey key = keyIterator.next();

                    if (key.isAcceptable()) {
                        System.out.println("處理連接事件");
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        client.configureBlocking(false);

                        System.out.println("註冊讀事件");
                        client.register(selector, SelectionKey.OP_READ);
                    }else if (key.isReadable()) {
                        System.out.println("處理讀事件");
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int bytesRead = client.read(buffer);

                        System.out.println("bytesRead: " + bytesRead);
                        if (bytesRead > 0) {
                            buffer.flip();
                            System.out.println("接收到資料：" +new String(buffer.array(), 0, bytesRead));

                            System.out.println("註冊寫事件");
                            client.register(selector, SelectionKey.OP_WRITE);
                        }else if (bytesRead < 0) {
                            System.out.println("關閉連接");
                            client.close();
                        }
                    }else if (key.isWritable()) {
                        System.out.println("處理寫事件");
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.wrap("Hello, Client!".getBytes());
                        client.write(buffer);
                        System.out.println("已發送");

                        System.out.println("註冊讀事件");
                        client.register(selector, SelectionKey.OP_READ);
                    }
                    keyIterator.remove();
                }
            }

        }
    }
```

輸出
```text
readyChannels: 1
處理連接事件
註冊讀事件

readyChannels: 1
處理讀事件
bytesRead: 5
接收到資料：123

註冊寫事件

readyChannels: 1
處理寫事件
已發送
註冊讀事件

readyChannels: 1
處理讀事件
bytesRead: 5
接收到資料：456

註冊寫事件

readyChannels: 1
處理寫事件
已發送
註冊讀事件

readyChannels: 1
處理讀事件
bytesRead: 5
接收到資料：789

註冊寫事件

readyChannels: 1
處理寫事件
已發送
註冊讀事件

readyChannels: 1
處理讀事件
bytesRead: -1
關閉連接
```