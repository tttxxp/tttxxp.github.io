---
title: "Attention 1: Kotlin interface不能使用Lambda表达式"
date: 2019-10-18 14:22
category:
- Kotlin
---

> 最近尝试使用Kotlin编写Android App，在将Java文件转换成Kotlin时遇到了这个问题：
>   在Kotlin中使用interface时，interface不能转换成lambda表达式  

<!-- more -->

# 使用场景
## Kotlin调用内部interface
``` kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        // 错误❌
        // Error: Type mismatch: interred type is (View) -> Unit but KotlinInterface was expected
        setKotlinInterface { child: View -> useAppContext() }

        // 正确✅
        setKotlinInterface(object: KotlinInterface {
            override fun onClick(child: View) {
                
            }
        })
    }

    fun setKotlinInterface(x: KotlinInterface) = Unit

    interface KotlinInterface {
        fun onClick(child: View)
    }
}
```

## Kotlin调用Java interface
``` kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        // 正确✅
        setJavaInterface(object : JavaInterface {
            override fun onClick(child: View?) {
            }
        })

        // 正确✅，使用lambda表达式
        setJavaInterface(JavaInterface { })
    }

    fun setJavaInterface(x: JavaInterface) = Unit
}
```

* 增加的JavaInterface.java

``` java
// JavaInterface.java
public interface JavaInterface {
    void onClick(View child);
}
```

## Kotlin使用Java类调用Java interface
``` kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        val testA = TestA()
        // 正确✅，使用lambda表达式
        testA.setJavaInterface { useAppContext() }

        // 正确✅，使用lambda表达式
        // 注意这种写法此种写法此时是正确的，而在第一种场景中就是不正确的
        testA.setJavaInterface { child: View? -> print("${child?.id}") }

        // 正确✅
        testA.setJavaInterface(object : JavaInterface {
            override fun onClick(child: View?) {
                useAppContext()
            }
        })
    }

    fun setJavaInterface(x: JavaInterface) = Unit
}
```
* 增加的TestA.java

``` java
// TestA.java
public class TestA {
    public void setJavaInterface(JavaInterface param) {
    }
}
```

# 总结
刚从Java切换到Kotlin，确实不太适应Kotlin内部的一些编码方式，关于下面的问题也还需要之后去深入的思考和探究：
> Kotlin在使用自己的interface时，为什么不能使用lambda表达式？
> 目前的想法：根据错误提示，在Kotlin内部方法也是对象，对应的lambda表达式所表示的方法对象与interface对象不能等同

* 验证猜测，在Kotlin源文件中同时声明如下两个方法并不会报错，也就是说这是两个`重载`的方法

``` kotlin
fun setKotlinInterface(x: KotlinInterface) = Unit
fun setKotlinInterface(x: (child: View) -> Unit) = Unit
```

# 思考🤔
在编码时，大部分情况是在设置监听器的时候使用interface，比如下面这种：
``` java
public class CustomComponent {
    interface Listener {
        void onListen(Object obj);
    }

    private Listener listener;

    public void setListener(Listener listener) {
        this.listener = listener;
    }
}
```
在Kotlin中，更好的方式可能是这样：
``` kotlin
class CustomComponent {
    private lateinit var listener: (any:Any) -> Unit

    fun setListener(listener: (any:Any) -> Unit) {
        this.listener = listener
    }
}
```
而不是这样：
``` kotlin
class CustomComponent {
    private lateinit var listener: Listener

    interface Listener {
        fun onListen(obj: Any)
    }

    fun setListener(listener: Listener) {
        this.listener = listener
    }
}
```





















