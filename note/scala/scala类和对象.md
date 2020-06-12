

1. scala 也支持构造函数的重载
2. 其它的构造函数应该使用 `def this(参数列表){}` 定义
3. scala 构造函数分为两种:

	① 主构造函数
	1. 只能有一个
	 2. 它的入参, 会自动的成为类的属性
	<br/>

	② 辅构造函数
 	1. 可以有多个(构造器的重载)
	2. 首航必须是调用主构造函数
 	3. 它的形参仅仅是一个普通的局部常量
 	4. 辅构造函数调用其他辅构造函数时, 只能后声明的调用先声明的


	```java
	package com.kino.scala.day03.work.obj
	
	import scala.beans.BeanProperty
	
	/**
	  * @author kino
	  * @date 2019/09/06 18:38:05
	  * @version 1.0.0
	  */
	object ObjDemo1 {
	    /**
	      * 1. scala 也支持构造函数的重载
	      * 2. 其它的构造函数应该使用 `def this(参数列表){}` 定义
	      * 3. scala 构造函数分为两种:
	      *     ① 主构造函数
	      *         1. 只能有一个
	      *         2. 它的入参, 会自动的成为类的属性
	      *
	      *     ② 辅构造函数
	      *         1. 可以有多个(构造器的重载)
	      *         2. 首航必须是调用主构造函数
	      *         3. 它的形参仅仅是一个普通的局部常量
	      *         4. 辅构造函数调用其他辅构造函数时, 只能后声明的调用先声明的
	      */
	    def main(args: Array[String]): Unit = {
	        val user = new User("kino", 18, "男")
	        println(user.toString)
	    }
	
	}
	
	//定义一个 User 类, 并指定主构造器为 User(String, Int, String)
	class User(@BeanProperty val name: String, @BeanProperty val age: Int, sex: String) {
	
	    //测试属性 是 User 类的成员变量, 还是 User(String, Int, String) 构造器的局部变量
	    //测试发现, 此处声明的两个变量均为 User 类的成员变量
	    @BeanProperty val phone: Int = 0
	    @BeanProperty val email: String = "email"
	
	    println("User 主构造器被调用.....")
	
	    //定义两个参数的 User 重载的构造器
	    //辅构造器的入参, 不能对变量声明 val/var
	    def this(name: String, age: Int) {
	        //重载的辅构造器必须调用主构造器
	        this(name, age, null)
	
	        //测试属性 是 User 类的成员变量, 还是 User(String, Int) 构造器的局部变量
	        //测试发现, 就算加了 @BeanProperty 注解, 该属性仅是 User(String, Int) 构造器的局部`常量`
	        @BeanProperty val address: String = "两参构造器的参数: address"
	
	        //** 辅构造器之间的调用 -> 不能调用声明在自己之后的辅构造器
	        //** idea 编译不会报错(scala插件有bug), 实际上编译是不通过的
	        //this(name)
	    }
	
	    //定义一个参数的 User 重载的构造器
	    //辅构造器的入参, 不能对变量声明 val/var
	    def this(name: String){
	        //重载的辅构造器必须调用主构造器
	//        this(name, 0, null)
	
	        this(name, 10)
	    }
	
	    override def toString: String = s"name: $name, age: $age, sex: $sex"
	
	}
	```