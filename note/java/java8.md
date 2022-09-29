



---
Java8 相关内容
- Lambda 表达式
- 函数式接口
- 方法引用 / 构造器引用
- Stream API
- 接口中的默认方法 / 静态方法
- 新时间日期 API
- 其他新特性


Java8 特点
- 速度更快
- 代码更少
- 强大的 StreamAPI
- 便于并行
- 最大化减少空指针异常(Optional)




---
# 一、Lambda 表达式
无参数、无返回值的语法格式: `() -> `
```java
// 无参数、无返回值的示例代码
public class TestLambda {
    public static void main(String[] args) {
        // 匿名内部类写法
        new Runnable(){
            @Override
            public void run() {
                //在局部类中引用同级局部变量
                //只读
                System.out.println("Hello World");
            }
        };
        // Lambda 表达式写法
        Runnable runnable = () -> System.out.println("Hello Lambda");
    }
}
```


有参数、有返回值的语法格式: 
```java
(a1, a2, ...) -> {
    System.out.println("Hello World");
    return a1;
}
```

```java
// 有参数、有返回值的示例代码: 
public class TestLambda {
    public static void main(String[] args) {
        // 匿名内部类
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1, o2);
            }

            @Override
            public boolean equals(Object obj) {
                return false;
            }
        };
        // 调用
        TreeSet<Integer> set = new TreeSet<>(comparator);
    }
}

// Java8 Lambda 表达式写法
public class TestLambda {
    public static void main(String[] args) {
        // Lambda 表达式
        Comparator<Integer> comparator = (o1, o2) -> Integer.compare(o1, o2);
        // 调用
        TreeSet<Integer> set = new TreeSet<>(comparator);

        System.out.println("===================或者==================");
        
        // 如果有多条记录, 需要加上 {}
        Comparator<Integer> comparator = (o1, o2) -> {
            System.out.println("Hello TestLambda");
            return Integer.compare(o1, o2);
        };
        // 调用
        TreeSet<Integer> set = new TreeSet<>(comparator);
    }
}
```


# 二、函数式接口
## 2.1 概念
函数式接口在 Java 中值: 有且仅有一个抽象方法的接口。

函数式接口适用于 **函数式编程**。Java 中的函数式编程体现就是 Lambda, 所以函数式接口就是可以适用于 Lambda 使用的接口。只有确保接口中有且仅有一个抽象方法，Java 中的 Lambda 才能顺利地进行推导。

## 2.2 格式
需要确保接口中有且只有一个抽象方法
```java
@FunctionalInterface
<权限修饰符> interface <接口名> {
    <返回值类型> <方法名称>(<入参>);
}
```
- `@FunctionalInterface`: 与 `@Override` 注解类型，Java8 中专门为函数式接口引入了这个注解；一旦使用该注解来定义接口，编译器会强制检查该接口是否确实有且仅有一个抽象方法，否则会报错。需要注意的是，即使不适用这个注解，只要满足函数式接口的定义，这个接口就仍然是一个函数式接口。


## 2.3 自定义函数式接口

```java
// ------------ define interface ------------
@FunctionalInterface
public interface MyInterface {
    Integer count(Integer a, Integer b);
}

// ------------ main -----------
public class TestMyInterface {
    
    public Integer operate(Integer a, Integer b, MyInterface myInterface) {
        return myInterface.count(a, b);
    }
    
    public static void main(String[] args) {
        MyInterface myInterface = (a, b) -> a + b;
        Integer result = operate(2, 99, myInterface);
        // 或者将上述两行写为
        Integer result2 = operate(2, 99, (a, b) -> a + b);
        System.out.println(result);
        System.out.println(result2);
    }
}
```

案例一: 使用 Collections.sort() 方法, 先比较工资，再比较年龄，使用 Lambda 表达式做参数传递。
```java
// ------------- define BOJO ------------
class Employee {
    private Integer id;
    private String name;
    private Integer age;
    private Double salary;
    
    // 省略 get/set 和 构造器
}

public class TestLambda1 {
    public static void main(String[] args) {
        List<Employee> emps = Arrays.asList(
            new Employee(1, "zhangsan", 39, 99999.0),
            new Employee(2, "lisi", 23, 5555.0),
            new Employee(3, "wangwu", 43, 3425.0),
            new Employee(4, "maliu", 19, 19921.0),
            new Employee(5, "tianqi", 26, 21101.0)
        );

        Collections.sort(emps, (e1, e2) -> {
            if (e1.getSalary() == e2.getSalary()) {
                return e1.getAge().compareTo(e2.getAge());
            } else {
                return Double.compare(e1.getSalary(), e2.getSalary());
            }
        });

        for (Employee emp : emps) {
            System.out.println(emp);
        }
    }
}
```

案例二: 声明函数式接口，接口中声明抽象方法: `String getValue(String val)`, 声明类 `TestLambda`, 在类中编写方法使用接口作为参数，将第一个字符串转换成大写，并作为方法的返回值。

```java
@FunctionalInterface
public interface MyInterface2 {
    String getValue(String val);
}

public class TestLambda {
    public static void main(String[] args) {
        String operateO = operate("KinO", (x) -> x.toUpperCase());
        System.out.println(operateO);
    }

    public static String operate(String x, MyInterface2 myInterface2) {
        return myInterface2.getValue(x);
    }
}
```

案例三: 声明一个带两个泛型的函数式接口，泛型类型为`<T, R>`, T 为参数，R 为返回值；接口中声明对应的抽象方法；在 TestLambda 类中声明方法，使用接口作为参数，计算两个 Long 类型参数的合；再计算两个 Long 类型参数的乘积。

```java
@FunctionalInterface
public interface MyInterface1<T, R> {
    R operate(T t);
}

public class TestLambda {
    public static void main(String[] args) {
        Long sum = operate(5L, (x) -> x + x);
        System.out.println("sum: "+sum);
        Long product = operate(5L, (x) -> x * x);
        System.out.println("product: "+product);
    }

    public static Long operate(Long x, MyInterface1<Long, Long> myInterface1) {
        return myInterface1.operate(x);
    }
}
```

## 2.4 内置函数式接口
| 函数式接口                | 参数类型 | 返回类型    | 用途                                                | 
|----------------------|------|---------|---------------------------------------------------| 
| Consumer</br> 消费类型接口 | T    | void    | 对类型为 T 的对象应用操作: void accept(T t)                  | 
| Supplier</br>提供型接口   | 无    | T       | 返回类型为 T 的对象: T get()                              |
| Function</br>函数型接口   | T    | R       | 对类型为 T 的对象应用操作，并返回结果为 R 类型的对象: R apply(T t)       |
| Predicate</br>断言型接口  | T    | boolean | 确定类型为 T 的对象是否满足某约束，并返回 boolean: boolean test(T t) |


## 2.5 Consumer
```java
public class TestConsumer {
    public static void main(String[] args) {
        method("hello world", (x) -> {
            System.out.println(x.split(" ", 1)[0]);
        });
    }
    
    public static void method(String name, Consumer<String> consumer) {
        consumer.accept(name);
    }
}
```
andThen() 方法
```java
public class TestConsumer {
    public static void main(String[] args) {
        method("hello world", (x) -> {
            System.out.println(x.split(" ", 1)[0]);
        }, (y) -> {
            System.out.println(y.split(" ", 2)[1]);
        });
    }
    
    public static void method(String name, Consumer<String> consumer1, Consumer<String> consumer2) {
        // consumer1 连接 consumer2, 先执行 consumer1 消费数据, 在执行 consumer1 消费数据
        consumer1.andThen(consumer2).accept(name);
    }
}
```

## 2.6 Supplier
使用 Supplier 接口作为方法参数类型，通过 Lambda 表达式求出 int 数组中的最大值。
```java
public class TestSupplier {
    public static void main(String[] args) {
        int[] arr = {10,20,1,999,90,-2,200};
        int maxValue = getMax(() -> {
            int max = arr[0];
            for (int i = 0; i < arr.length; i++) {
                if (max < arr[i]) {
                    max = arr[i];
                }
            }
            return max;
        });
        System.out.println(maxValue);
    }
    
    public static Integer getMax(Supplier supplier) {
        return supplier.get();
    }
}
```

## 2.7 Function


## 2.8 Predicate
做判断，返回有个 boolean 结果
```java
public class TestPredicate {
    public static void main(String[] args) {
        
    }
}
```




# 三、引用



# 四、StreamAPI



# 五、Optional



# 六、接口



# 七、Date / TimeAPI























