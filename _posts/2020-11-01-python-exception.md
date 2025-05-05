# Python的异常机制

## 捕获异常

```python
def fetcher(obj, index):
    return obj[index]
  
obj = 'spam'
fetcher(obj, 2)
fetcher(obj, 10)
```

函数fetcher用来获取obj对象索引index的值，以上的例子，当索引为2时返回'a'，当索引为10时会出现IndexError异常，导致程序直接退出；

我们修改代码，捕获异常：

```python
def fetcher(obj, index):
    try:
        return obj[index]
     except IndexError: 
          print('Got index error')
  
obj = 'spam'
fetcher(obj, 2)
fetcher(obj, 10)
```

## 引发异常

## 使用raise语句引发异常；

比如我要引发一个特定的异常，然后进行捕获异常的测试。

```python
try:
    raise TypeError
except TypeError:
    print('Got type error')
    
fetcher('spam', '2') # 这里会引发TypeError的异常
```



## 使用assert断言语句引发异常

```python
assert False, 'Nobody expects the Spanish Inquisition'  # 当assert的值为负的时候，会触发后面的异常
```



## 用户定义的异常

继承Exception异常类，来实现自定义的异常类。


```python
class Bad(Exception):
    pass
    
try:
    raise Bad
exception Bad:
    print('Got Bad Exception')
```

##  Method Resolution Order

Other Exception -> Exception -> BaseException

## 必定会执行的finally语句

```python
try:
    fetch('spam', '2')
 finally:
     print('always will excute')
```

## 异常捕获的意义

> 控制权在异常发生时就会立刻跳到处理器，没必要让所有代码都去预防错误的发生。再者，因为Python会自动检测错误，所以程序代码通常不需要事先检查错误。重点在于，异常让你大致上可以忽略罕见情况，并避免编写检查错误程序代码。

重点在于，异常让你大致上可以忽略罕见情况。

如果不想在Python引发异常时造成程序终止，只要把程序逻辑包装在try中进行捕捉就行了。这是网络服务器这类程序很重要的功能，因它们必须不断持续运行下去。

## 空的except子句

空的except子句是一种通用功能，可以捕获任务东西；不过，空except也会引发一些设计的问题，尽管方便，也可能捕获和程序代码无关、意料之外的系统异常，而且可能意外拦截其他处理器的异常。

Python 3.0 引入了一个替代方案来解决这些问题之一——捕获一个名为Exception的异常，几乎与一个空的except具有相同的效果，但是，忽略和系统退出相关的异常。

```python
try:
    action()
except:
    print('xxx')

try:
    action()
except Exception:
    print('xxx')
```



## try/else 分句

else子句在没有发生异常时执行；else和finally是可选的，可能会有0个或多个except，但是，如果出现一个else的话，必须有至少一个except。

```python
try:
    ...run code ...
except IndexError:
    ...handle exception...
else:
    ... no exception occurred...
```

