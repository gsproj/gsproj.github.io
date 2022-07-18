## **一：cpu和GIL必须都具备才可以执行代码；**

 拿到cpu权限-》拿到GIL解释器锁-》执行代码

 在python3.2之后GIL有了新的实现，目的是为了解决that GIL thrashing问题，这是Antoine Pitrou的功劳

## **二：GIL解释器锁会在两种情况下释放**

### **2.1、主动释放：自己主动交出来**

遇到IO操作或者分配的cpu时间片到时间了

注意，GIL存在的意义在于维护线程安全，x=10涉及到io操作，如果也被当成普通的io操作，主动交出GIL，那么一定会出现数据不安全问题，所以x=10一定是被区分对待了

至于x=10如何实现的被区分对待，这其实很好理解，任何的io操作都是向操作系统发送系统调用，即调用操作系统的某一接口实现的，比如变量赋值操作肯定是调用了一种接口，文件读写操作肯定也是调用了一种接口，网络io也是调用了某一种接口，这就给区分对待提供了实现的依据，即变量赋值操作并不属于主动释放的范畴，这样GIL在线程安全方面才会有所作为

### **2.2、被动释放**

python3.2之后定义了一个全局变量

```text
/* Python/ceval.c */
...
static volatile int gil_drop_request = 0;
```

注意当只有一个线程时，该线程会一直运行，不会释放GIL，当有多个线程时

例如thead1，thread2

如果thread1一直没有主动释放掉GIL，那肯定不会让他一直运行下去啊

实际上在thread1运行的过程时，thread2就会执行一个cv_wait(gil,TIMEOUT)的函数

（默认TIMEOUT值为5milliseconds，但是可以修改），一旦到了时间，就会将全局变量

gil_drop_request = 1;，线程thread1就会被强制释放GIL，然后线程thread2开始运行并

返回一个ack给线程thread1，线程thread1开始调用cv_wait(gil,TIMEOUT)

## **三：详见图解**

见Part4

**[http://www.dabeaz.com/python/Under](https://link.zhihu.com/?target=http%3A//www.dabeaz.com/python/UnderstandingGIL.pdf)**