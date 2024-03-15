# lambda 表达式的使用

> lambda 表达式概诉 () -> {} 对接口进行实现调用  “()” 中代表着接口的入参数 “{}” 中内容是接口的实现 当“()” 中的参数有切仅有一个时 “()” 本身可以省略
> 同理“{}”当方法实现有切仅有一行时"{}"本身也可以省略

### 使用实例

1. 声明接口

```
public interface PersonSayMethod {

	public void personSaySomthing(String val);
	
}
```

2. 使用接口

```
public class LambdaTest {

    public void PersonSaySomthing(String val, PersonSayMethod personSayMethod) {
        personSayMethod.personSaySomthing(val);
    }
	
	public static void main(String[] args) {
		//(1)历史接口使用方式
        LambdaTest lambdaTest = new LambdaTest();
        String somthing = "TongBlackLittle";
		//实现接口
		PersonSayMethod personSayMethod = new PersonSayMethod() {
            @Override
            public void personSaySomthing(String val) {
                System.out.println("hello lamdba:" + val);
            }
		}
		lambdatest.Personsaysomthing(somthing,personsaymethod);
		
		//(2)lambda 表达式接口使用方式-完整声明
		PersonSayMethod personsaymethod = (String val)->{System.out.println(val)};
		lambdatest.Personsaysomthing(somthing,personsaymethod);
		
		//(3)lambda 表达式接口使用方式-省略参数类型
		PersonSayMethod personsaymethod = (val)->{System.out.println(val)};
		lambdatest.Personsaysomthing(somthing,personsaymethod);
		
		//(4)lambda 表达式接口使用方式-省略括号
		PersonSayMethod personsaymethod = val->{System.out.println(val)};
		lambdatest.Personsaysomthing(somthing,personsaymethod);
		
		//(5)lambda 表达式接口使用方式-省略大括号
		PersonSayMethod personsaymethod = val->System.out.println(val);
		lambdatest.Personsaysomthing(somthing,personsaymethod);
		
		//(6) 最终版
		lambdatest.Personsaymethod(somthing,val->System.out.println(val))
		lambdaTest.PersonSaySomthing(somthing, System.out::println);
		
	}
}
```

