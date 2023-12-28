### Java IO 簡介

#### 什麼是IO
IO, 也就是輸入(In)與輸出(Out), 指應用程式與外部的資料傳輸 <br>
Java透過Stream處理IO, Stream是一個抽象概念指一連串的資料(byte)已先進先出的方式發送的通道 <br>
程式需要讀取或輸出資料時會先打開一個通向資料源的Stream, 資料源可以是檔案、記憶體、網路連接等等 <br>
* Stream的特性
  * 先進先出
  * 順序存取 (RandomAccessFile除外)
  * 只讀或只寫


#### Stream分類
輸入流與輸出流通常分為兩類(byte、character), 核心又分四個抽象類 (InputStream、OutputStream、Reader、Writer)


* character stream
  * Reader
    * CharArrayReader
    * PipedReader
    * FilterReader
      * PushbackReader
    * BufferedReader
    * InputStreamReader
      * FileReader
  * Writer
    * CharArrayWriter
    * PipedWriter
    * FilterWriter
    * BufferedWriter
    * OutputStreamWriter
      * FileWriter
    * PrintWriter

* byte Stream
  * InputStream
    * ByteArrayInputStream
    * PipedInputStream
    * FilterInputStream
      * BufferedInputStream
      * DataInputStream
    * FileInputStream
    * DataInputStream
    * ObjectInputStream
  * OutputStream
    * ByteArrayOutputStream
    * PipedOutputStream
    * FilterOutputStream
      * BufferedOutputStream
      * DataOutputStream
    * FileOutputStream
    * DataOutputStream
    * ObjectOutputStream

#### File Stream
檔案流分為兩類, 一類是character stream (FileReader、FilterWriter)另外是byte Stream (FileInputStream、FileOutputStream) <br>

```java
//FileInputStream範例
@SneakyThrows
public static void main(String[] args){

        int len = 0;
        byte[] buf = new byte[2];

        String filePath = "/Users/xiaoweiyun/test.txt";

        try (FileInputStream fis = new FileInputStream(filePath)) {
            while ((len = fis.read(buf)) != -1){
                System.out.print(new String(buf, 0, len, StandardCharsets.UTF_8));
            }
        }
}
```

```java
//FileOutputStream範例
@SneakyThrows
public static void main(String[] args){

        int len = 0;
        byte[] buf = new byte[2];

        String filePath = "/Users/xiaoweiyun/test.txt";
        String outputFilePath = "/Users/xiaoweiyun/test2.txt";

        try (FileInputStream fis = new FileInputStream(filePath);
        FileOutputStream fos = new FileOutputStream(outputFilePath)) {

          while ((len = fis.read(buf)) != -1){
            fos.write(buf, 0, len);
          }
        }
}
```

```java
//FileReader範例, 程式碼差異不大, 一個是byte一個是char (已解碼)
@SneakyThrows
public static void main(String[] args){

        int len = 0;
        char[] cbuf = new char[2];

        String filePath = "/Users/xiaoweiyun/test.txt";

        try(FileReader fr = new FileReader(filePath)) {
          while ((len = fr.read(cbuf)) != -1){
            System.out.print(new String(cbuf, 0, len));
          }
        }
}
```

```java
//FileWriter範例, 程式碼差異不大, 一個是byte一個是char (已解碼)
    @SneakyThrows
    public static void main(String[] args){

        int len = 0;
        char[] cbuf = new char[2];

        String filePath = "/Users/xiaoweiyun/test.txt";
        String outputFilePath = "/Users/xiaoweiyun/test2.txt";

        try(FileReader fr = new FileReader(filePath);
            FileWriter fw = new FileWriter(outputFilePath)) {
            while ((len = fr.read(cbuf)) != -1){
                fw.write(cbuf, 0, len);
            }
        }

    }
```

#### Array Stream
將資料寫入記憶體內, 特點是可以反覆讀取, 缺點是若資料太大會造成OOM 
```java
    @SneakyThrows
    public static void main(String[] args) {

        byte[] buf = new byte[64];

        try (ByteArrayInputStream bai = new ByteArrayInputStream("ABC123456".getBytes())) {

            int len = bai.read(buf);
            System.out.println(new String(buf, 0, len));

            bai.reset();

            len = bai.read(buf);
            System.out.println(new String(buf, 0, len));

        }
    }
```

```java
    @SneakyThrows
    public static void main(String[] args) {

        byte[] data = "ABC123456".getBytes();

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(data, 0, data.length);
        System.out.println(new String(baos.toByteArray(), StandardCharsets.UTF_8));

    }
```

#### Piped Stream (PipedInputStream、PipedOutputStream)
相同的process內不同的thread之間可以透過管道通訊, 已一個實際的案例來舉例的話 <br>
假設有一個很大的報表需要下載, 通常作法是先儲存到本地端然後再上傳到雲端的儲存系統內, 生成連結給用戶下載 <br>
可以改用管道流去處理, 生成報表與上傳作業可以同時進行, 兩者業務由不同的thread負責.
```java
    @SneakyThrows
    public static void main(String[] args){

        try(PipedInputStream pis = new PipedInputStream();
            PipedOutputStream pos = new PipedOutputStream(pis)){

            new Thread(()->{
                try {
                    pos.write("ABC123456".getBytes(StandardCharsets.UTF_8));
                    pos.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }).start();

            int len = 0;
            byte[] buf = new byte[2];
            while ((len = pis.read(buf))!=-1){
                System.out.print(new String(buf, 0, len));
            }

        }

    }
```

#### Data Stream
可以讀寫基本類型, 讀取順序需與寫入順序一致否則會報錯

```java
    @SneakyThrows
    public static void main(String[] args) {

        String filePath = "/Users/xiaoweiyun/testData.txt";

        try(FileOutputStream fos = new FileOutputStream(filePath);
            DataOutputStream dos = new DataOutputStream(fos)){

            dos.writeBoolean(true);
            dos.flush();
            dos.writeUTF("ABC123456");
            dos.flush();
            dos.writeInt(100);
            dos.flush();
        }

        try(FileInputStream fis = new FileInputStream(filePath);
            DataInputStream dis = new DataInputStream(fis)){
            System.out.println(dis.readBoolean());
            System.out.println(dis.readUTF());
            System.out.println(dis.readInt());
        }

    }
```
```java
    @SneakyThrows
    public static void main(String[] args) {
        String filePath = "/Users/xiaoweiyun/testData2.txt";

        try(FileOutputStream fos = new FileOutputStream(filePath);
            ObjectOutputStream oos = new ObjectOutputStream(fos)){
            
            oos.writeObject(new Test("aabb", 30));
        }

        try(FileInputStream fis = new FileInputStream(filePath);
            ObjectInputStream ois = new ObjectInputStream(fis)){

            System.out.println((Test)ois.readObject());

        }

    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Test implements Serializable {
        private String name;

        private int age;
    }
```

#### Buffered Stream
簡單來說緩衝流會在實作中設定一個緩衝區, 當緩衝區滿了或空了才會與設備交互, 大幅減少io交互次數
```java
//用法都差不多
@SneakyThrows
public static void main(String[] args) {
        String filePath = "/Users/xiaoweiyun/test.txt";

        int len = 0;
        byte[]buf = new byte[2];

        try(FileInputStream fis = new FileInputStream(filePath);
            BufferedInputStream bis = new BufferedInputStream(fis)){

            while ((len = bis.read(buf)) != -1){
                System.out.print(new String(buf, 0, len));
            }
        }
}
```

#### Print Stream
實際上System.out.print 的 out 就是 PrintStream, 
比較常用的建構子如下
* PrintStream(OutputStream)
* PrintStream(OutputStream, boolean autoFlush)
* PrintStream(File outputFile)
* PrintStream(String outputFileName)

```java
@SneakyThrows
public static void main(String[] args) {

        String filePath = "/Users/xiaoweiyun/testPrintStream.txt";

        try(FileOutputStream fos = new FileOutputStream(filePath);
        java.io.PrintStream ps = new java.io.PrintStream(fos)){

          ps.println("ABC123456");
          ps.println(1);
          ps.print(true);
          ps.println("GGGGGGGG");
          ps.flush();
        }
}
```