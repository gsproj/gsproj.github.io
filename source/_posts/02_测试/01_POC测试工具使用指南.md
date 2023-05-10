---
title: 国产化POC测试工具使用指南
date: 2022-07-06 11:19:52
categories:
- 测试
- 信创POC测试
tags:
- 工作
---

# 一、性能测试

>测试台式机/笔记本的软硬件性能

## 1 开机时间

> 测试整机启动的时间（从按下电源开始计时，到出现登录界面计时结束）

测试步骤

```shell
手机计时
从按下电源的一刻到进入系统的时间
```



## 2 CPU性能测试（整型/浮点型）

>测试CPU整型计算性能，包括单线程基准值、满线程基准值
>
>测试CPU浮点计算性能，包括单线程基准值、满线程基准值

### 2.1 CPU2006

>测试环境:
>
>​	CPU：飞腾D2000
>
>​	内存：16G
>
>​	系统：麒麟V10-SP1-2107 桌面操作系统

1、准备

```shell
测试工具包：
d2000-spec2006test.tar.gz
```

2、测试步骤

```shell
1、把下载好的d2000-spec2006test.tar.gz测试包放到根目录下（建议整机内存≥16GB，否则会影响测试跑分）
#如果不知道如何放到根目录下，可以先把d2000-spec2006test.tar.gz这个包放到桌面，然后点击鼠标右键打开终端，sudo su ，然后输入密码进入root权限，然后输入cp -r d2000-spec2006test.tar.gz /  ，这样就复制到了根目录下。 
2、cd / ，进入根目录，sudo su ，然后输入密码，进入root权限。
3、解压，tar -xvf d2000-spec2006test.tar.gz
4、更改权限，chmod -R 777 cpu2006-1.2
5、进入spec2006目录，cd cpu2006-1.2
6、执行脚本，bash run.sh   #配置依赖环境，！！需要联网，会自动下载部分依赖！！
7、source /etc/profile  #更新profile文件,此处一定要更新
8、source ./shrc    #配置环境变量	
9、ulimit -s unlimited #内存分配无限制，大于或者等于16G内存的机器可以加此命令，如果是8G内存的机器不要运行此命令，会因内存不够用影响整型测试的429测试项
10、echo 3 > /proc/sys/vm/drop_caches   #执行测试前，建议清一下缓存
11、gcc -v   #测试前查看一下gcc的版本是否为GCC931,若不是GCC931，输入source /etc/profile， 再gcc -v查看版本，是GCC931即可执行第12步启动测试
12、bash d2000-test.sh  #执行8核、单核的整形和浮点测试
```

3、结果查看

```shell
PDF测试结果存放在results文件夹
```

4、编译问题处理（如遇到编译问题，可参考）

```shell
# 450报错：
# mpsinput.cc: In member function 'bool soplex::MPSInput::readLine()': mpsinput.cc:75:52: error: no match for 'operator==' (operand types are 'std::basic_istream<char>::__istream_type' {aka 'std::basic_istream<char>'} and 'int')
g++ -std=c++03

# lto1 fatal error bytecode stream generated with lto version 
编译选项添加: fno-lto

# 解决zfnlm问题:
actual argument contains too few elements for dummy argument 'zfnlm'
cfg文件编译选项设置： -std=legacy

# undefined reference to '__gcov_exit'
-lgcov --coverage

# glob.c:(.text+0x53c): undefined reference to `__alloca'
编辑glob.c文件
修改第54行"=="改为">="
# if _GNU_GLOB_INTERFACE_VERSION >= GLOB_INTERFACE_VERSION

# 2006的题483在v10-sp1-2107跑起来，我干了什么？ 否则老是报错483.xalancbmk  non-zero return code (rc=254, signal=0) XML等
1、安装统信UOS的GCC8.3.0
2、SPEC2006的工具集重新编译，再安装
3、清除题下面的run/build/exe
4、执行了一次apt-get install gcc?? 不知道有没有影响
5、放到没有中文的路径/home/gretwall/cpu2006
6、使用普通用户greatwall执行

# 416报错
# Note: The following floating-point exceptions are signalling: IEEE_UNDERFLOW_FLAG IEEE_DENORMAL

```

4、测试项说明

```shell
source shrc
# 多核测试（8）
runspec -c greatwall.cfg -r 8 -n 1 -i ref -T base -I all
选项解析：
	-c # 指定config/中的配置文件
	-r # --rate的缩写，跑rate测试，指定测试的副本数（Copies）
	--speed # 指定跑speed测试，和测试副本数（Copies）（默认跑speed测试）
	-n # 指定测试的次数	
	-i（-size） # 指定测试的规模，由小到大为test/train/ref,其中test最快，适合用来验证CPU的完整性，跑分要用ref
	--reportable / --noreportable # 表示检测/不检测生成的二进制文件是否修改过
	--action=build	# 只编译，不测试
	--rebuild	# 重新编译选题的二进制文件
	--tune（-T） # 可选peak / base / all，默认跑all
	最后的参数	# int-(int选题) / fp-(fp选题) / all-(所有选题) / 单个选题-（如483） 
```

5、去除PDF报告的Invalid标记

```shell
cp CINT2006.559.ref.rsf CINT2006.559.ref.rsf.retry
rawformat --flagsurl myfixedflags.xml --output_format pdf,raw CINT2006.559.ref.rsf.retry
```



### 2.2 CPU2017





## 3 lmbench测试（用时3小时）

>测试简单的系统调用时间、shell命令启动时间、系统信号处理时间、统计2p/16K的上下文切换性能、16p/64K的上下文切换性能、0K/10K文件创建时间、0K/10K文件删除时间

源码包准备：

```shell
wget http://www.bitmover.com/lmbench/lmbench3.tar.gz
```

编译前的准备：

```shell
tar xvf lmbench3.tar.gz
cd lmbench3
mkdir SCCS
touch  SCCS/s.ChangeSet
```

编译并测试

```she
make results OS=arch-linux
```

读取测试结果

```shell
make see
```

错误一：

```shell
报错: undefined reference to 'llseek'
解决方法:
修改src/disk.c文件
将llseek改为lseek64
```

错误二：

```shell
gmake[1]: Entering directory `/home/testcpu/test652/tools/lmbench3/src'
gmake[1]: *** No rule to make target `../SCCS/s.ChangeSet', needed by `bk.ver'.  Stop.
gmake[1]: Leaving directory `/home/testcpu/test652/tools/lmbench3/src'

解决方法：
	vim Makefile
	231行修改：
	$O/lmbench : ../scripts/lmbench bk.ver
	改为
	$O/lmbench : ../scripts/lmbench
```

## 4 文件读写测试

>测试硬盘内文件（10G）拷贝性能，记录时间

测试步骤

```shell
# 创建10G的大文件
dd if=/dev/zero of=big_file count=10 bs=1G

# 测试拷贝时间
time cp big_file  big_file_bak
```



## 5 USB存储设备读写性能

>测试USB存储设备读写性能（Mb/s），平均读写速度等

```shell
# 可选准备
1.将U盘（USB3.0）插入被测试机器,假定识别设备为sdc
2.创建vfat文件系统分区
/dev/sdb1分区容量大于30GB
umount /dev/sdc1
mkfs -t vfat /dev/sdc1
mkdir /upan
mount -t vfat /dev/sdc1 /upan
```

```shell
# 测试写性能
cd /upan
dd if=/dev/zero of=./largefile bs=64k count=10000

# 测试读性能
sync && echo 3 > /proc/sys/vm/drop_caches
dd if=./largefile of=/dev/null bs=64k
```



## 6 硬盘读写测试-IOZONE/

安装步骤：

```shell
# 下载源码包
wget http://www.iozone.org/src/current/iozone3_487.tar
# 解压
tar -vxf iozone3_487.tar && cd iozone3_489/src/current
# 编译
make linux
```

参数说明：

```shell
iozone
    -a 全面测试，比如块大小它会自动加
    -i N 用来选择测试项, 比如Read/Write/Random 比较常用的是0 1 2,可以指定成-i 0 -i 1 -i2.这些别的详细内容请查man
        0=write/rewrite
        1=read/re-read
        2=random-read/write
        3=Read-backwards
        4=Re-write-record
        5=stride-read
        6=fwrite/re-fwrite
        7=fread/Re-fread
        8=random mix
        9=pwrite/Re-pwrite
        10=pread/Re-pread
        11=pwritev/Re-pwritev
    	12=preadv/Re-preadv
    -r block size 指定一次写入/读出的块大小
    -s file size 指定测试文件的大小
    -f filename 指定测试文件的名字,完成后会自动删除(这个文件必须指定你要测试的那个硬盘中)
    -F file1 file2… 指定多线程下测试的文件名
    
    批量测试项:
    -g -n 指定测试文件大小范围,最大测试文件为4G,可以这样写 -g 4G
    -y -q 指定测试块的大小范围
    
    输出:
    下面是几个日志记录的参数.好像要输出成图象进行分析，需要指定-a的测试才能输出
    -R 产生Excel到标准输出
    -b 指定输出到指定文件上. 比如 -Rb ttt.xls
```

> 1、测试硬盘读写性能（Mb/s），包括随机和顺序读写平均读写速度（IOzone设置块大小16M，文件大小为物理内存2倍、1倍、1/2倍三组数据）

测试步骤

```shell
# 需要root权限
sudo su
# 块大小16M，文件大小为物理内存2倍（测试机器内存为8G的情况下）
./iozone -i 0 -i 1 -i 2 -s 16g -r 16m -f /iozone.tmpfile -Rb ./report/iotest_16G_0.xls
# 块大小16M，文件大小为物理内存1倍（测试机器内存为8G的情况下）
./iozone -i 0 -i 1 -i 2 -s 8g -r 16m -f /iozone.tmpfile -Rb ./report/iotest_8G_0.xls
# 块大小16M，文件大小为物理内存0.5倍（测试机器内存为8G的情况下）
./iozone -i 0 -i 1 -i 2 -s 4g -r 16m -f /iozone.tmpfile -Rb ./report/iotest_4G_0.xls
```

>2、测试数据盘（裸设备）的在块大小为512B、1MB下的读写性能（包括IOPS、带宽和响应时间）



## 7 内存读写性能测试-Stream

>测试单线和并发读写性能（Mb/s）

测试步骤

```shell
# 安装gfortran
sudo apt-get install gfortran

# 编译stream
make

# 测试
stream_c 200000000   # 参数为Array大小
```

多线程测试

```she
# 单线程编译
gcc -O3 stream.c -o stream
# 多线程编译
gcc -O3 -fopenmp stream.c -o stream.omp
```

## 10 操作系统综合性能测试-unixbench（用时1小时）

```shell
# 解压UnixBench工具包：
unzip Unixbench-Kylin.zip
# 进入Unixbench解压后的目录下
cd UnixBench
# 修改Makefile文件74行
OPTON = -O3 -fomit-frame-pointer -fforce-addr -ffast-math -Wall -static -flto
# 编译：
sudo make clean && make
# 清除缓存：
sudo su
sync
echo 3 > /proc/sys/vm/drop_caches
# 测试单线程和多线程性能，执行命令：
sudo ./Run -c 1 -c N
// 其中N代表cpu核数
```



## 11 显卡性能测试-unixbench（用时约半小时）

>1、测试2D显示处理性能，主要包括画点、画线、画三角形、画平行四边形、画正方形、画多边形等性能测试
>
>2、测试3D显示处理性能，主要包括3D的显示、色彩填充、渲染、旋转等性能测试

编译Unibench

```shell
# 安装依赖
apt-get install libgl1-mesa-dev
# 修改Makefile文件,防止编译报错
vim UnixBench/Makefile
# 第47行取消注释
GRAPHIC_TESTS = defined
# 第50行加上-lm
GL_LIBS = -lGL -lXext -lX11 -lm
# 第74行修改
OPTON = -O2 -fomit-frame-pointer -fforce-addr -ffast-math -Wall

# 修改Run文件
vim UnixBench/Run
# 第109-112行，修改可支持的最大核数，当前是8
'system'    => { 'name' => "System Benchmarks", 'maxCopies' => 8 },
'2d'        => { 'name' => "2D Graphics Benchmarks", 'maxCopies' => 8 },
'3d'        => { 'name' => "3D Graphics Benchmarks", 'maxCopies' => 8 },
'misc'      => { 'name' => "Non-Index Benchmarks", 'maxCopies' => 8 },
```

编译运行

```shell
make clean && make -j 8
./Run graphics
```

3D测试优化

```shell
# 创建文件10-vsync.conf
<driconf>
    <option name="vblank_mode" value="0" />
</driconf>

# 拷贝文件到指定路径
sudo cp 10-vsync.conf  ~/.drirc

# 运行测试
vblank_mode=0 ./Run ubgears
```

## 11 显卡测试-Glmark2

安装

```shell
sudo ./waf configure --with-flavors=x11-gl
sudo ./waf build -j 4
sudo ./waf install
```

测试

```shell
glmark2
```

## 12 显卡测试-Glxgears

安装

```she
sudo apt-get install mesa-utils
```

测试

```shell
# 终端执行
glxgears
```

## 12 网络性能测试-iperf3

>测试网络传输速率、重传等

可以用源安装或者编译安装

```shell
sudo apt-get install iperf3
```

常用参数：

```shell
-u：发送 UDP 包，仅客户端可用，服务端默认 tcp udp 都可以接收
-b：指定发送速率（比如 100M），发送端不受限速影响，如果有限速，也只是接收端有影响
-p：后接服务端监听的端口
-i：设置带宽报告的时间间隔，单位为秒
-t：设置测试的时长，单位为秒
-w：设置tcp窗口大小，一般可以不用设置，默认即可
-B：绑定客户端的ip地址
-4：指定 ipv4
-n：指定传输的字节数
-f：格式化带宽数输出，后接单位，比如 K，M
--get-server-output：在客户端直接获取服务端输出的结果
```

使用方法：

```shell
# 1、服务端10.47.74.25开启server服务
iperf3 -s

# 2、客户端10.47.76.198发起连接
iperf3 -c 10.47.74.25 -i 5 -t 30

# 3、如服务端拒绝连接请求，需考虑firewall或者selinux
```



