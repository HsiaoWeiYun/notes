### 第一條: 用靜態工廠方法代替建構子

這邊說的靜態工廠方法不等於Design Pattern中的工廠模式 <br>

ex: 
```java
public static String valueOf(boolean b) {
        return b ? "true" : "false";
    }
```

* 靜態工廠不同於建構子的第一優勢: 有名稱, 能更清楚描述行為 (ex: BigInteger.probablePrime)
* 靜態工廠不同於建構子的第二優勢: 不用每次呼叫建立一個新物件
* 靜態工廠不同於建構子的第三優勢: 可以返回原返回類型的子類, ex: EnumSet.noneOf
```java
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```
* 靜態工廠不同於建構子的第四優勢: 返回的物件可隨著參數不同而不同, <br>
ex: EnumSet.noneOf 未超過64返回RegularEnumSet其他返回JumboEnumSet
* 靜態工廠不同於建構子的第五優勢: 方法返回的物件的實作在方法撰寫時可以不存在, <br> 
ex: Service Provider Framework (JDBC、ServiceLoader)

<hr>

* 靜態工廠的第一個缺點: 如果物件不含public or protected的建構子則沒辦法被繼承 (是缺點也是優點)
* 靜態工廠的第二個缺點: 相對建構子工程師不好發現他們 (個人覺得不算什麼缺點)

<hr>

#### 命名慣例
* from, 類型轉換
```java
Date date = Date.from(instant);
```
* of, 聚合
```java
EnumSet<Word> words = EnumSet.of(A, B, C);
```
* valueOf, 比from和of更繁瑣的替代方法
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```
* instance、getInstance: 透過方法與參數返回實例
* create、newInstance: 同上, 但保證是一個新實例
* get${Type}: 類似getInstance, 但有指定類型
```java
FileStore fs = Files.getFileStore(path);
```
* new${Type}: 類似newInstance但有指定類型
```java
BufferedReader bf = Files.newBufferedReader(path);
```
* $Type: get${Type}、new${Type}的簡化版
```java
    public static <T> ArrayList<T> list(Enumeration<T> e) {
        ArrayList<T> l = new ArrayList<>();
        while (e.hasMoreElements())
            l.add(e.nextElement());
        return l;
    }
```

#### 結論: 靜態工廠方法與建構子各有優缺, 通常情況靜態工廠方法會更加適合 