---

title: 第155场周赛--LeetCode
tags:
  - 算法
  - LeetCode
  - 数组
categories:
  - 算法
date: 2019-09-22

---

## 第 155 场周赛--LeetCode

## 第一题：最小绝对差

**题目描述：** 给你个整数数组 `arr`，其中每个元素都 **不相同**。 请你找到所有具有最小绝对差的元素对，并且按升序的顺序返回。 **示例 1：**

```text
输入：arr = [4,2,1,3]
输出：[[1,2],[2,3],[3,4]]
```

**示例 2：**

```text
输入：arr = [1,3,6,10,15]
输出：[[1,3]]
```

**示例 3：**

```text
输入：arr = [3,8,-10,23,19,-4,-14,27]
输出：[[-14,-10],[19,23],[23,27]]
```

**提示：**

* `2 <= arr.length <= 10^5`
* `-10^6 <= arr[i] <= 10^6`

**分析** 先排序，遍历算最小差值

**代码实现**

```java
package compete;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * 最小绝对差
  */ public class C5197 {
    public static List<List<Integer>> minimumAbsDifference(int[] arr) {
        if (arr.length<2) return null;
        Arrays.sort(arr);
        for (int i : arr){
            System.out.print(i);
        }
        List<List<Integer>> outList = new ArrayList<>();
        int min = arr[1]-arr[0];
        for (int i=0; i<arr.length-1; i++){
            System.out.println(min);
            if (arr[i+1]-arr[i]<min){
                min = arr[i+1]-arr[i];
                outList.clear();
            }
            if(arr[i+1]-arr[i]==min){
                List<Integer> tmpList = new ArrayList();
                tmpList.add(arr[i]);
                tmpList.add(arr[i+1]);
                outList.add(tmpList);
            }
        }
        return outList;
    }

    public static void main(String[] args) {
        int[] arr = {4,2,1,3};

        List out = minimumAbsDifference(arr);
        System.out.println(out);

    }
}
```

**总结** 算法的时间复杂度为O\(n\)

## 第二题：丑数III

**题目描述** 请你帮忙设计一个程序，用来找出第 `n` 个丑数。 丑数是可以被 `a` **或** `b` **或** `c` 整除的 **正整数**。

**示例 1：**

```text
输入：n = 3, a = 2, b = 3, c = 5
输出：4
解释：丑数序列为 2, 3, 4, 5, 6, 8, 9, 10... 其中第 3 个是 4。
```

**示例 2：**

```text
输入：n = 4, a = 2, b = 3, c = 4
输出：6
解释：丑数序列为 2, 3, 4, 6, 8, 9, 12... 其中第 4 个是 6。
```

**示例 3：**

```text
输入：n = 5, a = 2, b = 11, c = 13
输出：10
解释：丑数序列为 2, 4, 6, 8, 10, 11, 12, 13... 其中第 5 个是 10。
```

**示例 4：**

```text
输入：n = 1000000000, a = 2, b = 217983653, c = 336916467
输出：1999999984
```

**提示：**

* `1 <= n, a, b, c <= 10^9`
* `1 <= a * b * c <= 10^18`
* 本题结果在 `[1, 2 * 10^9]` 的范围内

**分析**

* 方法一暴力法，从1开始遍历
* 方法二，空间换时间，三个计数器，分别存储`a`,`b`,`c`与自然数的乘积，统计到`n`截至

**代码实现** **方法一**：

```java
@OverHideMethods("超时")
public static int nthUglyNumber1(int n, int a, int b, int c) {
    int i = 1;
    while (n>0){
        if (i%a==0 || i%b==0 || i%c==0) n--;
        i++;
    }
    return i-1;
}
```

**方法二**：

```java
public static int nthUglyNumber(int n, int a, int b, int c){
    int i=1, j=1, k=1;
    int tmpA=0,tmpB=0,tmpC=0;
    while (n>0){
        tmpA = a*i;
        tmpB = b*j;
        tmpC = c*k;
        if (tmpA <= Math.min(tmpB,tmpC)) i++;
        if (tmpB <= Math.min(tmpA,tmpC)) j++;
        if (tmpC <= Math.min(tmpA,tmpB)) k++;
        n--;
        System.out.print("tmpA = "+tmpA +" i = "+i + "\t"+"\t"
  + " tmpB = "+tmpB +" j = "+ j +"\t"+"\t"
  + " tmpC = "+tmpC +" k = "+k + "\t");
        System.out.println();
    }
    return Math.min(tmpA,Math.min(tmpB, tmpC));
}
```

**总结** 方法一理论上可以实现，但是缺点是耗时太长； 方法二不知道为啥会缺少了40计数，等参考答案出来我再看看。

> 本周排名：574 / 1602
