---
layout: post
title: 使用 RecylerView 实现触底加载
date: 2015-04-24 10:07
tag: 
    - android
---

最近项目中需要实现列表到底后加载更多的功能，由于我使用的是 RecyclerView，所以我就基于它实现了一个，现在分享给大家。

要实现触底的时候刷新，首先要做的事当然是监听列表滑动到最底部的事件了，在 RecyclerView 当中，实现的方法和以前的 ListView 不同。因为前者有一个非常厉害的 LayoutManager，我们可以利用它来获取一些需要用到的数据。

具体可以移步 [LayoutManager 的 API 文档](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html)

需要用到的是其中的 [getChildCount()](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html#getChildCount())，这个方法用于获取当前可视的 item 的数量；还有是[getItemCount()](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html#getItemCount())，用于获取 RecyclerView 中当前所有 item 的数目。

得到这两个量其实还不够，我们还需要一个量。在 LayoutManager 的子类 [LinearLayoutManager](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html) 中有一个扩展的方法 [findFirstVisibleItemPosition()](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html#findFirstVisibleItemPosition())，这个方法返回当前可视的 item 中第一个 item 的 position，换句话说，其实就是返回了已经滚动过的 item 的数量。

我们很容易可以通过这三个量判断是否已经达到底部，思路是，监听 RecyclerView 的 onScroll 事件，如果当前可视 item 的数目加上已经滚动过的 item 数目大于或者等于所有 item 数目，那么就可以认为列表已经到达底部了。

接下来是 coding time.

首先定义成员变量，这里需要注意的是有一个 onLoading，这个变量用来防止在未加载完成的状态下再一次触发到底事件时重复请求加载。

```java

private RecyclerView list;
private LinearLayoutManager linearLayoutManager;
private boolean onLoading;
private MyAdapter myAdapter;

```

在 onCreate() 里初始化

```java

// onCreate

onLoading = false;

linearLayoutManager = new LinearLayoutManager(this);

myAdapter = new MyAdapter(foo,bar);

list.setLayoutManager(linearLayoutManager);
list.setAdapter(myAdapter);

```

现在就可以监听 list 的 onScroll 事件了：

```java

list.setOnScrollListener(new RecyclerView.OnScrollListener() {
    @Override
    public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
    
        visibleItemCount = layoutManager.getChildCount();
        totalItemCount = layoutManager.getItemCount();
        pastItems = layoutManager.findFirstVisibleItemPosition();
 
        if (!onLoading) {
        
            if ((pastItems + visibleItemCount) >= totalItemCount) {
                Toast.makeText(getApplicationContext(),"loading...",SHORT).show();
                onLoading = true;
                // load something new and set adapter notifyDatasetChanged
                // 记得在 load something 完了以后把 onLoading 赋值为 false
            }
        }
}

```

这样就实现了触底加载，在未来几天我会在 Github 上放一个 Demo app。

快下课了，就写这么多。
