---
title: "Why nested LazyColumn is not allowed in compose and some solutions you should try"
date: 2024-07-01 00:00:00 +0000
categories: [Compose, Android]
tags: [scrolling, UI, Android, Compose]
---

Implementing nested lists is a common requirement in many applications, which we have traditionally managed using `RecyclerView`. However, in Jetpack Compose, attempting nested `LazyColumn` or `LazyRow` can lead to the following crash.

**Exception:**
java.lang.IllegalStateException: Vertically scrollable component was measured with an infinity maximum height constraints, which is disallowed. One of the common reasons is nesting layouts like LazyColumn and Column(Modifier.verticalScroll()). If you want to add a header before the list of items please add a header as a separate item() before the main items() inside the LazyColumn scope. There are could be other reasons for this to happen: your ComposeView was added into a LinearLayout with some weight, you applied Modifier.wrapContentSize(unbounded = true) or wrote a custom layout. Please try to remove the source of infinite constraints in the hierarchy above the scrolling container.
{: .notice--danger}

Now let's try to understand the issue here. LazyColumns by default have an infinite maximum height because they lazy-load items. Due of this nature, it cannot determine its height and thus uses an infinite as its height. Also, since LazyColumns are scrollable, it needs to keep track of the items visible on the screen. When an item is scrolled out of the screen, a new item gets added. Now since the internal LazyColum's height is infinity, it disrupts the outer LazyColumn from knowing its child's actual height. This is why we are not allowed to nest LazyColumns.

Now that we understand the problem, let's look what solutions are avilable to us.

##### 1. Flatten the list
One way to solve the nesting of LazyColumns is to flatten the list, such that we can get rid of the internal LazyColumn. This way we will have only 1 lazy column which solves the problem.
### Pro
- Solves the LazyColumn issue without adding any new components.

### Con
- Loses data hierarchy. EG: If we are building a feed like Facebook, where we have a post and 2 comments on the post and then the next post. If the user likes a comment, then we can't immediately identify which post the comment belongs to because our list is flattened.

##### 2. Compute the height of the nested LazyColumn
As we found earlier, the infinite height of the LazyColumn is the root cause of the issue. This is understandable as there would be different-sized elements loaded into the LazyColumn. However, this is not the case always. For example, we are building a contacts app or a simple calendar events list. Here the height of the component redered by the list remains consistent. In such cases, we can compute the height of our list in `dp` and set it as the height of our inner LazyColumn. This caused, the height of the list nit be infinite and which solves the problem.

EG: The height of 1 item in the nested list is of `40.dp` and the list has 10 items. It means we can set the height of the LazyColumn to 40 * 10 = `400.dp`

### Pro
- Unlike a flattened list, the hierarchy is still maintained.

### Con
- Computing the expected height can be challenging depending on the data rendered by the composable functions.

##### 3. Setting max height
If you have worked with Veiws before there is an attribute `android:maxWidth` & `android:maxHeight`. The purpose of this is to set the content in the View scale the view but beyond a certain point. Since we are setting a max height, initially the composable function will be loaded with the height required by the content, and is allowed to scale up to the set max height. In Compose, we use the `Modifier.heightIn(max)` to set the max height for a component.

### Pro
- Easy to implement with minimal code changes.

### Con
- Requires hardcoding a maximum height, which may be challenging for dynamic layouts.

Example
```kotlin
@Preview
@Composable
private fun PreviewNestedWithWeight() {
    LazyColumn {
        items(2) { num ->
            if (num == 0) {
                // Using a fixed height for the LazyColumn as I know the height of each item
                // and the number of items in the list. ie: 100.dp * 10 = 1000.dp
                LazyColumn(modifier = Modifier.height(1000.dp)) {
                    items(10) { i ->
                        Text(
                            text = i.toString(),
                            Modifier
                                .size(width = 100.dp, height = 100.dp)
                                .background(Color.Gray)
                        )
                    }
                }
            } else {
                // Using an upper bound for the height of the LazyColumn. This can be some
                // arbitrary value like 10000.dp which we don't expect this Column to reach.
                LazyColumn(modifier = Modifier.heightIn(max = 10000.dp)) {
                    items(10) { i ->
                        Text(
                            text = i.toString(),
                            Modifier
                                .size(100.dp)
                                .background(Color.DarkGray)
                        )
                    }
                }
            }
        }
    }
}
```

#### Conclusion
We explored several approaches to address the crash caused by nested LazyColumns or LazyRows in Compose. Personally, I prefer setting a max height using `Modifier.heightIn(max)` as it is straightforward and requires minimal code changes. However, the best solution depends on your specific requirements. I hope these solutions help you resolve the nested scrolling issue in your Compose application.
