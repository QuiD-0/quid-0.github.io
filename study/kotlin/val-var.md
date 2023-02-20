# val 와 var

## val

<mark style="color:green;">val</mark>ue -> 값을 변경할 수 없다. final 타입

<pre class="language-kotlin"><code class="lang-kotlin"><strong>        val a = 1
</strong>        val b = 2
        b = 3 // Error
        println("a = $a, b = $b")
</code></pre>

## var

<mark style="color:green;">var</mark>iable -> 값을 변경할 수 있다.

```kotlin
        var x = 5
        x = 1
        println("x = $x")
```
