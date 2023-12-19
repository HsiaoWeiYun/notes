### ArrayList 擴容機制解析 (java 11)


#### 建構子
<hr>

建立arraylist時指定大小, 會立即建立一個指定大小的array, 若指定大小為0則會暫時指定一個空array
```java
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
建立arraylist時不指定大小則會暫時指定一個空的array
```java
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
建立arraylist時傳入Collection物件此時會先將物件轉為array, 接著複製為新的array
```java
public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            //若同為ArrayList直接讓elementData指向轉換後的array就好
            if (c.getClass() == ArrayList.class) {
                elementData = a;
                //若為其他集合則將資料複製讓elementData指向
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

擴容機制

遞增modCount, 紀錄結構被變更的次數
```java
public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
```
傳入參數依序為: 期望新增的元素、目前的資料、容量 <br>
先檢查容量, 如果容量與元素個數一致則代表已滿, 開始擴容 <br>
擴容完畢後在把預期新增的元素新增進去
```java
private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
```
指定擴容最小容量, newCapacity會計算新的容量大小, 並將元素集複製到新的array內

```java
private Object[] grow() {
        return grow(size + 1);
    }

private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,
        newCapacity(minCapacity));
}
```
新的容量計算方式為: 原容量 + (原容量/2), 也就是 原容量的1.5倍 <br>
如果新的容量比最小容量還要小的話且不是下列的幾個情況就直接用最小容量作為擴充容量
1. 資料是否為初始化階段, 若是則直接DEFAULT_CAPACITY (10) 與 minCapacity 取其大值
2. 如果最小容量為負數代表overflow, 拋錯

<br>

如果新的容量沒有比Integer.MAX_VALUE - 8 還大則會直接採用, <br>
反之進入newCapacity判斷, 最大容量不會超過Integer.MAX_VALUE
<br>

當第1個元素新增時此時minCapacity為1, newCapacity為0, 會進入 if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) <br>
故新的容量為10 <br>
新增到第11個元素時minCapacity為11, newCapacity為15, 則容量為15

```java
private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```