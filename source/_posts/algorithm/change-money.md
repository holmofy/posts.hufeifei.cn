---
title: 换零钱问题
date: 2021-10-27
categories: 算法
mathjax: false
post_src: http://sigenzhe.top/python/2019/11/05/%E6%8D%A2%E9%9B%B6%E9%92%B1%E7%9A%84%E6%96%B9%E6%B3%95.html
tags:
- algorithm
- 递归
- 动态规划
---

## 换零钱问题

在《SICP》第一章 1.22节，刚介绍完递归和迭代，给了一道相当有难度的例题：统计换零钱的方法

> 给定半美元、25美分、10美分、5美分、1美分 5种硬币，将 1 美元换成硬币，有多少种硬币组合？
> 给定任意数量的现金 和 任意组合的硬币种类，计算换零钱所有方式的种数。

## 解题思路

书上给出的解题思想为递归分治： 将总数为 a 的现金换成 n 种硬币组合的数目等于两部分之和：

- 第一部分为现金 `a` 换成除第一种硬币外的所有其他硬币种类的数目
- 第二部分为现金数 `a-d` 换成所有种类硬币的不同方式数目，其中 `d` 为第一种硬币的币值。

怎么来理解呢，这里是将换零钱的方法分为两组，第一组是都没有使用第一种硬币，第二组是都用了第一咱硬币。显然，换成零钱的全部方式就等于完全不用第一种硬币的方式的数目，加上用了第一种硬币的方式的数目。可以看出，分成两组后，这两个问题新问题相对于原问题的范围缩小了，第一组现金没有变，但可选的硬币种类少了一种；第二组硬币种类没有变，但是现金减少了第一种硬币的币值。
这样，就可以将给定金额的换零钱方式问题，通过递归化简为对 更少现金 和 更少种类硬币的同一问题，而求解这样的递归子问题，我们只需确定好问题的边界在哪里，就可以完成运算。边界问题如下：

- 当现金数 `a` 为 0 时，应该算作是有 1 种换零钱的方法
- 当现金数 `a` 小于 0 时，应该算作是有 0 种换零钱的方法
- 当换零钱可选的硬币种类为 0 时，应该算作是有 0 种换零钱的方法

## 分析

光看这些解释还不够直观，我们来用 10 美分来演算一下。

10 美分能够用到的硬币种类只有`[10, 5, 1]`三种。我们用 `10$[10, 5, 1]`这种记法来标记问题。先按币值为10的硬币来分类，将问题化为：

1. 第一组，不使用 10 美分的硬币来表示，只用5美分和1美分来表示 即 `10$[5, 1]`
2. 第二组，使用了 10 美分的硬币，剩下的金额 0 使用全部的硬币种类表示，即 `0$[10, 5, 1]`

根据上面的规则，当现金数为0 时，应该算作是有 1 种换零钱的方法，所以第二组的值为 1。我们只用求解第一组的 `10$[5, 1]`, 然后加上第二组的 1 就是问题 `10$[10, 5, 1]` 的结果。继续化简问题 `10$[5, 1]`，按币值为 5 的硬币来分解:

- 1-1. 第一组，不使用 5 美分的硬币，只用 1 美分的币值来表示 10 美分，即 `10$[1]`
- 1-2. 第二组，使用了一个 5 美分，剩下的 5 美分仍用`[5, 1]` 两种来表示，即 `5$[5, 1]`

其中第一组的结果为 1 ，继续求解第二组问题 `5$[5, 1]`，仍按 5 美分来分类，可以分为：

- 1-2-1. 第一组，不使用 5 美分的硬币，只用 1 美分的币值来表示 5 美分，即 `5$[1]`
- 1-2-2. 第二组，使用一个 5 美分的硬币，剩下的金额 0 使用全部的硬币种类表示 `0$[5, 1]`

综上所述，10 美分金额的硬币换的结果为：

```python
10$[10, 5, 1] = 10$[5, 1] + 0$[10, 5, 1]
              = 10$[1] + 5$[5, 1] + 0$[10, 5, 1]
              = 10$[1] + 5$[1] + 0$[5, 1] + 0$[10, 5, 1]
```

按边界规则，可以直观看出，当金额为 0 或 可选币种只有 1 种时，组合数都为 1 。
所以 `10$[10, 5, 1]` 的最终结果为 4 。这其中的思想就是将一个复杂的问题归约为简单的子问题求解。

## scheme代码

书上 1.22节给出的示例代码如下：

```python
#lang planet neil/sicp
(define (count-change amount)
  (cc amount 5))

(define (cc amount kinds-of-coins)
  (cond ((= amount 0) 1)
        ((or (< amount 0) (= kinds-of-coins 0)) 0)
        (else (+ (cc amount (- kinds-of-coins 1))
                 (cc (- amount
                        (first-denomination kinds-of-coins))
                     kinds-of-coins)))))

(define (first-denomination kinds-of-coins)
  (cond ((= kinds-of-coins 1) 1)
        ((= kinds-of-coins 2) 5)
        ((= kinds-of-coins 3) 10)
        ((= kinds-of-coins 4) 25)
        ((= kinds-of-coins 5) 50)))

(count-change 100)
```

由于第一章刚刚介绍完 Lisp 的判断语句，没有学习数据结构，需要用判断语句来实现硬币种类，所以上面的示例相对麻烦一些。
在学习完第二章的数据结构链表后，可以用表来表示硬币的种类和币值，这样可以轻易更换兑换币种。将上面的程序再重写一下。

```python
#lang planet neil/sicp

(define us-coins (list 50 25 10 5 1))
(define uk-coins (list 100 50 20 10 5 2 1 0.5))

(define (new-cc amount coin-values)
  (cond ((= amount 0) 1)
        (( or (< amount 0) (null? coin-values)) 0)
        (else
         (+ (new-cc amount
                    (cdr coin-values))
            (new-cc (- amount
                       (car coin-values))
                    coin-values)))))

(new-cc 100 us-coins)
(new-cc 100 uk-coins)  
```

上面的两种方案都是用递归来化简问题，其中用 `car`、 `cdr`、 `null?` 来实现对列表的操作。

## Python代码

```python
# -- coding = utf-8 --
def count_change(amount, coins_list):
    if amount == 0:
        return 1
    elif amount < 0 or not coins_list:
        return 0
    else:
        part1 = count_change(amount, coins_list[1:])
        part2 = count_change(amount-coins_list[0], coins_list[:])
    return part1 + part2

us_coins = [1, 5, 10, 25, 50]
print(count_change(100, us_coins)
```

上面代码使用递归方法实现，逻辑非常清楚直观。缺点也比较明显，一是Python 有递归深度限制，当数据金额特别大时可能报错。二是空间复用度高。

凡是递归能解决的问题都可以用迭代解决，具体到这个题目，可以用动态规划的思路来解决。 下面是网上大神的动态规划代码，特别优雅，只有6行。

```python
# -- coding = utf-8 --

def count_change_iterate(count, coins_list):
    result=[1] + [0] * (count) # 设置结果集
    for coin_value in coins_list: # 控制变量，硬币种类从第一种开始，逐一添加
        for amount in range(coin_value, count+1):  
            # result[amount] 存储未添加当前种类硬币币值时 金额 amount 全部兑换的方式
            # result[amount - coin_value] 为包括当前币值时 金额 amount - coin_value可以兑换的方式
            # 两者相加，则为将 result[amount] 更新为 包括当前币值金额 amount可以兑换的方式
            result[amount] += result[amount - coin_value]
    return result[count]

us_coins = [1, 5, 10, 25, 50]
print(count_change_iterate(100, us_coins)
```

## 总结

通过对换零钱问题的分析和求解，让我们更深入的理分治算法的思想，就是将复杂的问题分成两个或更多的相同或相似的子问题，再把子问题分成更小的子问题……直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。