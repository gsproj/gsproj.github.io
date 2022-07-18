## **一、timeit模块的使用**

timeit模块下主要有两个函数十分有用，分别为timeit.timeit、timeit.repeat

### **1.1 timeit.timeit的使用**

 timeit.timeit参数：

```text
# stmt
指定要执行的语句/statement,值可以是字符串形式的表达式，也可以是一个函数，或者是一个变量的形式。

# number
指定stmt语句执行的次数，默认值为一百万次

# setup
这个参数可以将stmt的环境传进去。比如各种import以及参数。多个值用分号；分隔开

# timer 
指定计数器函数，使用默认time.perf_counter就好，详见下一小节
```

 stmt参数使用示例：

```text
from timeit import timeit, repeat

# 1、stmt值为字符串表达式
res1 = timeit(stmt="[i for i in range(1000)]", number=100)

#2、stmt值为一个变量
statement = """
l = []
for i in range(1000):
    l.append(i)
"""
res2 = timeit(stmt=statement, number=100)


#3、stmt值为一个函数
def foo():
    l = []
    for i in range(1000):
        l.append(i)

res3 = timeit(stmt=foo, number=100)
"""
注意：如果stmt="foo()"，那么必须通过setupo来导入from __main__ import foo如下
res3=timeit(stmt="foo()",setup='from __main__ import foo',number=10)
"""


print(res1)  # 0.003296864000731148
print(res2)  # 0.008153499999025371
print(res3)  # 0.008153499999025371
```

 setup参数使用示例

```text
from timeit import timeit

res=timeit(stmt="json.loads(json_data)",number=1000,
           setup="import json;data={'name':'egon','age':18};json_data=json.dumps(data)"
           )
print(res)
```

\###1.2 timeit.repeat

 其实repeat就比timeit多了一个参数，参数名与函数名一致，也叫repeat，用来指定重复的次数

```text
from timeit import timeit, repeat

res = repeat(stmt="[i for i in range(1000)]", number=100,repeat=3)
print(res)

# repeat=3，所以返回的列表中包含三个元素，所代表的含义依次为：
# 1、第一个100次执行指定语句所耗费时间
# 2、第二个100次执行指定语句所耗费时间
# 3、第三个100次执行指定语句所耗费时间
[0.0033898779984156135, 0.003476719997706823, 0.00342772699877969]
```

\##二、编写通用计时装饰器

 python2和python3里面的计时函数是不一样的，所以推荐建议使用timeit模块中的timeit.default_timer()，它会根据平台不同选取合适的计时函数，详解如下

 由timeit.default_timer()的官方文档可知，计时时间精度和平台以及使用的函数有关：

```text
"Define a default timer, in a platform-specific manner. On Windows, time.clock() has microsecond granularity, but time.time()’s granularity is 1/60th of a second. On Unix, time.clock() has 1/100th of a second granularity, and time.time() is much more precise. On either platform, default_timer() measures wall clock time, not the CPU time. This means that other processes running on the same computer may interfere with the timing."

翻译过来就是：
“定义在默认的计时器中，针对不同平台采用不同方式。在Windows上，time.clock()具有微秒精度，但是time.time()精度是1/60s。在Unix上，time.clock()有1/100s精度，而且time.time()精度远远更高。在另外的平台上，default_timer()测量的是墙上时钟时间，不是CPU时间。这意味着同一计算机的其他进程可能影响计时。”
```

 具体区别可以查看python2和3中timeit的实现

 在python2中：

```text
if sys.platform == "win32":
    # On Windows, the best timer is time.clock()
    default_timer = time.clock
else:
    # On most other platforms the best timer is time.time()
    default_timer = time.time
```

在python3中：default_timer = time.perf_counter

```text
由time.clock()的官方文档可以看出：

"Deprecated since version 3.3: The behaviour of this function depends on the platform: use perf_counter() or process_time() instead, depending on your requirements, to have a well defined behaviour."

翻译过来就是：
“python3.3版本后time.clock()就过时了：这个函数的行为受平台影响，用time.perf_counter()”或者time.process_time()代替来得到一个定义更好的行为，具体取决于你的需求。”
```

 更多详细信息请看官方文档中的time.get_clock_info()

```text
https://docs.python.org/3/library/time.html#time.get_clock_info
```

 综上，我们可以定一个同时适用于Python2和Python3解释器的通用计时装饰器

```text
import timeit

def clock(func):
    def clocked(*args, **kwargs):
        start = timeit.default_timer()
        res = func(*args, **kwargs)
        run_time = timeit.default_timer() - start
        func_name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('调用>>>%s(%s)   返回值>>>%r   耗时>>>%0.8fs' % (func_name, arg_str, res, run_time))
        return res

    return clocked


@clock
def func(n):
    """累计加1"""
    res=0
    for i in range(n):
        res+=1
    return res


if __name__ == '__main__':
    func(10000000)
```