# 更好的理解 Kotlin 标准函数 let、apply、also、run 和 with

Kotlin 的 let、apply、also、run、with 这 5 个函数是标准库 kotlin-stdlib-common.jar 包 kotlin.util.Standard.kt 文件内定义的标准函数，通过源代码中函数定义的签名就可以很好的理解透其使用方式。

Standard.kt 源码中的函数定义：

```
public inline fun <T, R> T.let   (block: (T) -> R)               : R
public inline fun <T>    T.apply (block: T.() -> Unit)           : T
public inline fun <T>    T.also  (block: (T) -> Unit)            : T
public inline fun <T, R> T.run   (block: T.() -> R)              : R
public inline fun <R>      run   (block: () -> R)                : R
public inline fun <T, R>   with  (receiver: T, block: T.() -> R) : R
```

> 函数名为 `T.xxx` 的为扩展函数，其余则为普通函数。

总结：

| -              | T.let    | T.apply | T.also | T.run | run     | with
|--------------|---------|-----------|---------|---------|---------|-------
| 对象获取 | it        | this       | it        | this    | -         | this
| 返回值    | result | this       | this     | result | result | result

代码验证：

```
package tech.simter.po

import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Test

// run - 普通函数
val value = run {
  // here has no this
  "newValue"
}

class User(var name: String)
class StandardTest {
  private val clazz = this

  @Test
  fun test() {
    assertEquals("newValue", value)

    val user = User(name = "oldName")
    var result: Any

    // T.let 扩展函数
    result = user.let {
      assertEquals(clazz, this)
      assertEquals(user, it)
      assertEquals("oldName", it.name)
      "newName"
    }
    assertEquals("newName", result)

    // T.apply 扩展函数
    result = user.apply {
      assertEquals(user, this)
      assertEquals("oldName", name) // or this.name
    }
    assertEquals(user, result)

    // T.also 扩展函数
    result = user.also {
      assertEquals(clazz, this)
      assertEquals(user, it)
      assertEquals("oldName", it.name)
    }
    assertEquals(user, result)

    // T.run 扩展函数
    result = user.run {
      assertEquals(user, this)
      assertEquals("oldName", name) // or this.name
      "newName"
    }
    assertEquals("newName", result)

    // T.run 扩展函数 - 相当于 this.run {...}
    result = run {
      assertEquals(clazz, this)
      "newName"
    }
    assertEquals("newName", result)

    // with - 普通函数
    result = with(user) {
      assertEquals(user, this)
      assertEquals("oldName", name) // or this.name
      "newName"
    }
    assertEquals("newName", result)
  }
}
```