---
title: 从Java到Kotlin——基础语法
date: 2018-02-27 15:28:38
tags:
- Kotlin
---

# Print输出

``` Kotlin
print("Hello, World!")
println("Hello, World!")
```

> Java
> 
> ``` Java 
System.out.print("Hello, World!");
System.out.println("Hello, World!");
```

# 常量

``` Kotlin
val x: Int
val y = 1
```

> Java
>
> ``` Java
final int x;
final int y = 1;
```

# 变量

``` Kotlin
var w: Int
var z = 2
z = 3
w = 1 
```

> Java
>
> ``` Java
int w;
int z = 2;
z = 3;
w = 1;
```

# 可空变量

``` Kotlin
val name: String? = null

var lastName: String?
lastName = null

var firstName: String
firstName = null // Compilation error!!
```

> Java
>
> ``` Java
final String name = null;
String lastName;
lastName = null
```


# 空值检查

``` Kotlin
val length = text?.length

val length = text!!.length // NullPointerException if text == null
```

> Java
>
> ``` Java
if(text != null){
  int length = text.length();
}
```

# String

``` Kotlin
val name = "John"
val lastName = "Smith"
val text = "My name is: $name $lastName"
val otherText = "My name is: ${name.substring(2)}"
```

> Java
>
> ``` Java
String name = "John";
String lastName = "Smith";
String text = "My name is: " + name + " " + lastName;
String otherText = "My name is: " + name.substring(2);
```

# 三元运算符

``` Kotlin
val text = if (x > 5)
              "x > 5"
            else "x <= 5"
```

> Java
>
> ``` Java
String text = x > 5 ? "x > 5" : "x <= 5";
```

