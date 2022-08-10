



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

自定义接口:@FunctionalInterface
- 需要接口中只有一个抽象方法

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


# 二、函数式接口



# 三、引用



# 四、StreamAPI



# 五、Optional



# 六、接口



# 七、Date / TimeAPI























