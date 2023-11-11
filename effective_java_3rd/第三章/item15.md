### 第15條: 最小化class and member 的可訪問性 (封裝)

* 封裝也是隱藏資訊, 對於外部組件而言是否隱藏了實作細節, 從而達到解耦、獨立開發、測試、優化的優點
  <br> <br>
* java提供了 access control機制, 提供class nad member 可訪問性, 原則很簡單 '盡量使類或成員不可被外部訪問'
  * private: 只有宣告這個成員的class可訪問
  * package-private (default): 宣告成員的class所屬的package內的class都可訪問
  * protected: 宣告成員的class與其子class可訪問, 同一package的class也能訪問
  * public: 任何class都可訪問
<br> <br>
* public api 對於class來說因是公開給其他組件用具有長期維護的責任
<br> <br>
* protected api 對於class來說等於是承諾了內部實作, 對相依class而言也需要長期維護
<br> <br>
* 如果一個子類覆蓋了父類的方法則期訪問級別就不可以低於父類, 這是保證使用父類實例的地方也可使用子類, 同時也滿足 '里氏替換原則'
<br> <br>
* 當成員被宣告為public即代表你失去了對他的控制能力, 即使宣告為final也一樣, 因為雖不能改變指向但能改變指向物件的值, 若有需求應該如下這樣做
```java
    private static final Data[] PRIVATE_DATA = {...};
    public static final List<Data> DATA = Collections.unmodifiableList(Arrays.asList(PRIVATE_DATA));
```

