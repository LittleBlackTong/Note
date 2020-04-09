# 策略模式

### 概括
> 指一个类的行为或算法可以在运行时更更改
> 首先创建各种策略的对象, 然后创建一个行为随着策略对象改变而改变的 context 对象

### 应用场景及意义
1. 把类似的算法一个个的封装起来并且可以使他们进行替换
2. 解决类似算法执行时的if-else 带来的复杂度
3. 搬家可以用货车拉,也可以用自行车拉,每种方式都是一种策略

### 代码以及实现

1. 声明策略接口
```
package org.example.srategy;

public interface Strategy {
    public String testStrategy(String strategyName);
}
```
2. 实现策略接口的方法

```
package org.example.srategy;

public class StrategyOne implements Strategy {
    @Override
    public String testStrategy(String strategyName) {
        return strategyName + ":this is Strategy One";
    }
}


package org.example.srategy;

public class StrategyTwo implements Strategy {
    @Override
    public String testStrategy(String strategyName) {
        return strategyName + " this is StrategyTow";
    }
}
```
3. 声明行为类,负责执行策略

```
package org.example.srategy;

public class StrategyContext {
    private Strategy strategy;

    public StrategyContext(Strategy strategy) {
        this.strategy = strategy;
    }

    public String excuteStrategy(String name) {
        return strategy.testStrategy(name);
    }
}
```

4. 更具策略不同得到不同行为

```
package org.example.srategy;

public class StrategyContext {
    private Strategy strategy;

    public StrategyContext(Strategy strategy) {
        this.strategy = strategy;
    }

    public String excuteStrategy(String name) {
        return strategy.testStrategy(name);
    }
}

```
