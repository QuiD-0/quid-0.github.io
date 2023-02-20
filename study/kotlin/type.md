# Type

&#x20;코틀린은 초기값을 보고 타입을 추론하며 기본 타입들간의 변환은 명시적으로 이루어진다.

```kotlin
val num1 = 1 // num1은 Int형
val num2 = 1L // num2은 Long형
```

## Type 변환

```kotlin
val toInt = num3.toInt()
val toLong = num1.toLong()
val toDouble = num2.toDouble()
```

## is, as

<mark style="color:green;">is</mark> = java의 instanceof와 같이 동작

<mark style="color:green;">as</mark> = 타입 캐스팅

<pre class="language-kotlin"><code class="lang-kotlin">if(num1 is Int){
<strong>    println("num1 is Int")
</strong>    val num4 = num1 as Long
    println("num4 is Long")
}
</code></pre>

<mark style="color:green;">as?</mark> -> null이라면 null반환, 아니라면 뒤의 타입으로 반환

## Any

자바의 Object 타입

모든 primitive 타입의 최상위 타입도 Any

null을 포함할 수 없어서 null을   사용하고 싶다면 Any? 를 사용

```kotlin
fun anyFunction() {
    val any = 1
    if (any is Any) {
        println("any is Any")
    }
}
```

## Unit

java의 void

void와 다르게 Unit은 그 자체로 타입 인자로 사용 가능

함수형 프로그래밍에서 Unit은 단 하나의 인스턴스만 가지는타입을 의미

```kotlin
fun unitFun(): Unit {
    println("unitFun")
}

```



## Nothing

함수가 정상적으로 끝나지 않았다는 사실을 표현

<pre class="language-kotlin"><code class="lang-kotlin">fun nothingFun(): Nothing{
<strong>    throw Exception("Nothing")
</strong>}
</code></pre>





