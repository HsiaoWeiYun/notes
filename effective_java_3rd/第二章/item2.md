### 第二條: 物件參數過多使用builder模式取代建構子

* builder優點: 使用上簡潔、靈活、不容易出錯
* builder缺點: 新增參數時inner class也要一同新增、需要額外建立builder class

```java
package com.hsiaoweiyun.designpatterns.builder;

public class Test {

    private final int p1;
    private final int p2;
    private final int p3;

    private Test(Builder builder) {
        p1 = builder.p1;
        p2 = builder.p2;
        p3 = builder.p3;
    }

    public int getP1() {
        return p1;
    }

    public int getP2() {
        return p2;
    }

    public int getP3() {
        return p3;
    }

    public static class Builder {

        private int p1;
        private int p2;
        private int p3;

        public Builder p1(int val) {
            p1 = val;
            return this;
        }

        public Builder p2(int val) {
            p2 = val;
            return this;
        }

        public Builder p3(int val) {
            p3 = val;
            return this;
        }

        public Test build() {
            return new Test(this);
        }

    }

    public static void main(String[] args) {
        Test test = new Builder().p1(1).p2(2).p3(3).build();
        System.out.println(test.getP1());
        System.out.println(test.getP2());
        System.out.println(test.getP3());
    }


}

```

書上還有提到也能用在'物件層次結構上', Pizza.Builder泛型中使用了recursive type parameter的概念, <br>
並應用了模擬self的技巧

```java
package com.hsiaoweiyun.designpatterns.builder;

import java.util.EnumSet;
import java.util.Set;

public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(topping);
            return self();
        }

        abstract Pizza build();

        protected abstract T self();
    }

    public Pizza(Builder<?> builder) {
        this.toppings = builder.toppings.clone();
    }
}

```

```java
package com.hsiaoweiyun.designpatterns.builder;

import java.util.Objects;

import static com.hsiaoweiyun.designpatterns.builder.Pizza.Topping.ONION;
import static com.hsiaoweiyun.designpatterns.builder.Pizza.Topping.PEPPER;

public class NyPizza extends Pizza {

    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {

        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        Pizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    public NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    public static void main(String[] args) {
        Pizza nyPizza = new NyPizza.Builder(Size.SMALL).addTopping(PEPPER).addTopping(ONION).build();
    }

}

```