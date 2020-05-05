# 原型模式

### 概括

> 用于创建重复的对象,当直接创建对象代价比较大时可以使用这种模式.主要通过对象克隆的方式创建新的对象.

### 应用场景及意义

1. 通过原型实例指定创建对象的种类,然后通过克隆该对象生成新的对象
2. 需要优化资源的场景可以考虑,创建对象非常繁琐,一个对象多个修改者场景,可以使用这种模式

### 代码以及实现

1. 创建一个实现 cloneable 的接口

```
package org.example.prototype;

public abstract class Fish implements Cloneable {
    private Integer id;
    protected String type;

    abstract void swim();

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getType() {
        return type;
    }

    @Override
    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

```

2. 编写接口的实现类

```
package org.example.prototype;

public class DolphinFish extends Fish {

    public DolphinFish() {
        type = "dolphin";
    }

    @Override
    public void swim() {
        System.out.println("海豚 愉快的游");
    }
}


package org.example.prototype;

public class HairtailFish extends Fish {

    public HairtailFish() {
        type = "hairtail";
    }

    @Override
    public void swim() {
        System.out.println(" 带鱼 弯弯曲曲的游");
    }
}
```

3. 编写原型工厂

```
package org.example.prototype;

import java.util.Hashtable;

public class FishCache {
    private static Hashtable<Integer, Fish> fishMap = new Hashtable<>();

    public static Fish getFish(Integer fishId) {
        Fish cacheFish = fishMap.get(fishId);
        return (Fish) cacheFish.clone();
    }

    public static void loadCache() {
        DolphinFish dolphinFish = new DolphinFish();
        dolphinFish.setId(1);
        fishMap.put(1, dolphinFish);

        HairtailFish hairtailFish = new HairtailFish();
        hairtailFish.setId(2);
        fishMap.put(2, hairtailFish);
    }
}
```

4. 
