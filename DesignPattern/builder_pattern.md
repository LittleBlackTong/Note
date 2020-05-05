# 建造者模式

### 概括

> 将一个个简单的对象一步一步构造成一个复杂的对象,数据“创建型模式”,它提供了一种创建对象的最佳方式,Builder 类独立于其他的对象.

### 应用场景及意义

1. 将一个复杂的构建与其表示相分离,使得同样的构建过程可以创建不同的表示
2. 有时候面临剧烈的变化,通常由各个部分的自对象用一定的算法构成;由于需求的变化这个复杂对象的各个部分面临变化,但是组合在一起的算法却稳定
3. 建造者创建实例和工厂模式的区别在于,建造者更关注零件的装配顺序.

### 代码以及实现

1. 声明组合对象中基本对象的接口

```
package org.example.builder;

public interface Commodity {
	public String name();
	public float price();
	public Packing packing();
}


package org.example.builder;

public interface Packing {
	public String packing();
}
```
2. 编写基本对象的抽象父类的实现

```
package org.example.builder;

public abstract class Buger implements Commodity {

    @Override
    public Packing packing() {
        return new Wrapper();
    }

    @Override
    public abstract float price();
}

package org.example.builder;

public abstract class ColdDrink implements Commodity {
    @Override
    public Packing packing() {
        return new Bottle();
    }

    @Override
    public abstract float price();
}

```

3.编写基本对象的实现

```
package org.example.builder;

public class ChickenBuger extends Buger {

    @Override
    public String name() {
        return "Chicken Buger";
    }

    @Override
    public float price() {
        return 50.0F;
    }
}

package org.example.builder;

public class VegBuger extends Buger {
    @Override
    public String name() {
        return "Veg Buger";
    }

    @Override
    public float price() {
        return 25.0F;
    }
}

package org.example.builder;

public class Coke extends ColdDrink {
    @Override
    public String name() {
        return "Coke";
    }

    @Override
    public float price() {
        return 10F;
    }
}

package org.example.builder;

public class Pepsi extends ColdDrink {
    @Override
    public String name() {
        return "Pepsi";
    }

    @Override
    public float price() {
        return 15.0F;
    }
}

```

4. 编写复杂类的组装类

```
package org.example.builder;

import java.util.ArrayList;
import java.util.List;

public class Meal {

    private List<Commodity> commodityList = new ArrayList<>();

    public void addCommodity(Commodity commodity) {
        commodityList.add(commodity);
    }

    public float getCost() {
        float cost = 0F;
        for (Commodity commodity : commodityList) {
            cost += commodity.price();
        }
        return cost;
    }

    public void showCommodity() {
        for (Commodity commodity : commodityList) {
            System.out.println(commodity.name());
            System.out.println(commodity.packing());
            System.out.println(commodity.price());
        }
    }
}
```

5. 编写建造者类对复杂类进行组装

```
package org.example.builder;

public class BuilderMeal {

    public Meal prepareVegMeal() {
        Meal meal = new Meal();
        meal.addCommodity(new VegBuger());
        meal.addCommodity(new Pepsi());
        return meal;
    }

    public Meal prepareNonVegMeal() {
        Meal meal = new Meal();
        meal.addCommodity(new ChickenBuger());
        meal.addCommodity(new Coke());
        return meal;
    }
}
```

6. 通过建造者创建不同组合的复杂对象

```
package org.example.builder;

public class MainBuilder {
    public static void main(String[] args) {
        BuilderMeal builderMeal = new BuilderMeal();
        Meal meal = builderMeal.prepareVegMeal();
        meal.showCommodity();
        System.out.println("it is cost:" + meal.getCost());

        Meal chickenMeal = builderMeal.prepareNonVegMeal();
        chickenMeal.showCommodity();
        System.out.println("it is cost:" + chickenMeal.getCost());
    }
}
```
