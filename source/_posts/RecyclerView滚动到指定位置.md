---
title: RecyclerView滚动到指定位置
date: 2017-02-09 16:27:29
tags:
---

RecyclerView提供了若干个方法来进行滚动：
1.scrollTo(int x, int y)
  >Set the scrolled position of your view.
  
  和scrollBy(int x, int y)
  >Move the scrolled position of your view. 

  这2个方法需要我们去控制移动的距离，自己计算高度或者宽度。在动态的布局中且各项样式高度可能都不一样的情况下，比较有难度。

<!-- more -->

2.scrollToPosition(int position)
  >Convenience method to scroll to a certain position. RecyclerView does not implement scrolling logic, rather forwards the call to RecyclerView.LayoutManager.scrollToPosition(int) 
  
使用scrollToPosition时，移动到当前屏幕可见列表的前面的项时，它会将要显示的项置顶。但是移动到后面的项时，一般会显示在最后的位置。

因此，综合以上两个方法，我们可以先用scrollToPosition方法，将要置顶的项先移动显示出来，然后计算这一项离顶部的距离，用scrollBy完成最后的距离！

首先，按照要置顶的那一项在当前屏幕可见的列表中的相对位置来区分要处理的情况：

``` java
    private void moveToPosition(int index) {
        //获取当前recycleView屏幕可见的第一项和最后一项的Position
        int firstItem = linearLayoutManager.findFirstVisibleItemPosition();
        int lastItem = linearLayoutManager.findLastVisibleItemPosition();
        //然后区分情况
        if (index <= firstItem) {
            //当要置顶的项在当前显示的第一个项的前面时
            rv.scrollToPosition(index);
        } else if (index <= lastItem) {
            //当要置顶的项已经在屏幕上显示时，计算它离屏幕原点的距离
            int top = rv.getChildAt(index - firstItem).getTop();
            rv.scrollBy(0, top);
        } else {
            //当要置顶的项在当前显示的最后一项的后面时
            rv.scrollToPosition(index);
            //记录当前需要在RecyclerView滚动监听里面继续第二次滚动
            move = true;
        }
    }
```

为recycleView添加滚动监听，当完成第一次滚动后，进行第二次的滚动：

``` java
    /**
     * 用户点击的分类在rv的位置
     */
    private int mIndex;
    /**
     * rv是否需要第二次滚动
     */
    private boolean move = false;

    class RecyclerViewListener extends RecyclerView.OnScrollListener {
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            //在这里进行第二次滚动（最后的距离）
            if (move) {
                move = false;
                //获取要置顶的项在当前屏幕的位置，mIndex是记录的要置顶项在RecyclerView中的位置
                int n = mIndex - linearLayoutManager.findFirstVisibleItemPosition();
                if (0 <= n && n < rv.getChildCount()) {
                    //获取要置顶的项顶部离RecyclerView顶部的距离
                    int top = rv.getChildAt(n).getTop();
                    //最后的移动
                    rv.scrollBy(0, top);
                }
            }
        }
    }
```

至此，完成RecyclerView滚动到指定位置的任务。
