---
title: 包含min函数的栈
tags:
  - 牛客网
  - 栈
categories:
  - 栈
  - 算法
date: 2019-10-10
---

## 牛客网题目-包含min函数的栈

## 题目描述

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

## 解法一

双栈实现，数据栈加最小值栈 代码如下：

```java
import java.util.Stack;

public class Solution {


    Stack<Integer> dataStack = new Stack();
    Stack<Integer> minStack = new Stack<>();
    public void push(int node) {
        dataStack.push(node);
        if (minStack.isEmpty() || node<minStack.peek())minStack.push(node);
        else minStack.push(minStack.peek());
    }

    public void pop() {
        dataStack.pop();
        minStack.pop();
    }

    public int top() {
        return dataStack.peek();
    }

    public int min() {
        return minStack.peek();    //辅助栈取值
    }
}
```

## 解法二

栈+ArrayList实现 代码如下：

```java
import java.util.Stack;
import java.util.ArrayList;
import java.util.List;
public class Solution {
    Stack<Integer> dataStack = new Stack();
    List<Integer> minList = new ArrayList<>();
    public void push(int node) {
        dataStack.push(node);
        if (minList.isEmpty() || node < min()) minList.add(node);
        else minList.add( min());
    }
    public void pop() {
        dataStack.pop();
        minList.remove(minList.get(minList.size()-1));
    }
    public int top() {
        return dataStack.peek();
    }
    public int min() {
        return minList.get(minList.size()-1);
    }
}
```
