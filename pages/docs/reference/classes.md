---
type: doc
layout: reference
category: "Syntax"
title: "类和继承"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# 类和继承

## 类

Kotlin 中使用关键字 *class*{:.keyword} 声明类

``` kotlin
class Invoice {
}
```

类声明由类名、类头（指定其类型参数、主
构造函数等）和由大括号包围的类体构成。类头和类体都是可选的；
如果一个类没有类体，可以省略花括号。

``` kotlin
class Empty
```


### 构造函数

在 Kotlin 中的一个类可以有一个**主构造函数**和一个或多个**次构造函数**。主
构造函数是类头的一部分：它跟在类名（和可选的类型参数）后。

``` kotlin
class Person constructor(firstName: String) {
}
```

如果主构造函数没有任何注解或者可见性修饰符，可以省略这个 *constructor*{: .keyword }
关键字。

``` kotlin
class Person(firstName: String) {
}
```

主构造函数不能包含任何的代码。初始化的代码可以放
到以 *init*{:.keyword} 关键字作为前缀的**初始化块（initializer blocks）**中：

``` kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

注意，主构造的参数可以在初始化块中使用。它们也可以在
类体内声明的属性初始化器中使用：

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

事实上，声明属性以及从主构造函数初始化属性，Kotlin 有简洁的语法：


``` kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ……
}
```

与普通属性一样，主构造函数中声明的属性可以是
可变的（*var*{: .keyword }）或只读的（*val*{: .keyword }）。

如果构造函数有注解或可见性修饰符，这个 *constructor*{: .keyword } 关键字是必需的，并且
这些修饰符在它前面：

``` kotlin
class Customer public @Inject constructor(name: String) { …… }
```

更多详情，参见[可见性修饰符](visibility-modifiers.html#构造函数)


#### 次构造函数

类也可以声明前缀有 *constructor*{: .keyword }**的次构造函数**：

``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

如果类有一个主构造函数，每个次构造函数需要委托给主构造函数，
可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数
用 *this*{: .keyword } 关键字即可：

``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

如果一个非抽象类没有声明任何（主或次）构造函数，它会有一个生成的
不带参数的主构造函数。构造函数的可见性是 public。如果你不希望你的类
有一个公有构造函数，你需要声明一个带有非默认可见性的空的主构造函数：

``` kotlin
class DontCreateMe private constructor () {
}
```

> **注意**：在 JVM 上，如果主构造函数的所有的参数都有默认值，编译器会生成
> 一个额外的无参构造函数，它将使用默认值。这使得
> Kotlin 更易于使用像 Jackson 或者 JPA 这样的通过无参构造函数创建类的实例的库。
>
> ``` kotlin
> class Customer(val customerName: String = "")
> ```
{:.info}

### 创建类的实例

要创建一个类的实例，我们就像普通函数一样调用构造函数：

``` kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

注意 Kotlin 并没有 *new*{: .keyword } 关键字。

创建嵌套类、内部类和匿名内部类的类实例在[嵌套类](nested-classes.html)中有述。

### 类成员

类可以包含

* 构造函数和初始化块
* [函数](functions.html)
* [属性](properties.html)
* [嵌套类和内部类](nested-classes.html)
* [对象声明](object-declarations.html)


## 继承

在 Kotlin 中所有类都有一个共同的超类 `Any`，这对于没有超类型声明的类是默认超类：

``` kotlin
class Example // 从 Any 隐式继承
```

`Any` 不是 `java.lang.Object`；尤其是，它除了 `equals()`、`hashCode()`和`toString()`外没有任何成员。
更多细节请查阅[Java互操作性](java-interop.html#对象方法)部分。

要声明一个显式的超类型，我们把类型放到类头的冒号之后：

``` kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

如果该类有一个主构造函数，其基类型可以（并且必须）
用（基类型的）主构造函数参数就地初始化。

如果类没有主构造函数，那么每个次构造函数必须
使用 *super*{: .keyword} 关键字初始化其基类型，或委托给另一个构造函数做到这一点。
注意，在这种情况下，不同的次构造函数可以调用基类型的不同的构造函数：

``` kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

类上的 *open*{: .keyword} 标注与 Java 中 *final*{: .keyword} 相反，它允许其他类
从这个类继承。默认情况下，在 Kotlin 中所有的类都是 final，
对应于 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html)书中的
第 17 条：**要么为继承而设计，并提供文档说明，要么就禁止继承**。

### 覆盖方法

我们之前提到过，Kotlin 力求清晰显式。与 Java 不同，Kotlin 需要显式
标注可覆盖的成员（我们称之为*开放*）和覆盖后的成员：

``` kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```

Derived.v() 函数上必须加上 **override**标注。如果没写，编译器将会报错。
如果函数没有标注 **open** 如 `Base.nv()`，则子类中不允许定义相同签名的函数，
不论加不加 **override**。在一个 **final** 类中（没有用 **open** 标注的类），开放成员是禁止的。

标记为 *override*{: .keyword} 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 *final*{: .keyword} 关键字：

``` kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 覆盖属性

属性覆盖与方法覆盖类似；在超类中声明然后在派生类中重新声明的属性必须以 *override*{: .keyword } 开头，并且它们必须具有兼容的类型。每个声明的属性可以由具有初始化器的属性或者具有 getter 方法的属性覆盖。

``` kotlin
open class Foo {
    open val x: Int get { …… }
}

class Bar1 : Foo() {
    override val x: Int = ……
}
```

你也可以用一个 `var` 属性覆盖一个 `val` 属性，但反之则不行。这是允许的，因为一个 `val` 属性本质上声明了一个 getter 方法，而将其覆盖为 `var` 只是在子类中额外声明一个 setter 方法。

请注意，你可以在主构造函数中使用 *override*{: .keyword } 关键字作为属性声明的一部分。

``` kotlin 
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

### 覆盖规则

在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接超类继承相同成员的多个实现，
它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）。
为了表示采用从哪个超类型继承的实现，我们使用由尖括号中超类型名限定的 *super*{: .keyword }，如 `super<Base>`：

``` kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 接口成员默认就是“open”的
    fun b() { print("b") }
}

class C() : A(), B {
    // 编译器要求覆盖 f()：
    override fun f() {
        super<A>.f() // 调用 A.f()
        super<B>.f() // 调用 B.f()
  }
}
```

同时继承 `A` 和 `B` 没问题，并且 `a()` 和 `b()` 也没问题因为 `C` 只继承了每个函数的一个实现。
但是 `f()` 由 `C` 继承了两个实现，所以我们**必须**在 `C` 中覆盖 `f()` 
并且提供我们自己的实现来消除歧义。

## 抽象类

类和其中的某些成员可以声明为 *abstract*{: .keyword}。
抽象成员在本类中可以不用实现。
需要注意的是，我们并不需要用 `open` 标注一个抽象类或者函数——因为这不言而喻。

我们可以用一个抽象成员覆盖一个非抽象的开放成员

``` kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 伴生对象

与 Java 或 C# 不同，在 Kotlin 中类没有静态方法。在大多数情况下，它建议简单地使用
包级函数。

如果你需要写一个可以无需用一个类的实例来调用、但需要访问类内部的
函数（例如，工厂方法），你可以把它写成该类内[对象声明](object-declarations.html)
中的一员。

更具体地讲，如果在你的类内声明了一个[伴生对象](object-declarations.html#伴生对象)，
你就可以使用像在 Java/C# 中调用静态方法相同的语法来调用其成员，只使用类名
作为限定符。
