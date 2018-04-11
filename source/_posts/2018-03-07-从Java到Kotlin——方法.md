---
title: 从Java到Kotlin——方法
date: 2018-03-07 15:22:31
tags:
- Kotlin
---

# 无参数、无返回值

``` Kotlin
fun hello() {
    println("Hello, World!")
}
```

> Java
> 
> ``` Java
public void hello() {
  System.out.print("Hello, World!");
}
```

# 带参数、无返回值

``` Kotlin
fun hello(name: String) {
    println("Hello, $name!")
}
```

> Java
> 
> ``` Java
public void hello(String name){
  System.out.print("Hello, " + name + "!");
}
```

## 参数带有默认值

``` Kotlin
fun hello(name: String = "World") {
    println("Hello, $name!")
}
```

> Java
> 
> ``` Java
public void hello(String name) {
  if (name == null) {
    name = "World";
  }

  System.out.print("Hello, " + name + "!");
}
```

# 带返回值

``` Kotlin
fun hasItems() : Boolean {
    return true
}
```

> Java
> 
> ``` Java
public boolean hasItems() {
  return true;
}
```


## 简写

``` Kotlin
fun cube(x: Double) : Double = x * x * x
```

> Java
> 
> ``` Java
public double cube(double x) {
  return x * x * x;
}
```

# 传入数组

``` Kotlin
fun sum(vararg x: Int) { }
```

> Java
> 
> ``` Java
public int sum(int... numbers) { }
```

# 主函数/Main方法

``` Kotlin
fun main(args: Array<String>) { }
```

> Java
> 
> ``` Java
public class MyClass {
    public static void main(String[] args){ }
}
```

# 多个参数

``` Kotlin
fun main(args: Array<String>) {
  openFile("file.txt", readOnly = true)
}

fun openFile(filename: String, readOnly: Boolean) : File { }
```

> Java
> 
> ``` Java
public static void main(String[]args){
  openFile("file.txt", true);
}

public static File openFile(String filename, boolean readOnly) { }
```

# 可选参数

``` Kotlin
fun main(args: Array<String>) {
  createFile("file.txt")

  createFile("file.txt", true)
  createFile("file.txt", appendDate = true)

  createFile("file.txt", true, false)
  createFile("file.txt", appendDate = true, executable = true)

  createFile("file.txt", executable = true)
}

fun createFile(filename: String, appendDate: Boolean = false, executable: Boolean = false): File { }
```

> Java
> 
> ``` Java
public static void main(String[]args){
  createFile("file.txt");

  createFile("file.txt", true);

  createFile("file.txt", true, false);

  createExecutableFile("file.txt");
}

public static File createFile(String filename) { }

public static File createFile(String filename, boolean appendDate) { }

public static File createFile(String filename, boolean appendDate, boolean executable) { }

public static File createExecutableFile(String filename) { }
```

# 泛型

``` Kotlin
fun init() {
  val module = createList<String>("net")
  val moduleInferred = createList("net")
}

fun <T> createList(item: T): List<T> { }
```

> Java
> 
> ``` Java
public void init() {
  List<String> moduleInferred = createList("net");
}

public <T> List<T> createList(T item) { }
```








