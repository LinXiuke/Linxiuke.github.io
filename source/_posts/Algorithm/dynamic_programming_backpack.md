---
title: 动态规划-背包问题

date: 2020-02-10

categories: 
- Algorithm

tags:
- 动态规划
---

### 动态规划原理
动态规划与分治法类似，都是把大问题拆分成小问题，通过寻找大问题与小问题的递推关系，解决一个个小问题，最终达到解决原问题的效果。但不同的是，分治法在子问题和子子问题等上被重复计算了很多次，而动态规划则具有记忆性，通过填写表把所有已经解决的子问题答案纪录下来，在新问题里需要用到的子问题可以直接提取，避免了重复计算，从而节约了时间，所以在问题满足最优性原理之后，用动态规划解决问题的核心就在于填表，表填写完毕，最优解也就找到。
<!--more-->
### 01背包问题

#### 问题描述

有n种物品，它们有各自的体积和价值，现有给定容量的背包，如何让背包里装入的物品具有最大的价值总和？

由于一个物品是不可拆分的，只能选择不放入或者放入，所以称为01背包问题。


w（体积） | v（价值）
---|---
2 | 3
3 | 4
4 | 5
5 | 6

#### 解决方法

先定义一些变量：w(i)代表第i个物品的体积，v(i)代表第i个物品的价值，dp(i,j)表示当前背包容量为j，前i个物品组合对应的最大价值。

- 当第i个物品不放入时，dp(i, j) = dp(i-1, j)
- 当第i个物品放入时， dp(i, j) = dp(i-1, j-w(i))+v(i)，表示背包容量减少了w(i)，价值增加了v(i)

在物品放入的时候需要考虑背包容量j的大小，并且要考虑最大价值，在能放入的前提下是选择放入还是不放入才能取得最大值

- 当j < w(i)时：  dp(i, j) = dp(i-i, j)
- 当j >= w(i)时： dp(i, j) = max(dp(i-1, j), dp(i-1, j-w(i))+v(i) )

#### 表格填充

来看下过程： 先补全条件，给定背包容量为10，填充表格，确定边界


w|v|i/j | 0|1|2|3|4|5|6|7|8|9|10
---|---|---|---|---|---|---|---|---|---|---|---|---|---|
0|0|0 | 0|0|0|0|0|0|0|0|0|0|0
2|3|1 | 0
3|4|2 | 0
4|5|3 | 0
|5|6|4 | 0

然后一行一行填表，第i行表示前i个物品放入时的最大价值，最终结果如下：


w|v|i/j | 0|1|2|3|4|5|6|7|8|9|10
---|---|---|---|---|---|---|---|---|---|---|---|---|---|
0|0|0 | 0|0|0|0|0|0|0|0|0|0|0
2|3|1 | 0|0|3|3|3|3|3|3|3|3|3
3|4|2 | 0|0|3|4|4|7|7|7|7|7|7
4|5|3 | 0|0|3|4|5|7|8|9|9|12|12
|5|6|4 | 0|0|3|4|5|7|8|9|10|12|12

最终得到的最大价值dp(4, 10)=12， 如果给的背包容量是8的话，最终得到的最大容量dp(4, 8)=10


#### 代码


```

// 物品数量
int N = 5;
// 背包体积
int V = 10

for(int i = 1; i <= N; i++) {
    for(int j = 1; j <= V; j++) {
        if(j < w[i])
            dp[i][j] = dp[i-1][j];
        else
            dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j-w[i]]+v[i])
    }
}

return dp[N][V];

```

#### 一维数组实现

##### 原理

从以上代码和表格可以看出，第i行数据的值都是通过第i-1行数据推到出的，如果不需要保存历史计算数据只要求得出结果的话就可以通过一维数据来实现。

```
// 物品数量
int N = 5;
// 背包体积
int V = 10

// 先将dp中元素全初始化为0
for(int i = 1; i <= N; i++) {
    for(int j = V; j >= w[i]; j--) {
        dp[j] = Math.max(dp[j], dp[j-w[i]]+v[i]);
    }
}

return dp[V];

```

注意内循环一定要从后往前逆序遍历，否则前面的值可能已经被修改会导致后面的值计算错误，逆序遍历的话因为数据不需要用到后面的值所以修改了也不受影响。


##### 其它例子

[三角形最小路径之和](https://leetcode-cn.com/problems/triangle/)


```
class Solution {

    public int minimumTotal(List<List<Integer>> triangle) {
    
        int size = triangle.size();
        int[] dp = new int[size];

        List<Integer> last = triangle.get(size - 1);
        for (int i = 0; i < size; i++)
            dp[i] = last.get(i);


        for (int i = size-2; i >= 0; i--) {
            List<Integer> list = triangle.get(i);
            for (int j = 0; j < i+1; j++) {
                dp[j] = list.get(j) + Math.min(dp[j], dp[j+1]);
            }
        }


        return dp[0];
    }
}
```


### 完全背包问题


#### 问题描述

有n种物品，每种物品的数量不限，现有给定容量的背包，如何让背包里装入的物品具有最大的价值总和？


#### 解决方法
和01背包问题不同的是，每种物品可以在背包容量允许的条件下放无数个。

- 当第i个物品不放入时，dp(i, j) = dp(i-1, j)，和01背包相同，取上一行的值
- 当第i个物品放入时， dp(i, j) = dp(i, j-w(i))+v(i)，由于第i个物品可以重入放入，和01背包不同，取得是本行的值


#### 表格填充

w|v|i/j | 0|1|2|3|4|5|6|7|8|9|10
---|---|---|---|---|---|---|---|---|---|---|---|---|---|
0|0|0 | 0|0|0|0|0|0|0|0|0|0|0
2|3|1 | 0|0|3|3|6|6|9|9|12|12|15
3|4|2 | 0|0|3|4|6|7|9|10|12|13|15
...|


#### 代码

```

// 物品数量
int N = 5;
// 背包体积
int V = 10

for(int i = 1; i <= N; i++) {
    for(int j = 1; j <= V; j++) {
    
//    以下注解掉的方法也是正确的
//        for(int k = 0; k*w[i] <= j; k++)
//            dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j-k*w[i]]+k*v[i] )
    
        if(j >= w[i])
            dp[i][j] = Math.max(dp[i-1][j], dp[i][j-w[i]]+v[i]);
        else
            dp[i][j] = dp[i-1][j];
    }
}

return dp[N][V];

```



#### 一维数组实现

```

for(int i = 1; i <= N; i++) {
    for(int j = w[i]; j <= V; j++) {
        dp[j] = Math.max(dp[j], dp[j-w[i]]+v[i]);
    }
}

return dp[V];

```

由于二维数组的dp用到的是dp[i-1][j]和dp[i][j-w[i]]的值，第i件物品可以重复放入，所以用的是正序遍历。


### 多重背包问题

#### 问题描述

有n种物品，每种物品的数量不同，现有给定容量的背包，如何让背包里装入的物品具有最大的价值总和？


w（体积） | v（价值）|n（数量）
---|---|---
2 | 3|3
3 | 4|1
4 | 5|2
5 | 6|4


#### 解决方法

该问题需要考虑第i种物品的第n个是否放入的问题

- 不放入 dp[i][j] = max(dp[i][j], dp[i-1][j])
- 放入   dp[i][j] = max( dp[i-1][j], dp[i-1][j-n*w[i]] + n*v[i] )


```

// 物品数量
int N = 5;
// 背包体积
int V = 10

for(int i = 1; i <= N; i++) {
    for(int j = 1; j <= V; j++) {
        for(int k = 0; k < n[i] && k*w[i] <= j; k++) {
            dp[i][j] = Math.max( dp[i-1][j], dp[i-1][j-k*w[i]] + k*v[i] )
        }
    }
}

return dp[N][V];

```

#### 一维数组实现


```
// 物品数量
int N = 5;
// 背包体积
int V = 10

for(int i = 1; i <= N; i++) {
    for(int j = V; j >= w[i]; j--) {
        for(int k = 0; k < n[i] && k*n[i] <= j; k++) {
            dp[j] = Math.max(dp[j], dp[j-k*w[i]]+k*v[i])
        }
    }
}

return dp[N][V];
```

#### 转换

将多重背包问题转换为01背包问题， 取第i种物品的个数小于n[i]，且不超过 V/w[i]。