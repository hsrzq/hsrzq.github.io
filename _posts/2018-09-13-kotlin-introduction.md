---
layout:      post
title:       Kotlin 入门
categories:  编程语言
tag:         Android Kotlin
---

## 变量和类型

Kotlin中定义变量使用如下结构定义常量/变量
```
val 常量名: 类型 = 常量值
var 变量名: 类型 = 初始值
```

### 空安全

Java中最常见的一个问题就是NullPointException，在Kotlin中为了减少此问题推出了optional语法。

1. 在定义一个变量时，若在常规类型后加一个`?`，表示此变量可以为null。
    ```kotlin
    /** str1不能为null */
    var str1: String = ""
    /** str2可以为null */
    var str2: String? = null
    ```

2. 在调用可空对象的方法时，需要加一个`?`以进行安全的调用。
    ```kotlin
    /** str1一定不为null，因此直接访问它的属性就是安全的 */
    val len1 = str1.length
    /** str2可能为null，必须加?以安全访问它的属性，当str2为null时，len2也为null */
    val len2 = str2?.length
    ```

3. 如果一个对象定义为可空，但在使用时确信一定不为null，则可以使用`!!`进行强转，**但强烈建议不要这么做**
    ```kotlin
    // 当str2为null时会抛NPE
    val len2 = str2!!.length
    ```

4. 结合`?:`操作符，可以省掉很多null判断（`A ?: B`的含义为，只有当A为null时才会执行B）

    + 例一
    ```kotlin
    fun joinRoom(roomId: Long?, password: String?): Boolean {
        roomId ?: return false
        password ?: return false
        ……
    }
    ```

    + 例二
    ```java
    public String recordingFaceUrl() {
        if (mediaBean != null) {
            if (mediaBean.recording != null) {
                if (mediaBean.recording.face_cover_url != null) {
                    return mediaBean.recording.face_cover_url;
                } else {
                    return mediaBean.recording.cover_image;
                }
            } else {
                return "";
            }
        } else {
            return "";
        }
    }
    ```
    ```kotlin
    fun recordingFaceUrl(): String {
        return mediaBean?.recording?.face_cover_url
                ?: mediaBean?.recording?.cover_image
                ?: ""
    }
    ```

### 类型推导

1. 当定义变量时初始值类型明确时，可以省略类型声明，Kotlin会自动推导出来。
    ```
    // 自动推导出str1的类型为String
    var str1 = "abcde"
    // 会自动推导出len的类型为Int
    var len = "ABDEF".length
    ```

2. 当已经经过了类型检测时，可以直接当作检测后的类型使用，而无需强转。
    ```java
    void forbar(Object obj) {
        if (obj instanceof String) {
            ((String) obj).chatAt(0);
        }
    }
    ```
    ```kotlin
    fun forbar(any: Any) {
        if (any is String) {
            // obj已经被自动转成了String类型
            obj.chatAt(0)
        }
    }
    fun forbar(any: Any) {
        if (any !is String) return
        // obj已经被自动转成了String类型
        obj.chatAt(0)
    }
    ```

3. 在泛型相关调用中，多数泛型参数也能够被自动推导填充（注：目前Java8也支持大部分泛型的类型推导）

4. 与Java进行互调时，自动推导出的类型不是空安全的，并且会失去空安全检测。
    
    ```kotlin
    // 自动推导出str1类型是 String! ，即不是 String 也不是 String?
    val str1 = context.getString(R.string.title)
    // 编译不会报错并可运行，但有NPE风险
    var len1 = str1.length
    
    // 编译不会报错并可运行，但有NPE风险
    val str2: String = context.getString(R.string.title)
    ```

    解决办法：在Java的方法中使用`@Nullable`和`@NotNull`注解明确返回值是否可空

### 特殊类型
1. Kotlin中数字类型处理方式与Java很不同，表面上全部都是封装的类类型，但实际可能根据上下文章自动装箱或拆箱。\
例外情况是可空类型一定对应封装类型，如Kotlin的`Int?`一定对应Java的`Integer`
2. Kotlin中不存在数字的隐式转换，所有不同的类型必须显式调用相应的转换方法。
3. 对于Int和Long类型的的位操作，在Kotlin中必须使用位操作函数或中缀表达式，不能使用操作符。

Java操作 | Kotlin函数形式 | Kotlin中缀形式
:-:|:-:|:-:
`a << b` | `a.shl(b)` | `a shl b`
`a >> b` | `a.shr(b)` | `a shr b`
`a >>> b` | `a.ushr(b)` | `a ushr b`
`a & b` | `a.and(b)` | `a and b`
`a | b` | `a.or(b)` | `a or b`
`~a` | `a.inv()` | 

4. 另外在Kotlin中定义数组的方式也和在Java中不同，尤其基本类型有其自己的数组定义方式。

Java类型 | Kotlin类型
:-:|:-:
`int[]` | `IntArray`
`Integer[]` | `Array&lt;Int&gt;` 或 `Array&lt;Int?&gt;`
`String[]` | `Array&lt;String&gt;` 或 `Array&lt;String?&gt;`

## 方法和属性
### 方法
#### 常规方法

Kotlin中使用如下格式定义方法：
```
fun 方法名(参数列表) = 一句话表达式
fun 方法名(参数列表): 返回类型 {
    方法体
}
```

1. 当返回类型是Unit时（相当于Java中的void），可以省略返回类型

2. 当方法体只有一句话时，可以使用简约格式
    ```kotlin
    fun square(x: Int) = x * x
    ```

#### 包级方法

在Kotlin文件中可以像C++一样直接定义方法，这样的方法在其它Kotlin中看起来是属于包的，不属于任何一个具体的类，在Java中各种Utils类的静态方法都可以使用包级方法实现。例如在IO.kt中有如下方法时，在Java中可以用`IOKt.closeQuietly(is)`调用。
```kotlin
fun closeQuietly(closable: Closable) {}
```

#### 扩展方法

在Kotlin中可以按照如下格式给类增加方法，新增加的方法和类原本的方法调用形式完全相同。
```
fun 类型.方法名(参数列表): 返回类型 {
    方法体
}
```

1. 如果定义的扩展方法是包级方法，则任何使用该类型的地方均可调用。

2. 也可以在某个类型内部定义扩展方法，此时扩展方法只能在外部类型内使用，并且可以访问外部类型的成员变量。
    ```kotlin
    abstract class BaseFragment : Fragment() {
        private val composite = CompositeDisposable()
        // 只有在BaseFragment及其子类中的Disposable才可调用autoDispose方法
        fun Disposable.autoDispose() {
            /*
             * 在扩展方法内部的this指代的是被扩展的对象，此例中指Disposable对象
             * 若要访问外部对象则需要用@明确标识
             */
            this@BaseFragment.composite.add(this)
        }
    }
    ```
    对应于Java中如下代码：
    ```java
    class BaseFragment extends Fragment {
        CompositeDisposable composite = CompositeDisposable();
        public void autoDispose(Disposable disposable) {
            this.composite.add(disposable);
        }
    }
    ```

#### 特殊参数

1. 默认参数

    定义方法时可以给参数设置默认值，在调用时可以省略最右侧的参数
    ```kotlin
    fun foobar(arg1: String, arg2: String = "", arg3: String? = null) {
        // Do something
    }
    
    /* 以下三个调用完全相同 */
    foobar("8023")
    foobar("8023", "")
    foobar("8023", "", null)
    ```
    Kotlin实现此功能时，实际是将被省略的参数自动补全，若要在Java中调用也能省略参数，则需要给方法增加`@JvmOverloads`注解。

2. 命名参数

    在调用方法时，可以指定参数名称，这样就只给指定名称的参数赋值。
    ```kotlin
    fun foobar(arg1: String = "", arg2: String = "", arg3: String? = null) {
        // Do something
    }
    
    /* 以下三个调用完全相同 */
    foobar(arg2 = "8023")
    foobar("", "8023")
    foobar("", "8023", null)
    ```

3. 变长参数

    + 和Java中使用`...`不同，Kotlin中使用varargs标记变长参数。
        ```java
        void foobar(String... args) {}
        ```
        ```kotlin
        fun foobar(vararg args: String) {}
        ```

    + 若需要将变长参数作为其它变长参数的参数，则需要用`*`进行展开。

        ```kotlin
        fun foo(vararg args: Any) {}
        
        fun bar1(vararg args: Any) = foo(args)
        fun bar2(vararg args: Any) = foo(*args)
        
        // 在foo中args[0][0]为“abc”
        bar1("abc")
        // 在foo中args[0]为“abc”
        bar2("abc")
        ```

#### 构造方法

1. Kotlin中任意类的构造方法名称都是`constructor`
2. 与类定义写在一起的构造方法是默认构造方法，其它构造方法都必须调用它。
3. 如果默认构造没有其它特殊要求（如设置显隐性等），`constructor`关键字可以省略
4. 如果默认构造方法的参数有`val`/`var`关键字修饰，则对应的参数自动变成成员常量/成员变量
5. 构造函数参数列表可以有默认值，如果加上`@JvmOverloads`注解则会自动生成多个构造方法以供Java调用
6. `init{}`代码块默认构造函数代码块
```kotlin
/**
 * 由于有JvmOverloads注解，会自动生成RatioLayout(context)和RatioLayout(context, attrs)两个构造方法
 * 由于context有var标记，则context自动变为成员变量
 * 另：Kotlin中定义class默认是final的，只有标为open才能被继承
 */
open class RatioLayout @JvmOverloads constructor(var context: Context, attrs: AttributeSet? = null)
        : FrameLayout(context, attrs, defStyle) {
    /** 非默认构造，必须调用默认构造，即this(context, attrs) */
    protected constructor(context: Context, attrs: AttributeSet?, defStyle: Int): this(context, attrs)
    /** 初始化代码块，相当于默认构造方法的方法体 */
    init {
        /** 在初始化代码块中，可以访问默认构造的所有参数，即使没有标记val/var */
        Log.e("RatioLayout", attrs.length)
    }
}
```

### 属性
在传统Java开发中经常会使用到JavaBean。但JavaBean通常需要手写很多余的getter和setter方法。而在Kotlin中，所有非private变量默认都会自动生成bean方法。
1. 如果不需要自动生成bean方法，可以使用`@JvmField`注解
2. 如果需要在getter或setter中进行额外操作，可以覆盖`get()`和/或`set(value)`方法，其中get的返回类型和set的参数类型必须与属性兼容
3. 属性有字段属性和计算属性两类，带有初始值的为字段属性，字段属性的方法中中可以用field访问原始的字段值
4. 用`val`标识的属性为只读属性，只读属性只有`get()`方法
5. 在kotlin中访问属性时与在java访问public成员变量形式完全相同
```java
public boolean isFavored() {
    return this.recordings != null 
            && this.recordings.recording != null
            && this.recordings.recording.has_favored;
}

public void setFavored(boolean favored) {
    if (recordings != null && recordings.recording != null) {
        recordings.recording.has_favored = favored;
    }
}
```
```kotlin
/** 计算属性 */
var isFavored: Boolean
    get() = mediaBean?.recording?.has_favored ?: false
    set(value) { mediaBean?.recording?.has_favored = value }
/** 字段属性 */
var favoredCount: Int = 0
    private set(value) {
        field = Math.max(0, value)
    }
```

## 伴生对象
Kotlin没有静态变量、静态方法的概念，如果要完成相应的功能可以使用伴生对象。\
如果需要生成纯static变量或方法，可以增加`@JvmStatic`注解。（注意此时变量不能使用`const`关键字）
```kotlin
class SomeFragment: Fragment {
    companion object {
        private const val EXTRA_NAME = "extra_name"
        
        @JvmStatic
        fun newInstance(name: String): SomeFragment {
            val args = Bundle()
            args.put(EXTRA_NAME, name)
            val fragment = SomeFragment()
            fragment.arguments = args
            return fragment
        }
    }
}
```

## Lambda函数
1. 当一个方法（包括构造方法）的参数为有且仅有一个方法的**Java接口**类型时，可以使用更为简约的Lambda形式。
    ```kotlin
    Thread {
        // 异步线程
    }.start()
    ```
    ```java
    new Thread(new Runnable {
        @Override
        public void run() {
            // 异步线程
        }
    }).start();
    ```
2. 当接口有一个参数时，可以不用显示标明，会自动将参数命名为it，另外一般默认会将最后一个语句作为返回值
    ```kotlin
    // 按钮只能点击一次，点击一次后禁用掉
    button.setOnClickListener {
        it.enabled = false // it指onClick(view)的参数view，即button
    }
    ```
3. 当方法的最后一个参数为Java单方法接口时，也可以使用Lambda表达式，此时可以将Lambda写在圆括号之外
4. 当接口有多个参数时，必须明确标示出来，没有用到的参数可以用下划线`_`占位
    ```kotlin
    dialog.setPositiveButton("确定") { it, _ ->
        it.dismiss()
    }
    ```
5. **在Kotlin中定义的接口不能使用Lambda表达式简写**
6. 在Kotlin中可以直接定义lambda函数，并且lambda可以作为其它函数的参数使用。
    ```kotlin
    fun setCallback(callback: (Int, Int) -> Int): Int {
        return callback(1, 2)
    }
    // result的值为3，类型为Int
    val result = setCallback { arg1, arg2 ->
        return arg1 + arg2
    }
    ```
7. 当lambda函数可以为null，定义时需要使用optional语法，调用时需要使用invoke操作符。
    ```kotlin
    fun setCallback(callback: ((Int, Int) -> Int)?): Int {
        return callback?.invoke(1, 2) ?: 0
    }
    // result的值为3
    val result = setCallback { arg1, arg2 ->
        return arg1 + arg2
    }
    // result的值为0
    val result = setCallback(null)
    ```
8. lambda函数在定义时可以为扩展方法，也可以和泛型结合以达到更高级的用法，此时在lambda函数体内this指代的是被扩展的对象。
    ```kotlin
    fun <T, R> with(receiver: T, block: T.() -> R) {
        return receiver.block()
    }
    // result的结果是“这是”
    val result = with("这是测试代码") {
        // 此时this指代的是“这是测试代码”这个字符串对象
        return this.substring(0, 2)
    }
    ```

## 数组和集合
1. 集合分为可修改集合和只读集合，带有`Mutable`的集合为可修改集合，只读集合没有add/set/remove等方法
2. 若数组/集合的初始值已经确定，则可以使用`arrayOf`、`listOf`、`setOf`、`mapOf`等扩展方法
    ```kotlin
    val a = intArrayOf(1, 2, 3, 4)      // a的类型为IntArray
    val b = arrayOf(1, 2, 3, 4)         // b的类型为Array<Int>
    val c = listOf(1, 2, 3, 4)          // c的表面类型为List<Int>，实际承载类型是ArrayList，但不能调用add等方法
    val d = mutableListOf(1, 2, 3, 4)   // d的表面类型为MutableList<Int>，实际承载类型是ArrayList
    val e = mapOf("name" to "Qi",       // e的表面类型为Map<String, Any>，实际承载类型是HashMap
                  "sex" to true)        // key to value 语法表示一个k-v键值对，参见infix表达式
    ```
3. 在Kotlin中集合也可以完全按照数组的样式访问，并且对于Map类型的“下标”可以是非数字形式。（另：参见操作符重载之get/set）
    ```kotlin
    val list = listOf("Android", "iOS", "Web")
    val it1 = list[0]           // it1类型为String，值为Android
    val it2 = list[4]           // it2类型为String，但会抛出IndexOutOfBoundsException
    val it3 = list.getOrNull(4) // it3类型为String?，值为null，不抛异常
    val it4 = list.getOrElse(4, "RD") // it4类型为String，值为RD，不抛异常

    val map = HashMap()
    map["user_id"] = UserManager.INSTANCE.getCurrentUserID()
    map["song_id"] = "1234567890"
    ```
4. Kotlin为数组和集合提供了很多扩展方法，可以很方便的进行过滤、排序、循环、映射等诸多操作。\
每一个filter、map、forEach等都会产生一个循环调用，一般不应该出现多次连续调用的情况。\
（附注：部分文档中说Kotlin在编译时会有优化，无需太过于考虑性能浪费问题，未能验证）
    ```kotlin
    // 打印所有身高超过165的18岁女性用户姓名
    // 糟糕
    users.filter { it.height > 165 }
            .filter { it.age == 18 }
            .filter { it.sex = Sex.FEMALE }
            .map { it.name }
            .forEach { println(it) }
    // 较好
    users.filter { it.height > 165 && it.age == 18 && it.sex = Sex.FEMALE }
            .forEach { println(it.name) }
    ```
5. 数组和集合操作实际就是`for`循环，但在`for`循环中可以通过`continue`跳过本次循环，但在数组和集合操作中则需要使用`return@`语法，因为每一次调用都相当于调用了一次函数。
    ```kotlin
    fun main(args: Array<String>) {
        val list = listOf(1, 2, 3, 4, 5)
        list.map {
            // 若存在此语句，则不会有任何输出，因为map是一个inline函数，实际编译时会被展开
            // 此处的return返回的是main方法，map尚未执行完时main就已经结束了，后续不会执行
            // if (it % 2 == 0) return

            // map函数必须有返回值，并且有多少个传入参数就必须有多少个返回值
            if (it % 2 == 0) return@map 0
            it * it
            // map全部执行完成后的临时集合为[1, 0, 9, 0, 25]
        }.forEach {
            // forEach不要求有返回值，此时就相当于continue功能了
            if (it % 3 == 0) return@forEach
            print(it)
        }
        // 最终输出125
    }
    ```

## 区间
在Java中使用for循环时一般使用`for (int i = A; i < B; i += C)`格式，其中限制条件A、B、C在Kotlin中被封装成XXXRange的区间形式。（参见infix和操作符重载）

Java 代码 | Kotlin 代码
:-:|:-:|-
`for (int i = A; i <= B; i++)` | `for (i in A .. B)`
`for (int i = A; i <= B; i += C)` | `for (i in A .. B step C)`
`for (int i = A; i <= B; i += C)` | `for (i in A rangeTo B step C)`
`for (int i = A; i < B; i += C)` | `for (i in A until B step C)`
`for (int i = A; i >= B; i--)` | `for (i in A downTo B)`
`for (int i = A; i >= B; i -= C)` | `for (i in A downTo B step C)`

## <font color=red>异常</font>
与Java相同，Kotlin中所有异常均是`Throwable`的子类，`try……catch……finally`也与Java基本相同。\
<font color=red style="font-weight:bold;font-size:20pt">但在Java中强制捕获的异常在Kotlin中并不强制！</font>
