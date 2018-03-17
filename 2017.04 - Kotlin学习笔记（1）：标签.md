---
title: Kotlin学习笔记（1）：标签
date: 2017-05-26
tags: Kotlin
categories: Kotlin
---
> 标签的声明方式：标签名++"@"，如：abc@  
> 标签的引用方式："@"++标签名，如：@abc

Kotlin中标签的含义和Java中的基本一致，都是为了方便跳转到指定位置，常和 *break* 和 *continue* 搭配使用。但是Kotlin中的标签还可以和 *return* 搭配使用，表示在标签处返回。

# Break和Continue

举例如下：

```kotlin
fun main(args: Array<String>) {
    for (i in 1..4) {
        for (j in 1..4) {
            if (i == 2 && j == 2) {
                break
            }
            println("i = $i, j = $j")
        }
    }
}
```

以上代码表示中的 *break* 仅仅是跳出了内层循环，也就是当 i=2,j=2 时终止内层循环，然后 i 加 1 继续循环。打印结果如下：

```
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 1, j = 4
i = 2, j = 1
i = 3, j = 1
i = 3, j = 2
i = 3, j = 3
i = 3, j = 4
i = 4, j = 1
i = 4, j = 2
i = 4, j = 3
i = 4, j = 4
```

如果想让 i=2,j=2 时直接跳出外层循环，就可以使用标签，代码如下：

```kotlin
fun main(args: Array<String>) {
    loop@ for (i in 1..4) {
        for (j in 1..4) {
            if (i == 2 && j == 2) {
                break@loop
            }
            println("i = $i, j = $j")
        }
    }
}
```

在外层循环处声明一个标签<code>loop@</code>，当需要 *break* 的时候，直接使用<code>break@loop</code>就可以跳出外层循环。运行结果如下：

```
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 1, j = 4
i = 2, j = 1
```
上面的代码等价于如下Java代码：

```java
    public static void main(String[] args) {
        loop: for (int i = 1; i <= 4; i++) {
            for (int j = 1; j <= 4; j++) {
                if (i == 2 && j == 2) {
                    break loop;
                }
                System.out.println("i = " + i + ", j = " + j);
            }
        }
    }
```

*continue* 标签的使用方式和 *break* 一样，不再赘述。

# 标签处返回

举例如下：

```kotlin
fun main(args: Array<String>) {
    val ints = intArrayOf(1, 2, 3, 0, 4, 5, 6)
    ints.forEach {
        if (it == 0) return
        print(it)
    }
}
```

上面代码中的 *return* 指的是从 *main* 函数中返回，因为 *main* 函数是最直接包围它的函数。所以运行结果为：

```
123
```

如果想要从 forEach 中的 lambda 表达式中返回，就需要使用标签了。代码如下，在 lambda 表达式的前面声明一个标签<code>lit@</code>，然后在 *return* 处使用标签，即<code>return@lit</code>。

```kotlin
val ints = intArrayOf(1, 2, 3, 0, 4, 5, 6)
ints.forEach lit@ {
	if (it == 0) return@lit
	print(it)
}
```

运行结果为：

```
123456
```

除了这种方式之外，还可以使用隐式标签。 该标签与接受 lambda 表达式的函数同名，在上个例子中就是 forEach 。即：

```kotlin
val ints = intArrayOf(1, 2, 3, 0, 4, 5, 6)
ints.forEach {
	if (it == 0) return@forEach
	print(it)
}
```

如果不使用标签，还可以使用匿名函数替代 lambda 表达式实现上述功能，代码如下。 匿名函数内部的 *return* 语句将从该匿名函数自身返回，但使用这种方式不如使用 lambda 表达式代码清晰简洁。

```kotlin
val ints = intArrayOf(1, 2, 3, 0, 4, 5, 6)
ints.forEach(fun(value: Int) {
	if (value == 0) return
	print(value)
})
```

> 参考资料：[http://kotlinlang.org/docs/reference/returns.html](http://kotlinlang.org/docs/reference/returns.html)