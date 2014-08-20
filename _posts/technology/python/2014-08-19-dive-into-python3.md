---
layout: post
title:  "Dive into python3 读书小笔记"
date:   2014-08-19 20:38:26
categories: python
---

1.  Python3 里的字符串 (str 类) 不再是 ASCII 码, 而是 Unicode. 而无意义的二进制序列为 bytes 和 bytearray. 

2.  list, tuple 都是有序的. set, dict 是无序的.

3.  Python 中的函数调用, 第一个 `key=value` 之后的参数必须全是 `key=value` 形式. 

4.  `type(x)` 返回 x 的类型.
    `type(x) == int` 等价于 `type(x) is int` 等价于 `isinstance(x, int)`.
    int 是内置类型, 已预先定义. 

    对于自定义类: 
    `a = myClass()` 和 `type(a) == myClass` 等价于 `type(a) is myClass` 等价于 `isinstance(a, myClass)`.

5.  Python3 普通除法 "/" 为保留精度除, 而 "//" 才为C语言里的整数除 

6.  Python 中的 None 有自己的类型 (NoneType) 而这个类型只能创建着一个实例, 就是 None, 所以所有的 None 都是一样的.

7.  bytes 是字节串, 而 str 是 Unicode 字符串. 两者**不能**互操作! bytes 可以通过 decode 方法来转化成 str, str 可以通过 encode 来转换成 bytes 两者的参数都是编码方式.

8.  bytes 和 bytearray 的区别就是 bytes 是不可变的, 只能读. 
    而 bytearray 是可变的, 可以 `bytearray[1] = value`, 其中 value 的值只能介于 0 ~ 255, 这个很好理解吗, 应为一个字节的值就是介于 0 ~ 255. 其他的都一样.

9.  `len(bytes)` 求 bytes 所占的字节数, 所以 `len("string".encode('utf-8'))` 即可求出 utf-8 编码的 string 的字节数.

10. 松散正则表达式: 
    - 空白符被忽略.
    - 空格, 制表符和回车在正则表达式中并不会匹配空格, 制表符, 回车. 
      如果你想在正则表达式中匹配他们, 可以在前面加一个"\"来转义.
    - 注释信息被忽略. 松散正字表达式中的注释和 Python 代码中的一样,
      都是以"#"开头直到行尾. 它可以在多行正则表达式中增加注释信息,
      这就避免了在 Python 代码中的多行注释. 他们的工作方式是一样的.

11. 生成器 yield 的原理.
    比如说:

    ```python
    def a_yield():                  
        i = 0  
        while i &lt; 5:
            yield i   
            i += 1   
    ```

    调用该函数的时候 `a_yield()` 返回的将是一个生成器, 可用在 for 循环中.

    原理是对其调用 `iter()` 返回其自身, 对其调用 `next()` 则开始执行函数.
    函数在遇到 yield 时相当于 return 会返回 yield 之后的值.
    这个值就是 `next()` 返回的值, 并且函数会暂停. 
    下次调用 `next()` 的时候, 函数会从上次暂停的地方继续执行. 
    当函数执行完毕或者 return 时 (注意, return 不能有返回值), `next()` 会抛出 StopIteration.

12. 迭代器的原理.
    `__iter__(self)` 用于循环初始化, 注意的是, 它返回一个初始化之后的有 `__next__()` 方法的对象, 一般也就是自身 (self) 吗.
    `__next__(self)` 用于 next 调用. 其返回值相当于 `next()` 的返回值.

13. for 循环的原理.
    比如说:

    ```python
    for i in a_list: 
        loop-statement
    ```

    首先调用 `iter(a_list)` 然后对返回的对象 iter_return (而不是 a_list) 不断的调用 `next(iter_return)` 并将值付给 i 直至 StopIteration!

    `iter()` 相当于循环初始化.

14. assert 表达式[, "提示字符串"]
    判断表达式不成立时抛出 AssertionError 并打印"提示字符串".
    等价于:

    ```python
    if not 表达式: 
        raise AssertionError("提示字符串")
    ```

15. 有列表解析, 字典解析, 集合解析. 但是没有元组解析. 因为元组相应的操作会生成一个"生成器 (就是用 yield 创建的那个东西)".
    这个东西可以做为一些以可迭代数据为参数的函数的参数. 比如说: `list()`, `set()`. 在在这种情况下就不需要两边的括号了, 同时比列表解析更有效率.
    也即用: `list(i for i in a_list)` 代替 `list([i for i in a_list])`.

16. `io.StringIO()` 可以将一个字符串转换成流, 类似于文件, 可以用文件的接口来操作字符串 (seek, read, tell 等).
    `io.BytesIO()` 同样可以将一个 byte array 转换成字节文件.

17. with 可以创建一个上下文环境, 当进入该环境时, 会默认执行 \_\_enter\_\_ 方法, 离开时会执行 \_\_exit\_\_ 方法.

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

