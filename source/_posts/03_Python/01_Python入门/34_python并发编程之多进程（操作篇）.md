---
title: 34-并发编程（三）
date: 2022-07-18 10:12:22
categories:
- Python
- Python入门
tags:
---

### 一 multiprocessing模块介绍

python中的多线程无法利用多核优势，如果想要充分地使用多核CPU的资源（os.cpu_count()查看），在python中大部分情况需要使用多进程。Python提供了multiprocessing。 multiprocessing模块用来开启子进程，并在子进程中执行我们定制的任务（比如函数），该模块与多线程模块threading的编程接口类似。

multiprocessing模块的功能众多：支持子进程、通信和共享数据、执行不同形式的同步，提供了Process、Queue、Pipe、Lock等组件。

需要再次强调的一点是：与线程不同，进程没有任何共享状态，进程修改的数据，改动仅限于该进程内。

### 二 Process类的介绍

**创建进程的类**：

```python
Process([group [, target [, name [, args [, kwargs]]]]])，由该类实例化得到的对象，表示一个子进程中的任务（尚未启动）

强调：
1. 需要使用关键字的方式来指定参数
2. args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号
```

**参数介绍：**

```python
1 group参数未使用，值始终为None
2 
3 target表示调用对象，即子进程要执行的任务
4 
5 args表示调用对象的位置参数元组，args=(1,2,'egon',)
6 
7 kwargs表示调用对象的字典,kwargs={'name':'egon','age':18}
8 
9 name为子进程的名称
```

**方法介绍：**

```python
1 p.start()：启动进程，并调用该子进程中的p.run() 
 2 p.run():进程启动时运行的方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法  
 3 
 4 p.terminate():强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁
 5 p.is_alive():如果p仍然运行，返回True
 6 
 7 p.join([timeout]):主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。timeout是可选的超时时间，需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程
```

**属性介绍：**

```python
1 p.daemon：默认值为False，如果设为True，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，并且设定为True后，p不能创建自己的新进程，必须在p.start()之前设置
2 
3 p.name:进程的名称
4 
5 p.pid：进程的pid
6 
7 p.exitcode:进程在运行时为None、如果为–N，表示被信号N结束(了解即可)
8 
9 p.authkey:进程的身份验证键,默认是由os.urandom()随机生成的32字符的字符串。这个键的用途是为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功（了解即可）
```

### 三 Process类的使用

**注意：在windows中Process()必须放到# if __name__ == '__main__':下**

详细解释

```python
Since Windows has no fork, the multiprocessing module starts a new Python process and imports the calling module. 
If Process() gets called upon import, then this sets off an infinite succession of new processes (or until your machine runs out of resources). 
This is the reason for hiding calls to Process() inside

if __name__ == "__main__"
since statements inside this if-statement will not get called upon import.
由于Windows没有fork，多处理模块启动一个新的Python进程并导入调用模块。 
如果在导入时调用Process（），那么这将启动无限继承的新进程（或直到机器耗尽资源）。 
这是隐藏对Process（）内部调用的原，使用if __name__ == “__main __”，这个if语句中的语句将不会在导入时被调用。
```

**创建并开启子进程的两种方式**

方法一

```python
#开进程的方法一:
import time
import random
from multiprocessing import Process
def piao(name):
    print('%s piaoing' %name)
    time.sleep(random.randrange(1,5))
    print('%s piao end' %name)



p1=Process(target=piao,args=('egon',)) #必须加,号
p2=Process(target=piao,args=('alex',))
p3=Process(target=piao,args=('wupeqi',))
p4=Process(target=piao,args=('yuanhao',))

p1.start()
p2.start()
p3.start()
p4.start()
print('主线程')
```

方法二

```python
#开进程的方法二:
import time
import random
from multiprocessing import Process


class Piao(Process):
    def __init__(self,name):
        super().__init__()
        self.name=name
    def run(self):
        print('%s piaoing' %self.name)

        time.sleep(random.randrange(1,5))
        print('%s piao end' %self.name)

p1=Piao('egon')
p2=Piao('alex')
p3=Piao('wupeiqi')
p4=Piao('yuanhao')

p1.start() #start会自动调用run
p2.start()
p3.start()
p4.start()
print('主线程')
```

**进程直接的内存空间是隔离的**

```python
from multiprocessing import Process
n=100 #在windows系统中应该把全局变量定义在if __name__ == '__main__'之上就可以了
def work():
    global n
    n=0
    print('子进程内: ',n)


if __name__ == '__main__':
    p=Process(target=work)
    p.start()
    print('主进程内: ',n)
```

练习1：把上周所学的socket通信变成并发的形式

server端

```python
from socket import *
from multiprocessing import Process

server=socket(AF_INET,SOCK_STREAM)
server.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
server.bind(('127.0.0.1',8080))
server.listen(5)

def talk(conn,client_addr):
    while True:
        try:
            msg=conn.recv(1024)
            if not msg:break
            conn.send(msg.upper())
        except Exception:
            break

if __name__ == '__main__': #windows下start进程一定要写到这下面
    while True:
        conn,client_addr=server.accept()
        p=Process(target=talk,args=(conn,client_addr))
        p.start()
```

多个client端

```python
from socket import *

client=socket(AF_INET,SOCK_STREAM)
client.connect(('127.0.0.1',8080))


while True:
    msg=input('>>: ').strip()
    if not msg:continue

    client.send(msg.encode('utf-8'))
    msg=client.recv(1024)
    print(msg.decode('utf-8'))
```

这么实现有没有问题？？？

```python
每来一个客户端，都在服务端开启一个进程，如果并发来一个万个客户端，要开启一万个进程吗，你自己尝试着在你自己的机器上开启一万个，10万个进程试一试。
解决方法：进程池
```

***\*Process对象的join方法\****

join：主进程等，等待子进程结束

```python
from multiprocessing import Process
import time
import random

class Piao(Process):
    def __init__(self,name):
        self.name=name
        super().__init__()
    def run(self):
        print('%s is piaoing' %self.name)
        time.sleep(random.randrange(1,3))
        print('%s is piao end' %self.name)


p=Piao('egon')
p.start()
p.join(0.0001) #等待p停止,等0.0001秒就不再等了
print('开始')
```

有了join，程序不就是串行了吗？？？

```python
from multiprocessing import Process
import time
import random
def piao(name):
    print('%s is piaoing' %name)
    time.sleep(random.randint(1,3))
    print('%s is piao end' %name)

p1=Process(target=piao,args=('egon',))
p2=Process(target=piao,args=('alex',))
p3=Process(target=piao,args=('yuanhao',))
p4=Process(target=piao,args=('wupeiqi',))

p1.start()
p2.start()
p3.start()
p4.start()

#有的同学会有疑问:既然join是等待进程结束,那么我像下面这样写,进程不就又变成串行的了吗?
#当然不是了,必须明确：p.join()是让谁等？
#很明显p.join()是让主线程等待p的结束，卡住的是主线程而绝非进程p，

#详细解析如下：
#进程只要start就会在开始运行了,所以p1-p4.start()时,系统中已经有四个并发的进程了
#而我们p1.join()是在等p1结束,没错p1只要不结束主线程就会一直卡在原地,这也是问题的关键
#join是让主线程等,而p1-p4仍然是并发执行的,p1.join的时候,其余p2,p3,p4仍然在运行,等#p1.join结束,可能p2,p3,p4早已经结束了,这样p2.join,p3.join.p4.join直接通过检测，无需等待
# 所以4个join花费的总时间仍然是耗费时间最长的那个进程运行的时间
p1.join()
p2.join()
p3.join()
p4.join()

print('主线程')


#上述启动进程与join进程可以简写为
# p_l=[p1,p2,p3,p4]
# 
# for p in p_l:
#     p.start()
# 
# for p in p_l:
#     p.join()
```

**Process对象的其他方法或属性（了解）**

terminate与is_alive

```python
#进程对象的其他方法一:terminate,is_alive
from multiprocessing import Process
import time
import random

class Piao(Process):
    def __init__(self,name):
        self.name=name
        super().__init__()

    def run(self):
        print('%s is piaoing' %self.name)
        time.sleep(random.randrange(1,5))
        print('%s is piao end' %self.name)


p1=Piao('egon1')
p1.start()

p1.terminate()#关闭进程,不会立即关闭,所以is_alive立刻查看的结果可能还是存活
print(p1.is_alive()) #结果为True

print('开始')
print(p1.is_alive()) #结果为False
```

name与pid

```python
from multiprocessing import Process
import time
import random
class Piao(Process):
    def __init__(self,name):
        # self.name=name
        # super().__init__() #Process的__init__方法会执行self.name=Piao-1,
        #                    #所以加到这里,会覆盖我们的self.name=name

        #为我们开启的进程设置名字的做法
        super().__init__()
        self.name=name

    def run(self):
        print('%s is piaoing' %self.name)
        time.sleep(random.randrange(1,3))
        print('%s is piao end' %self.name)

p=Piao('egon')
p.start()
print('开始')
print(p.pid) #查看pid
```

**![img](https://pic4.zhimg.com/80/v2-1891198a21b860d672d8ac2fae66da97_720w.jpg)**

**僵尸进程与孤儿进程（了解）**

```python
参考博客：http://www.cnblogs.com/Anker/p/3271773.html

一：僵尸进程（有害）
　　僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。详解如下

我们知道在unix/linux中，正常情况下子进程是通过父进程创建的，子进程在创建新的进程。子进程的结束和父进程的运行是一个异步过程,即父进程永远无法预测子进程到底什么时候结束，如果子进程一结束就立刻回收其全部资源，那么在父进程内将无法获取子进程的状态信息。

因此，UNⅨ提供了一种机制可以保证父进程可以在任意时刻获取子进程结束时的状态信息：
1、在每个进程退出的时候，内核释放该进程所有的资源，包括打开的文件，占用的内存等。但是仍然为其保留一定的信息（包括进程号the process ID，退出状态the termination status of the process，运行时间the amount of CPU time taken by the process等）
2、直到父进程通过wait / waitpid来取时才释放. 但这样就导致了问题，如果进程不调用wait / waitpid的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵死进程，将因为没有可用的进程号而导致系统不能产生新的进程. 此即为僵尸进程的危害，应当避免。

　　任何一个子进程(init除外)在exit()之后，并非马上就消失掉，而是留下一个称为僵尸进程(Zombie)的数据结构，等待父进程处理。这是每个子进程在结束时都要经过的阶段。如果子进程在exit()之后，父进程没有来得及处理，这时用ps命令就能看到子进程的状态是“Z”。如果父进程能及时 处理，可能用ps命令就来不及看到子进程的僵尸状态，但这并不等于子进程不经过僵尸状态。  如果父进程在子进程结束之前退出，则子进程将由init接管。init将会以父进程的身份对僵尸状态的子进程进行处理。

二：孤儿进程（无害）

　　孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。

　　孤儿进程是没有父进程的进程，孤儿进程这个重任就落到了init进程身上，init进程就好像是一个民政局，专门负责处理孤儿进程的善后工作。每当出现一个孤儿进程的时候，内核就把孤 儿进程的父进程设置为init，而init进程会循环地wait()它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init进程就会代表党和政府出面处理它的一切善后工作。因此孤儿进程并不会有什么危害。

我们来测试一下（创建完子进程后，主进程所在的这个脚本就退出了，当父进程先于子进程结束时，子进程会被init收养，成为孤儿进程，而非僵尸进程），文件内容

import os
import sys
import time

pid = os.getpid()
ppid = os.getppid()
print 'im father', 'pid', pid, 'ppid', ppid
pid = os.fork()
#执行pid=os.fork()则会生成一个子进程
#返回值pid有两种值：
#    如果返回的pid值为0，表示在子进程当中
#    如果返回的pid值>0，表示在父进程当中
if pid > 0:
    print 'father died..'
    sys.exit(0)

# 保证主线程退出完毕
time.sleep(1)
print 'im child', os.getpid(), os.getppid()

执行文件，输出结果：
im father pid 32515 ppid 32015
father died..
im child 32516 1

看，子进程已经被pid为1的init进程接收了，所以僵尸进程在这种情况下是不存在的，存在只有孤儿进程而已，孤儿进程声明周期结束自然会被init来销毁。


三：僵尸进程危害场景：

　　例如有个进程，它定期的产 生一个子进程，这个子进程需要做的事情很少，做完它该做的事情之后就退出了，因此这个子进程的生命周期很短，但是，父进程只管生成新的子进程，至于子进程 退出之后的事情，则一概不闻不问，这样，系统运行上一段时间之后，系统中就会存在很多的僵死进程，倘若用ps命令查看的话，就会看到很多状态为Z的进程。 严格地来说，僵死进程并不是问题的根源，罪魁祸首是产生出大量僵死进程的那个父进程。因此，当我们寻求如何消灭系统中大量的僵死进程时，答案就是把产生大 量僵死进程的那个元凶枪毙掉（也就是通过kill发送SIGTERM或者SIGKILL信号啦）。枪毙了元凶进程之后，它产生的僵死进程就变成了孤儿进 程，这些孤儿进程会被init进程接管，init进程会wait()这些孤儿进程，释放它们占用的系统进程表中的资源，这样，这些已经僵死的孤儿进程 就能瞑目而去了。

四：测试
#1、产生僵尸进程的程序test.py内容如下

#coding:utf-8
from multiprocessing import Process
import time,os

def run():
    print('子',os.getpid())

if __name__ == '__main__':
    p=Process(target=run)
    p.start()

    print('主',os.getpid())
    time.sleep(1000)


#2、在unix或linux系统上执行
[root@vm172-31-0-19 ~]# python3  test.py &
[1] 18652
[root@vm172-31-0-19 ~]# 主 18652
子 18653

[root@vm172-31-0-19 ~]# ps aux |grep Z
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     18653  0.0  0.0      0     0 pts/0    Z    20:02   0:00 [python3] <defunct> #出现僵尸进程
root     18656  0.0  0.0 112648   952 pts/0    S+   20:02   0:00 grep --color=auto Z

[root@vm172-31-0-19 ~]# top #执行top命令发现1zombie
top - 20:03:42 up 31 min,  3 users,  load average: 0.01, 0.06, 0.12
Tasks:  93 total,   2 running,  90 sleeping,   0 stopped,   1 zombie
%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1016884 total,    97184 free,    70848 used,   848852 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   782540 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                        
root      20   0   29788   1256    988 S  0.3  0.1   0:01.50 elfin                                                                                                                      


#3、
等待父进程正常结束后会调用wait／waitpid去回收僵尸进程
但如果父进程是一个死循环，永远不会结束，那么该僵尸进程就会一直存在，僵尸进程过多，就是有害的
解决方法一：杀死父进程
解决方法二：对开启的子进程应该记得使用join，join会回收僵尸进程
参考python2源码注释
class Process(object):
    def join(self, timeout=None):
        '''
        Wait until child process terminates
        '''
        assert self._parent_pid == os.getpid(), 'can only join a child process'
        assert self._popen is not None, 'can only join a started process'
        res = self._popen.wait(timeout)
        if res is not None:
            _current_process._children.discard(self)

join方法中调用了wait，告诉系统释放僵尸进程。discard为从自己的children中剔除

解决方法三：http://blog.csdn.net/u010571844/article/details/50419798
```

**思考：**

```python
from multiprocessing import Process
import time,os

def task():
    print('%s is running' %os.getpid())
    time.sleep(3)

if __name__ == '__main__':
    p=Process(target=task)
    p.start()
    p.join() # 等待进程p结束后，join函数内部会发送系统调用wait，去告诉操作系统回收掉进程p的id号

    print(p.pid) #？？？此时能否看到子进程p的id号
    print('主')
```

答案

```python
#答案：可以
#分析：
p.join()是像操作系统发送请求，告知操作系统p的id号不需要再占用了，回收就可以，
此时在父进程内还可以看到p.pid,但此时的p.pid是一个无意义的id号，因为操作系统已经将该编号回收

打个比方：
我党相当于操作系统，控制着整个中国的硬件，每个人相当于一个进程，每个人都需要跟我党申请一个身份证号
该号码就相当于进程的pid，人死后应该到我党那里注销身份证号，p.join()就相当于要求我党回收身份证号，但p的家人（相当于主进程）
仍然持有p的身份证，但此刻的身份证已经没有意义
```

### 四 守护进程

主进程创建守护进程

其一：守护进程会在主进程代码执行结束后就终止

其二：守护进程内无法再开启子进程,否则抛出异常：AssertionError: daemonic processes are not allowed to have children

注意：进程之间是互相独立的，主进程代码运行结束，守护进程随即终止

```python
from multiprocessing import Process
import time
import random

class Piao(Process):
    def __init__(self,name):
        self.name=name
        super().__init__()
    def run(self):
        print('%s is piaoing' %self.name)
        time.sleep(random.randrange(1,3))
        print('%s is piao end' %self.name)


p=Piao('egon')
p.daemon=True #一定要在p.start()前设置,设置p为守护进程,禁止p创建子进程,并且父进程代码执行结束,p即终止运行
p.start()
print('主')
```

迷惑人的例子

```python
#主进程代码运行完毕,守护进程就会结束
from multiprocessing import Process
from threading import Thread
import time
def foo():
    print(123)
    time.sleep(1)
    print("end123")

def bar():
    print(456)
    time.sleep(3)
    print("end456")


p1=Process(target=foo)
p2=Process(target=bar)

p1.daemon=True
p1.start()
p2.start()
print("main-------") #打印该行则主进程代码结束,则守护进程p1应该被终止,可能会有p1任务执行的打印信息123,因为主进程打印main----时,p1也执行了,但是随即被终止
```

### 五 进程同步(锁)

进程之间数据不共享,但是共享同一套文件系统,所以访问同一个文件,或同一个打印终端,是没有问题的,

而共享带来的是竞争，竞争带来的结果就是错乱，如何控制，就是加锁处理

part1：多个进程共享同一打印终端

并发运行,效率高,但竞争同一打印终端,带来了打印错乱

```python
#并发运行,效率高,但竞争同一打印终端,带来了打印错乱
from multiprocessing import Process
import os,time
def work():
    print('%s is running' %os.getpid())
    time.sleep(2)
    print('%s is done' %os.getpid())

if __name__ == '__main__':
    for i in range(3):
        p=Process(target=work)
        p.start()
```

加锁：由并发变成了串行,牺牲了运行效率,但避免了竞争

```python
#由并发变成了串行,牺牲了运行效率,但避免了竞争
from multiprocessing import Process,Lock
import os,time
def work(lock):
    lock.acquire()
    print('%s is running' %os.getpid())
    time.sleep(2)
    print('%s is done' %os.getpid())
    lock.release()
if __name__ == '__main__':
    lock=Lock()
    for i in range(3):
        p=Process(target=work,args=(lock,))
        p.start()
```

part2：多个进程共享同一文件

文件当数据库，模拟抢票

并发运行，效率高，但竞争写同一文件，数据写入错乱

```python
#文件db的内容为：{"count":1}
#注意一定要用双引号，不然json无法识别
from multiprocessing import Process,Lock
import time,json,random
def search():
    dic=json.load(open('db.txt'))
    print('\033[43m剩余票数%s\033[0m' %dic['count'])

def get():
    dic=json.load(open('db.txt'))
    time.sleep(0.1) #模拟读数据的网络延迟
    if dic['count'] >0:
        dic['count']-=1
        time.sleep(0.2) #模拟写数据的网络延迟
        json.dump(dic,open('db.txt','w'))
        print('\033[43m购票成功\033[0m')

def task(lock):
    search()
    get()
if __name__ == '__main__':
    lock=Lock()
    for i in range(100): #模拟并发100个客户端抢票
        p=Process(target=task,args=(lock,))
        p.start()
```

加锁：购票行为由并发变成了串行，牺牲了运行效率，但保证了数据安全

```python
#文件db的内容为：{"count":1}
#注意一定要用双引号，不然json无法识别
from multiprocessing import Process,Lock
import time,json,random
def search():
    dic=json.load(open('db.txt'))
    print('\033[43m剩余票数%s\033[0m' %dic['count'])

def get():
    dic=json.load(open('db.txt'))
    time.sleep(0.1) #模拟读数据的网络延迟
    if dic['count'] >0:
        dic['count']-=1
        time.sleep(0.2) #模拟写数据的网络延迟
        json.dump(dic,open('db.txt','w'))
        print('\033[43m购票成功\033[0m')

def task(lock):
    search()
    lock.acquire()
    get()
    lock.release()
if __name__ == '__main__':
    lock=Lock()
    for i in range(100): #模拟并发100个客户端抢票
        p=Process(target=task,args=(lock,))
        p.start()
```

总结：

```python
#加锁可以保证多个进程修改同一块数据时，同一时间只能有一个任务可以进行修改，即串行的修改，没错，速度是慢了，但牺牲了速度却保证了数据安全。
虽然可以用文件共享数据实现进程间通信，但问题是：
1.效率低（共享数据基于文件，而文件是硬盘上的数据）
2.需要自己加锁处理



#因此我们最好找寻一种解决方案能够兼顾：1、效率高（多个进程共享一块内存的数据）2、帮我们处理好锁问题。这就是mutiprocessing模块为我们提供的基于消息的IPC通信机制：队列和管道。
1 队列和管道都是将数据存放于内存中
2 队列又是基于（管道+锁）实现的，可以让我们从复杂的锁问题中解脱出来，
我们应该尽量避免使用共享数据，尽可能使用消息传递和队列，避免处理复杂的同步和锁问题，而且在进程数目增多时，往往可以获得更好的可获展性。
```

### 六 队列（推荐使用）

进程彼此之间互相隔离，要实现进程间通信（IPC），multiprocessing模块支持两种形式：队列和管道，这两种方式都是使用消息传递的

**创建队列的类（底层就是以管道和锁定的方式实现）**：

```text
1 Queue([maxsize]):创建共享的进程队列，Queue是多进程安全的队列，可以使用Queue实现多进程之间的数据传递。
```

**参数介绍：**

```text
1 maxsize是队列中允许最大项数，省略则无大小限制。
```

**方法介绍：**

主要方法：

```python
1 q.put方法用以插入数据到队列中，put方法还有两个可选参数：blocked和timeout。如果blocked为True（默认值），并且timeout为正值，该方法会阻塞timeout指定的时间，直到该队列有剩余的空间。如果超时，会抛出Queue.Full异常。如果blocked为False，但该Queue已满，会立即抛出Queue.Full异常。
2 q.get方法可以从队列读取并且删除一个元素。同样，get方法有两个可选参数：blocked和timeout。如果blocked为True（默认值），并且timeout为正值，那么在等待时间内没有取到任何元素，会抛出Queue.Empty异常。如果blocked为False，有两种情况存在，如果Queue有一个值可用，则立即返回该值，否则，如果队列为空，则立即抛出Queue.Empty异常.
3  
4 q.get_nowait():同q.get(False)
5 q.put_nowait():同q.put(False)
6 
7 q.empty():调用此方法时q为空则返回True，该结果不可靠，比如在返回True的过程中，如果队列中又加入了项目。
8 q.full()：调用此方法时q已满则返回True，该结果不可靠，比如在返回True的过程中，如果队列中的项目被取走。
9 q.qsize():返回队列中目前项目的正确数量，结果也不可靠，理由同q.empty()和q.full()一样
```

其他方法(了解)：

```python
1 q.cancel_join_thread():不会在进程退出时自动连接后台线程。可以防止join_thread()方法阻塞
2 q.close():关闭队列，防止队列中加入更多数据。调用此方法，后台线程将继续写入那些已经入队列但尚未写入的数据，但将在此方法完成时马上关闭。如果q被垃圾收集，将调用此方法。关闭队列不会在队列使用者中产生任何类型的数据结束信号或异常。例如，如果某个使用者正在被阻塞在get()操作上，关闭生产者中的队列不会导致get()方法返回错误。
3 q.join_thread()：连接队列的后台线程。此方法用于在调用q.close()方法之后，等待所有队列项被消耗。默认情况下，此方法由不是q的原始创建者的所有进程调用。调用q.cancel_join_thread方法可以禁止这种行为
```

**应用：**

```python
'''
multiprocessing模块支持进程间通信的两种主要形式:管道和队列
都是基于消息传递实现的,但是队列接口
'''

from multiprocessing import Process,Queue
import time
q=Queue(3)


#put ,get ,put_nowait,get_nowait,full,empty
q.put(3)
q.put(3)
q.put(3)
print(q.full()) #满了

print(q.get())
print(q.get())
print(q.get())
print(q.empty()) #空了
```

**生产者消费者模型**

*在并发编程中使用生产者和消费者模式能够解决绝大多数并发问题。该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度。*

**为什么要使用生产者和消费者模式**

*在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。*

**什么是生产者消费者模式**

*生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。*

*基于队列实现生产者消费者模型*

```python
from multiprocessing import Process,Queue
import time,random,os
def consumer(q):
    while True:
        res=q.get()
        time.sleep(random.randint(1,3))
        print('\033[45m%s 吃 %s\033[0m' %(os.getpid(),res))

def producer(q):
    for i in range(10):
        time.sleep(random.randint(1,3))
        res='包子%s' %i
        q.put(res)
        print('\033[44m%s 生产了 %s\033[0m' %(os.getpid(),res))

if __name__ == '__main__':
    q=Queue()
    #生产者们:即厨师们
    p1=Process(target=producer,args=(q,))

    #消费者们:即吃货们
    c1=Process(target=consumer,args=(q,))

    #开始
    p1.start()
    c1.start()
    print('主')
```

生产者消费者模型总结

```python
#生产者消费者模型总结

    #程序中有两类角色
        一类负责生产数据（生产者）
        一类负责处理数据（消费者）

    #引入生产者消费者模型为了解决的问题是：
        平衡生产者与消费者之间的工作能力，从而提高程序整体处理数据的速度

    #如何实现：
        生产者<-->队列<——>消费者
    #生产者消费者模型实现类程序的解耦和
```

此时的问题是主进程永远不会结束，原因是：生产者p在生产完后就结束了，但是消费者c在取空了q之后，则一直处于死循环中且卡在q.get()这一步。

解决方式无非是让生产者在生产完毕后，往队列中再发一个结束信号，这样消费者在接收到结束信号后就可以break出死循环

生产者在生产完毕后发送结束信号None

```python
from multiprocessing import Process,Queue
import time,random,os
def consumer(q):
    while True:
        res=q.get()
        if res is None:break #收到结束信号则结束
        time.sleep(random.randint(1,3))
        print('\033[45m%s 吃 %s\033[0m' %(os.getpid(),res))

def producer(q):
    for i in range(10):
        time.sleep(random.randint(1,3))
        res='包子%s' %i
        q.put(res)
        print('\033[44m%s 生产了 %s\033[0m' %(os.getpid(),res))
    q.put(None) #发送结束信号
if __name__ == '__main__':
    q=Queue()
    #生产者们:即厨师们
    p1=Process(target=producer,args=(q,))

    #消费者们:即吃货们
    c1=Process(target=consumer,args=(q,))

    #开始
    p1.start()
    c1.start()
    print('主')
```

注意：结束信号None，不一定要由生产者发，主进程里同样可以发，但主进程需要等生产者结束后才应该发送该信号

主进程在生产者生产完毕后发送结束信号None

```python
from multiprocessing import Process,Queue
import time,random,os
def consumer(q):
    while True:
        res=q.get()
        if res is None:break #收到结束信号则结束
        time.sleep(random.randint(1,3))
        print('\033[45m%s 吃 %s\033[0m' %(os.getpid(),res))

def producer(q):
    for i in range(2):
        time.sleep(random.randint(1,3))
        res='包子%s' %i
        q.put(res)
        print('\033[44m%s 生产了 %s\033[0m' %(os.getpid(),res))

if __name__ == '__main__':
    q=Queue()
    #生产者们:即厨师们
    p1=Process(target=producer,args=(q,))

    #消费者们:即吃货们
    c1=Process(target=consumer,args=(q,))

    #开始
    p1.start()
    c1.start()

    p1.join()
    q.put(None) #发送结束信号
    print('主')
```

但上述解决方式，在有多个生产者和多个消费者时，我们则需要用一个很low的方式去解决

有几个消费者就需要发送几次结束信号：相当low

```python
from multiprocessing import Process,Queue
import time,random,os
def consumer(q):
    while True:
        res=q.get()
        if res is None:break #收到结束信号则结束
        time.sleep(random.randint(1,3))
        print('\033[45m%s 吃 %s\033[0m' %(os.getpid(),res))

def producer(name,q):
    for i in range(2):
        time.sleep(random.randint(1,3))
        res='%s%s' %(name,i)
        q.put(res)
        print('\033[44m%s 生产了 %s\033[0m' %(os.getpid(),res))



if __name__ == '__main__':
    q=Queue()
    #生产者们:即厨师们
    p1=Process(target=producer,args=('包子',q))
    p2=Process(target=producer,args=('骨头',q))
    p3=Process(target=producer,args=('泔水',q))

    #消费者们:即吃货们
    c1=Process(target=consumer,args=(q,))
    c2=Process(target=consumer,args=(q,))

    #开始
    p1.start()
    p2.start()
    p3.start()
    c1.start()

    p1.join() #必须保证生产者全部生产完毕,才应该发送结束信号
    p2.join()
    p3.join()
    q.put(None) #有几个消费者就应该发送几次结束信号None
    q.put(None) #发送结束信号
    print('主')
```

其实我们的思路无非是发送结束信号而已，有另外一种队列提供了这种机制

```python
#JoinableQueue([maxsize])：这就像是一个Queue对象，但队列允许项目的使用者通知生成者项目已经被成功处理。通知进程是使用共享的信号和条件变量来实现的。

   #参数介绍：
    maxsize是队列中允许最大项数，省略则无大小限制。    
　 #方法介绍：
    JoinableQueue的实例p除了与Queue对象相同的方法之外还具有：
    q.task_done()：使用者使用此方法发出信号，表示q.get()的返回项目已经被处理。如果调用此方法的次数大于从队列中删除项目的数量，将引发ValueError异常
    q.join():生产者调用此方法进行阻塞，直到队列中所有的项目均被处理。阻塞将持续到队列中的每个项目均调用q.task_done（）方法为止
from multiprocessing import Process,JoinableQueue
import time,random,os
def consumer(q):
    while True:
        res=q.get()
        time.sleep(random.randint(1,3))
        print('\033[45m%s 吃 %s\033[0m' %(os.getpid(),res))

        q.task_done() #向q.join()发送一次信号,证明一个数据已经被取走了

def producer(name,q):
    for i in range(10):
        time.sleep(random.randint(1,3))
        res='%s%s' %(name,i)
        q.put(res)
        print('\033[44m%s 生产了 %s\033[0m' %(os.getpid(),res))
    q.join()


if __name__ == '__main__':
    q=JoinableQueue()
    #生产者们:即厨师们
    p1=Process(target=producer,args=('包子',q))
    p2=Process(target=producer,args=('骨头',q))
    p3=Process(target=producer,args=('泔水',q))

    #消费者们:即吃货们
    c1=Process(target=consumer,args=(q,))
    c2=Process(target=consumer,args=(q,))
    c1.daemon=True
    c2.daemon=True

    #开始
    p_l=[p1,p2,p3,c1,c2]
    for p in p_l:
        p.start()

    p1.join()
    p2.join()
    p3.join()
    print('主') 

    #主进程等--->p1,p2,p3等---->c1,c2
    #p1,p2,p3结束了,证明c1,c2肯定全都收完了p1,p2,p3发到队列的数据
    #因而c1,c2也没有存在的价值了,应该随着主进程的结束而结束,所以设置成守护进程
```

### 七 管道

进程间通信（IPC）方式二：管道（不推荐使用，了解即可）

介绍

```python
#创建管道的类：
Pipe([duplex]):在进程之间创建一条管道，并返回元组（conn1,conn2）,其中conn1，conn2表示管道两端的连接对象，强调一点：必须在产生Process对象之前产生管道
#参数介绍：
dumplex:默认管道是全双工的，如果将duplex射成False，conn1只能用于接收，conn2只能用于发送。
#主要方法：
    conn1.recv():接收conn2.send(obj)发送的对象。如果没有消息可接收，recv方法会一直阻塞。如果连接的另外一端已经关闭，那么recv方法会抛出EOFError。
    conn1.send(obj):通过连接发送对象。obj是与序列化兼容的任意对象
 #其他方法：
conn1.close():关闭连接。如果conn1被垃圾回收，将自动调用此方法
conn1.fileno():返回连接使用的整数文件描述符
conn1.poll([timeout]):如果连接上的数据可用，返回True。timeout指定等待的最长时限。如果省略此参数，方法将立即返回结果。如果将timeout射成None，操作将无限期地等待数据到达。

conn1.recv_bytes([maxlength]):接收c.send_bytes()方法发送的一条完整的字节消息。maxlength指定要接收的最大字节数。如果进入的消息，超过了这个最大值，将引发IOError异常，并且在连接上无法进行进一步读取。如果连接的另外一端已经关闭，再也不存在任何数据，将引发EOFError异常。
conn.send_bytes(buffer [, offset [, size]])：通过连接发送字节数据缓冲区，buffer是支持缓冲区接口的任意对象，offset是缓冲区中的字节偏移量，而size是要发送字节数。结果数据以单条消息的形式发出，然后调用c.recv_bytes()函数进行接收    

conn1.recv_bytes_into(buffer [, offset]):接收一条完整的字节消息，并把它保存在buffer对象中，该对象支持可写入的缓冲区接口（即bytearray对象或类似的对象）。offset指定缓冲区中放置消息处的字节位移。返回值是收到的字节数。如果消息长度大于可用的缓冲区空间，将引发BufferTooShort异常。
```

基于管道实现进程间通信（与队列的方式是类似的，队列就是管道加锁实现的）

```python
from multiprocessing import Process,Pipe

import time,os
def consumer(p,name):
    left,right=p
    left.close()
    while True:
        try:
            baozi=right.recv()
            print('%s 收到包子:%s' %(name,baozi))
        except EOFError:
            right.close()
            break
def producer(seq,p):
    left,right=p
    right.close()
    for i in seq:
        left.send(i)
        # time.sleep(1)
    else:
        left.close()
if __name__ == '__main__':
    left,right=Pipe()
    c1=Process(target=consumer,args=((left,right),'c1'))
    c1.start()
    seq=(i for i in range(10))
    producer(seq,(left,right))
    right.close()
    left.close()
    c1.join()
    print('主进程')
```

**注意：生产者和消费者都没有使用管道的某个端点，就应该将其关闭，如在生产者中关闭管道的右端，在消费者中关闭管道的左端。如果忘记执行这些步骤，程序可能再消费者中的recv\**()\**操作上挂起。管道是由操作系统进行引用计数的,必须在所有进程中关闭管道后才能生产EOFError异常。因此在生产者中关闭管道不会有任何效果，付费消费者中也关闭了相同的管道端点。**

管道可以用于双向通信，利用通常在客户端/服务器中使用的请求／响应模型或远程过程调用，就可以使用管道编写与进程交互的程序

```python
from multiprocessing import Process,Pipe

import time,os
def adder(p,name):
    server,client=p
    client.close()
    while True:
        try:
            x,y=server.recv()
        except EOFError:
            server.close()
            break
        res=x+y
        server.send(res)
    print('server done')
if __name__ == '__main__':
    server,client=Pipe()

    c1=Process(target=adder,args=((server,client),'c1'))
    c1.start()

    server.close()

    client.send((10,20))
    print(client.recv())
    client.close()

    c1.join()
    print('主进程')
#注意：send()和recv()方法使用pickle模块对对象进行序列化。
```

### 八 共享数据

展望未来，基于消息传递的并发编程是大势所趋

即便是使用线程，推荐做法也是将程序设计为大量独立的线程集合

通过消息队列交换数据。这样极大地减少了对使用锁定和其他同步手段的需求，

还可以扩展到分布式系统中

**进程间通信应该尽量避免使用本节所讲的共享数据的方式**

```python
进程间数据是独立的，可以借助于队列或管道实现通信，二者都是基于消息传递的

虽然进程间数据独立，但可以通过Manager实现数据共享，事实上Manager的功能远不止于此

A manager object returned by Manager() controls a server process which holds Python objects and allows other processes to manipulate them using proxies.

A manager returned by Manager() will support types list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, Value and Array. For example,
```

进程之间操作共享的数据

```python
from multiprocessing import Manager,Process,Lock
import os
def work(d,lock):
    # with lock: #不加锁而操作共享的数据,肯定会出现数据错乱
        d['count']-=1

if __name__ == '__main__':
    lock=Lock()
    with Manager() as m:
        dic=m.dict({'count':100})
        p_l=[]
        for i in range(100):
            p=Process(target=work,args=(dic,lock))
            p_l.append(p)
            p.start()
        for p in p_l:
            p.join()
        print(dic)
        #{'count': 94}
```

### 九 信号量(了解）

信号量Semahpore（同线程一样）

```python
互斥锁 同时只允许一个线程更改数据，而Semaphore是同时允许一定数量的线程更改数据 ，比如厕所有3个坑，那最多只允许3个人上厕所，后面的人只能等里面有人出来了才能再进去，如果指定信号量为3，那么来一个人获得一把锁，计数加1，当计数等于3时，后面的人均需要等待。一旦释放，就有人可以获得一把锁

    信号量与进程池的概念很像，但是要区分开，信号量涉及到加锁的概念

from multiprocessing import Process,Semaphore
import time,random

def go_wc(sem,user):
    sem.acquire()
    print('%s 占到一个茅坑' %user)
    time.sleep(random.randint(0,3)) #模拟每个人拉屎速度不一样，0代表有的人蹲下就起来了
    sem.release()

if __name__ == '__main__':
    sem=Semaphore(5)
    p_l=[]
    for i in range(13):
        p=Process(target=go_wc,args=(sem,'user%s' %i,))
        p.start()
        p_l.append(p)

    for i in p_l:
        i.join()
    print('============》')
```

### 十 事件(了解)

Event（同线程一样）

```python
python线程的事件用于主线程控制其他线程的执行，事件主要提供了三个方法 set、wait、clear。

    事件处理的机制：全局定义了一个“Flag”，如果“Flag”值为 False，那么当程序执行 event.wait 方法时就会阻塞，如果“Flag”值为True，那么event.wait 方法时便不再阻塞。

clear：将“Flag”设置为False
set：将“Flag”设置为True


#_*_coding:utf-8_*_
#!/usr/bin/env python

from multiprocessing import Process,Event
import time,random

def car(e,n):
    while True:
        if not e.is_set(): #Flase
            print('\033[31m红灯亮\033[0m，car%s等着' %n)
            e.wait()
            print('\033[32m车%s 看见绿灯亮了\033[0m' %n)
            time.sleep(random.randint(3,6))
            if not e.is_set():
                continue
            print('走你,car', n)
            break

def police_car(e,n):
    while True:
        if not e.is_set():
            print('\033[31m红灯亮\033[0m，car%s等着' % n)
            e.wait(1)
            print('灯的是%s，警车走了,car %s' %(e.is_set(),n))
            break

def traffic_lights(e,inverval):
    while True:
        time.sleep(inverval)
        if e.is_set():
            e.clear() #e.is_set() ---->False
        else:
            e.set()

if __name__ == '__main__':
    e=Event()
    # for i in range(10):
    #     p=Process(target=car,args=(e,i,))
    #     p.start()

    for i in range(5):
        p = Process(target=police_car, args=(e, i,))
        p.start()
    t=Process(target=traffic_lights,args=(e,10))
    t.start()

    print('============》')
```

### 十一 进程池

在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。多进程是实现并发的手段之一，需要注意的问题是：

1. 很明显需要并发执行的任务通常要远大于核数
2. 一个操作系统不可能无限开启进程，通常有几个核就开几个进程
3. 进程开启过多，效率反而会下降（开启进程是需要占用系统资源的，而且开启多余核数目的进程也无法做到并行）

例如当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生多个进程，十几个还好，但如果是上百个，上千个。。。手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。

我们就可以通过维护一个进程池来控制进程数目，比如httpd的进程模式，规定最小进程数和最大进程数... *ps：对于远程过程调用的高级应用程序而言，应该使用进程池，Pool可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，就重用进程池中的进程。*

**创建进程池的类：如果指定numprocess为3，则进程池会从无到有创建三个进程，然后自始至终使用这三个进程去执行所有任务，不会开启其他进程**

```python
1 Pool([numprocess  [,initializer [, initargs]]]):创建进程池
```

**参数介绍：**

```python
1 numprocess:要创建的进程数，如果省略，将默认使用cpu_count()的值
2 initializer：是每个工作进程启动时要执行的可调用对象，默认为None
3 initargs：是要传给initializer的参数组
```

**方法介绍：**

主要方法：

```python
1 p.apply(func [, args [, kwargs]]):在一个池工作进程中执行func(*args,**kwargs),然后返回结果。需要强调的是：此操作并不会在所有池工作进程中并执行func函数。如果要通过不同参数并发地执行func函数，必须从不同线程调用p.apply()函数或者使用p.apply_async()
2 p.apply_async(func [, args [, kwargs]]):在一个池工作进程中执行func(*args,**kwargs),然后返回结果。此方法的结果是AsyncResult类的实例，callback是可调用对象，接收输入参数。当func的结果变为可用时，将理解传递给callback。callback禁止执行任何阻塞操作，否则将接收其他异步操作中的结果。
3    
4 p.close():关闭进程池，防止进一步操作。如果所有操作持续挂起，它们将在工作进程终止前完成
5 P.jion():等待所有工作进程退出。此方法只能在close（）或teminate()之后调用
```

其他方法（了解部分）

```python
方法apply_async()和map_async（）的返回值是AsyncResul的实例obj。实例具有以下方法
obj.get():返回结果，如果有必要则等待结果到达。timeout是可选的。如果在指定时间内还没有到达，将引发一场。如果远程操作中引发了异常，它将在调用此方法时再次被引发。
obj.ready():如果调用完成，返回True
obj.successful():如果调用完成且没有引发异常，返回True，如果在结果就绪之前调用此方法，引发异常
obj.wait([timeout]):等待结果变为可用。
obj.terminate()：立即终止所有工作进程，同时不执行任何清理或结束任何挂起工作。如果p被垃圾回收，将自动调用此函数
```

**应用：**

同步调用apply

```python
from multiprocessing import Pool
import os,time
def work(n):
    print('%s run' %os.getpid())
    time.sleep(3)
    return n**2

if __name__ == '__main__':
    p=Pool(3) #进程池中从无到有创建三个进程,以后一直是这三个进程在执行任务
    res_l=[]
    for i in range(10):
        res=p.apply(work,args=(i,)) #同步调用，直到本次任务执行完毕拿到res，等待任务work执行的过程中可能有阻塞也可能没有阻塞，但不管该任务是否存在阻塞，同步调用都会在原地等着，只是等的过程中若是任务发生了阻塞就会被夺走cpu的执行权限
        res_l.append(res)
    print(res_l)
```

异步调用apply_async

```python
from multiprocessing import Pool
import os,time
def work(n):
    print('%s run' %os.getpid())
    time.sleep(3)
    return n**2

if __name__ == '__main__':
    p=Pool(3) #进程池中从无到有创建三个进程,以后一直是这三个进程在执行任务
    res_l=[]
    for i in range(10):
        res=p.apply_async(work,args=(i,)) #同步运行,阻塞、直到本次任务执行完毕拿到res
        res_l.append(res)

    #异步apply_async用法：如果使用异步提交的任务，主进程需要使用jion，等待进程池内任务都处理完，然后可以用get收集结果，否则，主进程结束，进程池可能还没来得及执行，也就跟着一起结束了
    p.close()
    p.join()
    for res in res_l:
        print(res.get()) #使用get来获取apply_aync的结果,如果是apply,则没有get方法,因为apply是同步执行,立刻获取结果,也根本无需get
```

详解：apply_async与apply

```python
#一：使用进程池（异步调用,apply_async）
#coding: utf-8
from multiprocessing import Process,Pool
import time

def func(msg):
    print( "msg:", msg)
    time.sleep(1)
    return msg

if __name__ == "__main__":
    pool = Pool(processes = 3)
    res_l=[]
    for i in range(10):
        msg = "hello %d" %(i)
        res=pool.apply_async(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
        res_l.append(res)
    print("==============================>") #没有后面的join，或get，则程序整体结束，进程池中的任务还没来得及全部执行完也都跟着主进程一起结束了

    pool.close() #关闭进程池，防止进一步操作。如果所有操作持续挂起，它们将在工作进程终止前完成
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束

    print(res_l) #看到的是<multiprocessing.pool.ApplyResult object at 0x10357c4e0>对象组成的列表,而非最终的结果,但这一步是在join后执行的,证明结果已经计算完毕,剩下的事情就是调用每个对象下的get方法去获取结果
    for i in res_l:
        print(i.get()) #使用get来获取apply_aync的结果,如果是apply,则没有get方法,因为apply是同步执行,立刻获取结果,也根本无需get

#二：使用进程池（同步调用,apply）
#coding: utf-8
from multiprocessing import Process,Pool
import time

def func(msg):
    print( "msg:", msg)
    time.sleep(0.1)
    return msg

if __name__ == "__main__":
    pool = Pool(processes = 3)
    res_l=[]
    for i in range(10):
        msg = "hello %d" %(i)
        res=pool.apply(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
        res_l.append(res) #同步执行，即执行完一个拿到结果，再去执行另外一个
    print("==============================>")
    pool.close()
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束

    print(res_l) #看到的就是最终的结果组成的列表
    for i in res_l: #apply是同步的，所以直接得到结果，没有get()方法
        print(i)
```

**练习2：使用进程池维护固定数目的进程（重写练习1）**

server端

```python
#Pool内的进程数默认是cpu核数，假设为4（查看方法os.cpu_count()）
#开启6个客户端，会发现2个客户端处于等待状态
#在每个进程内查看pid，会发现pid使用为4个，即多个客户端公用4个进程
from socket import *
from multiprocessing import Pool
import os

server=socket(AF_INET,SOCK_STREAM)
server.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
server.bind(('127.0.0.1',8080))
server.listen(5)

def talk(conn,client_addr):
    print('进程pid: %s' %os.getpid())
    while True:
        try:
            msg=conn.recv(1024)
            if not msg:break
            conn.send(msg.upper())
        except Exception:
            break

if __name__ == '__main__':
    p=Pool()
    while True:
        conn,client_addr=server.accept()
        p.apply_async(talk,args=(conn,client_addr))
        # p.apply(talk,args=(conn,client_addr)) #同步的话，则同一时间只有一个客户端能访问
```

客户端

```python
from socket import *

client=socket(AF_INET,SOCK_STREAM)
client.connect(('127.0.0.1',8080))


while True:
    msg=input('>>: ').strip()
    if not msg:continue

    client.send(msg.encode('utf-8'))
    msg=client.recv(1024)
    print(msg.decode('utf-8'))
```

发现：并发开启多个客户端，服务端同一时间只有3个不同的pid，干掉一个客户端，另外一个客户端才会进来，被3个进程之一处理

**回掉函数：**

**需要回调函数的场景：进程池中任何一个任务一旦处理完了，就立即告知主进程：我好了额，你可以处理我的结果了。主进程则调用一个函数去处理该结果，该函数即回调函数**

**我们可以把耗时间（阻塞）的任务放到进程池中，然后指定回调函数（主进程负责执行），这样主进程在执行回调函数时就省去了I/O的过程，直接拿到的是任务的结果。**

```python
from multiprocessing import Pool
import requests
import json
import os

def get_page(url):
    print('<进程%s> get %s' %(os.getpid(),url))
    respone=requests.get(url)
    if respone.status_code == 200:
        return {'url':url,'text':respone.text}

def pasrse_page(res):
    print('<进程%s> parse %s' %(os.getpid(),res['url']))
    parse_res='url:<%s> size:[%s]\n' %(res['url'],len(res['text']))
    with open('db.txt','a') as f:
        f.write(parse_res)


if __name__ == '__main__':
    urls=[
        'https://www.baidu.com',
        'https://www.python.org',
        'https://www.openstack.org',
        'https://help.github.com/',
        'http://www.sina.com.cn/'
    ]

    p=Pool(3)
    res_l=[]
    for url in urls:
        res=p.apply_async(get_page,args=(url,),callback=pasrse_page)
        res_l.append(res)

    p.close()
    p.join()
    print([res.get() for res in res_l]) #拿到的是get_page的结果,其实完全没必要拿该结果,该结果已经传给回调函数处理了

'''
打印结果:
<进程3388> get https://www.baidu.com
<进程3389> get https://www.python.org
<进程3390> get https://www.openstack.org
<进程3388> get https://help.github.com/
<进程3387> parse https://www.baidu.com
<进程3389> get http://www.sina.com.cn/
<进程3387> parse https://www.python.org
<进程3387> parse https://help.github.com/
<进程3387> parse http://www.sina.com.cn/
<进程3387> parse https://www.openstack.org
[{'url': 'https://www.baidu.com', 'text': '<!DOCTYPE html>\r\n...',...}]
'''
```

爬虫案例

```python
from multiprocessing import Pool
import time,random
import requests
import re

def get_page(url,pattern):
    response=requests.get(url)
    if response.status_code == 200:
        return (response.text,pattern)

def parse_page(info):
    page_content,pattern=info
    res=re.findall(pattern,page_content)
    for item in res:
        dic={
            'index':item[0],
            'title':item[1],
            'actor':item[2].strip()[3:],
            'time':item[3][5:],
            'score':item[4]+item[5]

        }
        print(dic)
if __name__ == '__main__':
    pattern1=re.compile(r'<dd>.*?board-index.*?>(\d+)<.*?title="(.*?)".*?star.*?>(.*?)<.*?releasetime.*?>(.*?)<.*?integer.*?>(.*?)<.*?fraction.*?>(.*?)<',re.S)

    url_dic={
        'http://maoyan.com/board/7':pattern1,
    }

    p=Pool()
    res_l=[]
    for url,pattern in url_dic.items():
        res=p.apply_async(get_page,args=(url,pattern),callback=parse_page)
        res_l.append(res)

    for i in res_l:
        i.get()

    # res=requests.get('http://maoyan.com/board/7')
    # print(re.findall(pattern,res.text))
```

**如果在主进程中等待进程池中所有任务都执行完毕后，再统一处理结果，则无需回调函数**

```python
from multiprocessing import Pool
import time,random,os

def work(n):
    time.sleep(1)
    return n**2
if __name__ == '__main__':
    p=Pool()

    res_l=[]
    for i in range(10):
        res=p.apply_async(work,args=(i,))
        res_l.append(res)

    p.close()
    p.join() #等待进程池中所有进程执行完毕

    nums=[]
    for res in res_l:
        nums.append(res.get()) #拿到所有结果
    print(nums) #主进程拿到所有的处理结果,可以在主进程中进行统一进行处理
```

[进程池的其他实现方式：https://docs.python.org/dev/library/concurrent.futures.htmldocs.python.org/dev/library/concurrent.futures.html](https://link.zhihu.com/?target=https%3A//docs.python.org/dev/library/concurrent.futures.html)

## 视频链接：

[python快速入门（一）_哔哩哔哩 (゜-゜)つロ 干杯~-bilibiliwww.bilibili.com/video/av73342471?p=131![img](https://pic4.zhimg.com/v2-c64ada0dd06d0c57ed905be65d17acb7_180x120.jpg)](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/av73342471%3Fp%3D131)