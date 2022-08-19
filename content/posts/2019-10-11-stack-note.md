---
title: 牛客网题目-栈的压入、弹出序列
tags:
  - 算法
  - 栈
categories:
  - 算法
date: 2019-10-11
---

## 2019-10-11-栈的压入、弹出序列

## 题目描述

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

## 分析

1.辅助栈 2.找到弹出的位置

## 代码实现

```java
package nowcoder;

import java.util.Stack;

/**
 * 栈的压入、弹出序列
 */
public class IsPopOrder {
    public boolean isPopOrder(int [] pushA,int [] popA) {
        if (pushA.length==0 || popA.length==0) return false;
        Stack<Integer> stack = new Stack();
        int popIndex = 0;
        for (int i=0; i< pushA.length; i++){
            stack.push(pushA[i]);
            while (!stack.isEmpty() && stack.peek()==popA[popIndex]){
                stack.pop();
                popIndex++;
            }

        }
        return stack.isEmpty();
    }
}
```
