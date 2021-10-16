---
title: 动态规划-递归方式 (memeorization)
date: 2021-10-01 15:15:03
tags:
    - algorithm
---



## 定义动态规划

> 动态规划主要是对递归问题的优化。不管什么时候，当我们看到对于同样的输入，我们的解法有很多重复的计算时，我们就可以使用动态规划的技巧进行优化。我们的想法是缓存子问题的结果，当后面需要用到这些结果时，不用再进行重复的计算。这样优化后问题后，时间复杂度可以从`指数级` (exponential) 降到`多项式级` (polynomial)。



## 动态规划的类型

> 这里有两种动态规划的类型，`记忆法` (Memoization) 和`构建表格法` (Tabulation)，第一种是通过缓存已经计算过的结果来减少重复计算，第二种是通过入参的动态表格进行填充。



## 记忆法的示例



### 斐波拉契数列



问题描述：

> Write a function `fib(n)` that takes in a number as an argument.
>
> the function shoud return the n-th number of the Fibonacci sequence



> The 1st and 2nd number of the sequence is 1.
>
> To generate the next number of the sequence, we sum the previous two.



ex: 

> n: 		1,  2,  3,  4,  5,  6,  7,  8,  9, ...
>
> fib(n):  1,  1,  2,  3,  5,  8, 13, 21, 34, ...



分析：

fib(7)

```
                                          		f(7)
                               /                    						\
                          f(6)            													f(5)
                      /       	 \       												/      		 \
                 f(5)        		   f(4) 									f(4)       					f(3)
               / 	    \     	 		/ 	 \ 								/     	\   				/      \
          	f(4)    	f(3) 			f(3)  f(2) 						f(3) 			f(2) 			f(2)    f(1)
           /    \    /  	\    /    \                /
          f(3)  f(2)f(2)  f(1)f(2)  f(1)
         /    \
        f(2)  f(1)
                                  
```

