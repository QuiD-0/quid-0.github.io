# String

## String interpolation

${value} 를 사용

<pre class="language-kotlin"><code class="lang-kotlin">fun stringInterpolation(){
    val name = "Quid"
<strong>    println("Hello, $name!")
</strong>}
</code></pre>



## TrimIndent

```kotlin
fun trimIndent(){
    val text = """
            |Tell me and I forget.
            |Teach me and I remember.
            |Involve me and I learn.
            |(Benjamin Franklin)
            """.trimIndent()
    println(text)
}
```



## String indexing

파이썬과 비슷하게 String\[index]로 값을 가져올 수 있다.

```kotlin
fun stringIndexing() : Unit{
    val str = "abc"
    println(str[0])
    println(str.get(1))
}
```

