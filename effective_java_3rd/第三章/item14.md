### 第14條: Comparable、Comparator

* 物件實現了Comparable代表具有內在的排序關係, 實現了的物件要排序就如下這麼簡單
```java
Collections.sort();
Arrays.sort();
```

* 實現Comparable.compareTo會要求回傳一個int, 當該物件小於、等於、或大於分別回傳負整數、零、正整數 (-1、0、1)

* 在compareTo方法中使用 '>' '<' 很容易出錯, java 7 之後幾個基本類型的物件都有靜態compareTo方法可使用
```java
Integer.compare();
Double.compare();
```

* 如果一個類有多個成員, 應從最關鍵的成員開始比較下去, 如果某成員產生非零結果則操作結束
```java
    @Override
    public int compareTo(A o) {
        //依序從關鍵的成員開始比較, 直到產生非零結果
        int ans = Integer.compare(result1, o.result1);
        if (ans == 0) {
            ans = result2.compareTo(o.result2);
            if (ans == 0) {
                return 0;
            }
        }
        return ans;
    }
```

* java 8 時 Comparator有了一系列的default方法, 可以構造一個Comparator來實現compareTo
```java
    private static final Comparator<B> COMPARATOR = Comparator.comparingInt((B b) -> b.result1).thenComparing((B b) -> b.result2);

    @Override
    public int compareTo(B o) {
        return COMPARATOR.compare(this, o);
    }
```

* 也可以單純在排序時指定Comparator
```java
Arrays.sort(obj, COMPARATOR);
```