### java nio - channel

Channel就像是一個通道, 他建立了資料源 (檔案、網路)之間的連接,<br>
與BIO不同, BIO 流就有分輸入輸出, Channel是全雙工的與[buffer](java%20nio%20-%20buffer.md)打交道 <br>

常用的通道
* FileChannel
* SocketChannel
* ServerSocketChannel
* DatagramChannel

下面是一個讀取一個檔案的範例
```java
    @SneakyThrows
    public static void main(String[] args) {
        String filePath = "/Users/xiaoweiyun/test.txt";

        ByteBuffer buf = ByteBuffer.allocate(2);

        try (RandomAccessFile raf = new RandomAccessFile(new File(filePath), "r");
             FileChannel fc = raf.getChannel()) {

            int len = 0;

            while ((len = fc.read(buf)) != -1) {
                buf.flip();
                System.out.print(new String(buf.array(), 0, len));
                buf.clear();
            }
        }
    }
```
利用[sendfile](zero-copy.md)指令將一個檔案輸出至另一個檔案
```java
    @SneakyThrows
public static void main(String[] args) {
        String filePath = "/Users/xiaoweiyun/test.txt";
        String outputPath = "/Users/xiaoweiyun/testOutput.txt";

        try (RandomAccessFile iraf = new RandomAccessFile(new File(filePath), "rw");
        FileChannel ifc = iraf.getChannel();
        RandomAccessFile oraf = new RandomAccessFile(new File(outputPath), "rw");
        FileChannel ofc = oraf.getChannel()) {
            ifc.transferTo(0, ifc.size(), ofc);
        }
}
```
