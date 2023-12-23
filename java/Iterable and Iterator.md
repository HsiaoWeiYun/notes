### Iterable and Iterator

#### Iterable
Iterable代表這個物件具有迭代的能力, 所有的集合類揪實現Collection, 而Collection繼承Iterable interface, <br>
所以也就是說所有的集合都能被迭代. <br>

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

#### Iterator
Iterator代表迭代器, 如果實現了Iterable interface就會需要建立一個**內部類**去實現一個Iterator
```java
public interface Iterator<E> {
    
    boolean hasNext();

    E next();
}
```
使用Iterator迭代ArrayList 
```java
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");

        Iterator<String> its = list.iterator();

        while (its.hasNext()) {
            System.out.println(its.next());
        }
    }
}
```

然而foreach的語法糖其實也只不過是簡化版的iterator(), 因此能夠使用語法糖的都是有實現Iterable的物件

#### 為什麼Iterator會需要額外使用內部類去實現 (舉例ArrayList)
可以發現ArrayList並不是由本身去實作Iterator, 而是由內部類Itr去實現, 因此ArrayList的iterator()方法只是去建立一個ArrayList.Itr而已, <br>
這樣設計的用意是因為可能有多個迭代器去遍歷, 為了不彼此間後相干擾不能共用實體需維護各自的cursor

<br>

#### 遍歷時不要直接使用容器去刪除元素, 而是透過Iterator (舉例ArrayList)
下面程式執行時期會發生錯誤
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

for(String s : list){
    if(s.equals("B")){
        list.remove(s);
    }
}
```
是因為因為foreach語法糖實際是在操作Iterator, 這種機制可以確保每個元素都能被遍歷, <br>
Iterator.next()時會去執行 checkForComodification(), 去檢查結構是否被更改

```java
private class Itr implements Iterator<E> {
    int cursor;
    int lastRet = -1;
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
}
```

```java
final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
}
```

建立Itr時modCount就被指派給了expectedModCount, 當執行ArrayList.remove()時modCount的值被改變, <br>
Iterator.next() 檢查到不一致代表內容被變更過則拋出ConcurrentModificationException異常 <br>

應改為Iterator.remove(), cursor會被重置為上一個元素的位置就可以確保每個元素都能被遍歷
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

for(String s : list){
    if(s.equals("B")){
        Iterator.remove(s);
    }
}
```



