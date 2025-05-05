```python
def factorial(x):
  if x == 1:
    return 1
  else:
    return x * factorial(x -1)

factorial(3)
```

以上是使用递归计算阶乘的例子；



```python
def sum(x):
  if x == 1:
    return 1
  else:
    return x + sum(x -1)

 sum(10)
```



以上是使用递归计算1到某个正整数所有数字相加值的例子；

《图解算法》中提到要理解递归的概念，首先要理解栈的概念；程序中函数内部还有函数，这种调用会将外层未执行完的代码存储到栈中；栈是先进后出的数据结构；递归会不断调用自己，不断的入栈，直到满足基线条件，然后持续出栈。