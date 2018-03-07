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

# 位运算

``` Kotlin
val andResult  = a and b
val orResult   = a or b
val xorResult  = a xor b
val rightShift = a shr 2
val leftShift  = a shl 2
```

> Java
>
> ``` Java
final int andResult  = a & b;
final int orResult   = a | b;
final int xorResult  = a ^ b;
final int rightShift = a >> 2;
final int leftShift  = a << 2;
```

# is/as/in

``` Kotlin
if (x is Int) { }

val text = other as String

if (x in 0..10) { }
```

> Java
>
> ``` Java
if(x instanceof Integer){ }
final String text = (String) other;
if(x >= 0 && x <= 10 ){}
```


# when

``` Kotlin
val x = // value
val xResult = when (x) {
  0, 11 -> "0 or 11"
  in 1..10 -> "from 1 to 10"
  !in 12..14 -> "not from 12 to 14"
  else -> if (isOdd(x)) { "is odd" } else { "otherwise" }
}

val y = // value
val yResult = when {
  isNegative(y) -> "is Negative"
  isZero(y) -> "is Zero"
  isOdd(y) -> "is odd"
  else -> "otherwise"
}
```

> Java
> 
> ``` Java
final int x = // value;
final String xResult;

switch (x){
  case 0:
  case 11:
    xResult = "0 or 11";
    break;
  case 1:
  case 2:
    //...
  case 10:
    xResult = "from 1 to 10";
    break;
  default:
    if(x < 12 && x > 14) {
      xResult = "not from 12 to 14";
      break;
    }

    if(isOdd(x)) {
      xResult = "is odd";
      break;
    }

    xResult = "otherwise";
}
 
 
 
final int y = // value;
final String yResult;

if(isNegative(y)){
  yResult = "is Negative";
} else if(isZero(y)){
  yResult = "is Zero";
}else if(isOdd(y)){
  yResult = "is Odd";
}else {
  yResult = "otherwise";
}
```


# for

``` Kotlin
for (i in 1 until 11) { }

for (i in 1..10 step 2) {}

for (item in collection) {}
for ((index, item) in collection.withIndex()) {}

for ((key, value) in map) {}
```

> Java
> 
> ``` Java
for (int i = 1; i < 11 ; i++) { }

for (int i = 1; i < 11 ; i+=2) { }

for (String item : collection) { }
 
for (Map.Entry<String, String> entry: map.entrySet()) { }
```

# 集合

``` Kotlin
val numbers = listOf(1, 2, 3)

val map = mapOf(1 to "One",
                2 to "Two",
                3 to "Three")
```

> Java
> 
> ``` Java
final List<Integer> numbers = Arrays.asList(1, 2, 3);

final Map<Integer, String> map = new HashMap<Integer, String>();
map.put(1, "One");
map.put(2, "Two");
map.put(3, "Three");


// Java 9
final List<Integer> numbers = List.of(1, 2, 3);

final Map<Integer, String> map = Map.of(1, "One",
                                        2, "Two",
                                        3, "Three");
```

## forEach

``` Kotlin
numbers.forEach {
    println(it)
}
```

> Java
> 
> ``` Java
for (int number : numbers) {
  System.out.println(number);
}
```

## filter

``` Kotlin
numbers.filter  { it > 5 }
       .forEach { println(it) }
```

> Java
> 
> ``` Java
for (int number : numbers) {
  if(number > 5) {
    System.out.println(number);
  }
}
```

## groupBy

``` Kotlin
val groups = numbers.groupBy {
                if (it and 1 == 0) "even" else "odd"
             }
```

> Java
> 
> ``` Java
final Map<String, List<Integer>> groups = new HashMap<>();
for (int number : numbers) {
  if((number & 1) == 0){
    if(!groups.containsKey("even")){
      groups.put("even", new ArrayList<>());
    }

    groups.get("even").add(number);
    continue;
  }

  if(!groups.containsKey("odd")){
    groups.put("odd", new ArrayList<>());
  }

  groups.get("odd").add(number);
}

// or

Map<String, List<Integer>> groups = items.stream().collect(
  Collectors.groupingBy(item -> (item & 1) == 0 ? "even" : "odd")
);
```

## partition

``` Kotlin
val (evens, odds) = numbers.partition { it and 1 == 0 }
```

> Java
> 
> ``` Java
final List<Integer> evens = new ArrayList<>();
final List<Integer> odds = new ArrayList<>();
for (int number : numbers){
  if ((number & 1) == 0) {
    evens.add(number);
  }else {
    odds.add(number);
  }
}
```


## sortedBy

``` Kotlin
val users = getUsers()
users.sortedBy { it.lastname }
```

> Java
> 
> ``` Java
final List<User> users = getUsers();

Collections.sort(users, new Comparator<User>(){
  public int compare(User user, User otherUser){
    return user.lastname.compareTo(otherUser.lastname);
  }
});

// or

users.sort(Comparator.comparing(user -> user.lastname));
```

