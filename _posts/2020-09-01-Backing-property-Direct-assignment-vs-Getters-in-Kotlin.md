---
title: Backing property (Direct assignment vs Getters) in Kotlin
categories:
  - Kotlin
tags:
  - Android
  - Kotlin
---

When working with Android & MVVM, we use `LiveData` to propagate any changes to the View. To do it we use a backing property. What we intend to achieve by doing it is to hide the mutability of `LiveData` from external classes like shown below.

```kotlin
class ViewModel {
  private val _mLiveData = MutableLiveData<Boolean>()
  val mLiveData: LiveData<Boolean> = _mLiveData
}
```
OR

```kotlin
class ViewModel {
  private val _mLiveData = MutableLiveData<Boolean>()
  val mLiveData: LiveData<Boolean>
    get() = _mLiveData
}
```

When we look at it, both methods look similar. However, that's not the case. Let's look at some examples.

##### 1. With Direct assignment
We will start with the first code snippet, but with little bit logging.

```kotlin
class Test() {
  private val a = mutableListOf(5, 10, 15)
  val b : List<Int> = a // Hiding the mutablity from a external class
  
  fun updateList() {
    a.clear()
    a.apply {
      add(2)
      add(4)
      add(6)
    }
  }
}

val t = Test()
print("list = " + t.b.joinToString())
t.updateList()
print("updated list = " + t.b.joinToString())
```

And the output when the above code gets executed is. 
```txt
list = 5, 10, 15
updated list = 2, 4, 6
```

Well, what's wrong? It worked, didn't it?

The answer is Yes, it worked. Now let's make some changes to the above code.

```kotlin
class Test() {
  private var a = mutableListOf(5, 10, 15)
  val b : List<Int> = a  // Hiding the mutablity from a external class
  
  fun updateList() {
    a = mutableListOf(2, 4, 6)
  }
}

val t = Test()
print("list = " + t.b.joinToString())
t.updateList()
print("updated list = " + t.b.joinToString())
```

And the output of the above code is 
```txt
list = 5, 10, 15
updated list = 5, 10, 15
```

Wait, what? How did the output change?
Let's first understand what changed.
1. `a` become mutable (a was `val` but is `var` now)
2. Implementation of `updateList()` changed from updating the current list to creating a new list with new values.

Now let's try to understand why did it print `updated list = 5, 10, 15` instead of `updated list = 2, 4, 6`. For that, let's look under the hood and understand how does this kotlin code look when converted to Java using byte code converter.

```java
// Only has code relevent to us
class Test {
  private List a;
  private final List b;

  public Test() {
    this.b = this.a;
  }

  public void updateList() {
    a = CollectionsKt.mutableListOf(new Integer[]{2, 4, 6});
  }
}
```
As we can see here, `b` is like an indirect reference to the memory pointing to exact same address pointed by `a`. Now if we look at the `updateList()` above, what we did was change the address pointed by `a` by assigning a new value. However, we never update `b` to point to same address. So the `b` is still pointing to the old address where `a` was pointing. Which produced the bug in our code.

##### 2. With a property getter

Let's take the above code, which was buggy and use a getter instead of assignment.

```kotlin
class Test() {
  private var a = mutableListOf(5, 10, 15)
  val b : List<Int>  // Hiding the mutablity from a external class
    get() = a

  fun updateList() {
    a = mutableListOf(2, 4, 6)
  }
}

val t = Test()
print("list = " + t.b.joinToString())
t.updateList()
print("updated list = " + t.b.joinToString())
```

And the output is
```txt
list = 5, 10, 15
updated list = 2, 4, 6
```

Well, it worked as we expected. But why?

To understand why let's look at the kotlin byte code decompiled to Java
```java
class Test {
  private List a;

  @NotNull
  public final List getB() {
    return this.a;
  }
}
```

Well, here `b` is only acting as a getter of `a`. So whenever we call `b` it will always have the latest value of `a`. Isn't this we always wanted?

### Conclusion
We often need to use the backing property. But, it's always prone to bug and it's difficult to spot them. Hence, always use the backing property via a getter.