# Null safety

## Null 타입

타입 뒤에 '?' 를 붙여주면 해당 변수에 null이 들어갈 수 있다.

```kotlin
val a: String? = null
val b: String = "abc"
```



## Safe Call

val?.method -> val이 null이 아니면 method 실행 null이라면 null 반환

```kotlin
val length :Int? = a?.length
```

체인시 null체크가 간단하다.

```kotlin
bob?.department?.head?.name
```

기존의 자바 코드는 하나하나 null체크를 해주어야 하지만 코틀린에서는 '?' 로 가능



## Elvis Operator <a href="#elvis-operator" id="elvis-operator"></a>

val ?: val2 -> val이 null이라면 val2를 반환 null이 아니라면 val을 반환

```kotlin
 val c = a?.length ?: b?.length ?: -1
```



## !! Operator <a href="#the-operator" id="the-operator"></a>

무조건 실행

```kotlin
a!!.length
```

a가 null이여도 실행시킨다.

