* [一、数组的常用操作](#%E4%B8%80%E6%95%B0%E7%BB%84%E7%9A%84%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)
* [二、List集合的常用操作](#%E4%BA%8Clist%E9%9B%86%E5%90%88%E7%9A%84%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)
* [三、Set集合的常用操作](#%E4%B8%89set%E9%9B%86%E5%90%88%E7%9A%84%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)
* [四、Map的常用操作](#%E5%9B%9Bmap%E7%9A%84%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)
* [五、队列的常用操作](#%E4%BA%94%E9%98%9F%E5%88%97%E7%9A%84%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)


---

# 一、数组的常用操作

**定长数组**
| 操作 |说明  |
|--|--|
| `:+` | 在数组的最右边增加元素 |
|`+:` | 在数组的最左边增加元素
|`++` | 拼接两个数组，并返回一个新的数组
|`++=` | 拼接两个数组
|`mkString(str)` | 将数组转成字符，按照 指定的 “str” 进行拼接
| `length/size` | 获取数组的大小(长度)
| `sum` | 求数组元素的总和(仅对数值型数组有效)
|`head` | 获取第一个元素
|`last `| 获取最后一个元素
|`tail` | 去除第一个元素, 剩下的元素组成的集合
|`take(2)` | 去前 n个元素, 返回时一个新的集合

**可变数组：**
|操作| 说明 |
|--|--|
|`:+` | 在数组的最右边增加元素|
|`+` | 在数组的最左边增加元素
| `+=` | 在数组最右边增加元素
|`-=` | 删除数组元素，仅删除第一次匹配的元素
|`++` | 拼接两个数组，并返回一个新的数组
|`++=` | 拼接两个数组
|`insert(i, n)` | 在指定下标插入元素
|`remove(n)` | 删除指定下标的元素
|`remove(n, c)` | 从指定下标开始删除元素，连续删除 c 个

---

# 二、List集合的常用操作
|操作| 说明 |
|--|--|
| `:+` | 在集合的最右边增加元素
| `+:` | 在集合的最左边增加元素
| `::` | 在集合的最左边增加元素
| `++ `| 拼接两个集合
| `:::` | 拼接两个集合
| `:: Nil` | 拼接多个数字成为 集合

---
# 三、Set集合的常用操作
|操作| 说明 | 示例 |
|--|--| --|
| `set.add(n)` <br/>  `set += n` <br/> `set.+=(n)` | 添加元素| set2.add(100) // 方式一<br/>  set2 += 101 // 方式二 <br/>set2.+=(102) //方式三
| `set.remove(n)` <br/> `set -= n` <br/>  `set.-=(n)` | 删除元素 |set2.remove(100) <br/> set2 -= 101 <br/> set2.-=(102) |
| `for (elem <- set)` |遍历元素 | |
| `set++=list/set`   | 拼接 | set2 ++= List(-1,-2,-3) <br/> set2 ++= mutable.Set(-10,-20,-30)  |
| `set1 ++ set2` <br/> `set1.union(set2)` <br/> `A | B` | 取两个集合的并集 |  | 
| `set1 & set2` <br/> `set1.intersect(set2)` | 取两个集合的交集 |  |
| `set1 &~ set2`  <br/> `set1 -- set2` <br/> `set1.diff(set2)`| 取两个集合的差集 |  | 

---

# 四、Map的常用操作
**取值**
```scala
val mutableMap: mutable.Map[String, Int] = 
		mutable.Map[String, Int]("kino" -> 18, "jerry" -> 20, "Bob" -> 40)
```
        
|操作| 说明  | 示例 | 备注
|--|--| -- | -- |
| `map(K)` |  根据指定 Key取值| `mutableMap("kino")  `|  如果 K 存在, 返回对应 V, 如果 K 不存在, 抛异常
|  `map.get(K).get` |  根据指定 Key取值| `${mutableMap.get("kino").get} `| Map.get(K): 返回一个 Option对象, 如果 K 存在, 返回 Some, 如果 K 不存在, 返回 None <br/> Map.get(K).get: 如果 K 存在, 返回 V, 如果不存在, 抛异常
| `map.getOrElse(K, V)` |  根据指定 Key取值| `mutableMap.getOrElse("kino", 10)  `|  如果 K 存在, 返回 V, 如果不存在, 返回默认值
| `map.contains(K)` |  检查 K 是否存在| `mutableMap.contains("kino")`|  

**增删改查：**
```scala
val map: mutable.Map[String, Int] = mutable.Map(("A", 1), ("B", 2), ("C", 3))
```
|操作| 说明  | 示例 | 备注
|--|--| -- | -- |
| `map(K) = V` | Map 更新 |  map("D") = 4 |  ① 如果被增加的 Key已存在, 则覆盖之前的 V<br> ② Map 是可变的才能修改, 否则报错|
| `map+= (K -> V)` | Map 插入 |  map("D") = 4 |  如果被增加的 Key已存在, 则覆盖之前的 V<br> 
| `map+= ((K,V),(K,V))` | Map 插入多个 |  map += (("F", 5), ("E", 6)) |  如果被增加的 Key已存在, 则覆盖之前的 V<br> 
| `map-= (K -> V)` | Map 删除 |  map -= "E"|  删除元素, K 存在则删除, 不存在忽略<br> 
| `map-= ((K),(K))` | Map 删除多个 |  map -= ("F", "D")|  删除元素, K 存在则删除, 不存在忽略<br> 

**遍历：**
```scala
val map: mutable.Map[String, Int] = mutable.Map(("A", 1), ("B", 2), ("C", 3))

println("----------------map 遍历元素方式一: (k, v) <- map-----------------")
for ((k, v) <- map) {
    println(s"$k : $v")
}
println("----------------map 遍历元素方式二: v <- map.keys-----------------")
for (keys <- map.keys) {
    println(keys)
}
println("----------------map 遍历元素方式三: v <- map.values-----------------")
for (values <- map.values) {
    println(values)
}
println("----------------map 遍历元素方式四: v <- map，返回的是元祖-----------------")
for (v <- map) {
    println(v)
    println(s"$v key=${v._1} val=${v._2}")// 以元祖的形式取值
}
```
运行结果：
```scala
----------------map 遍历元素方式一: (k, v) <- map-----------------
A : 100
C : 3
B : 2
----------------map 遍历元素方式二: v <- map.keys-----------------
A
C
B
----------------map 遍历元素方式三: v <- map.values-----------------
100
3
2
----------------map 遍历元素方式四: v <- map-----------------
(A,100)
(A,100) key=A val=100
(C,3)
(C,3) key=C val=3
(B,2)
(B,2) key=B val=2

Process finished with exit code 0
```

---

# 五、队列的常用操作
```scala
//可变队列的创建
val queue = new mutable.Queue[Int]
```

|操作| 说明 | 示例
|--|--| --|
| `+=`  | 给队列增加元素 | queue += 1 
| `++=` | 给队列追加集合 | queue ++= List(14,5,6)
| `enqueue(n1,n2...)` | 入队 | queue.enqueue(100,1000,10000)
| `dequeue()`  |  出队，从队列的头部去除元素, 队列本身会变 |
| `head` | 返回队列第一个元素 | queue.head
| `last` | 返回队列最后一个元素 | queue.last
| `tail` | 取出队尾的数组(第一个之后的数据) | queue.tail
 