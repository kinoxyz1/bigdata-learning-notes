



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




















