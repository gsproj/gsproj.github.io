---
title: 国产化POC测试工具使用指南
date: 2022-07-06 11:19:52
categories:
- 测试
- 信创POC测试
tags:
- 工作
---

# POC测试工具使用指南

## 1、SPEC OMP 2012测试

SPEC OMP 2012是基于SPEC测试套件的OpenMP评测工具，其中包含15个基于OpenMP的并行程序。

1、安装工具

获取源包：omp2012-1.1.zip

解压后使用install.sh安装

```shell
# 安装
./install.sh -d /home/xx
```

2、测试步骤

```shell
source shrc
ulimit -s unlimited
export OMP_NUM_THREADS=48
runspec --config=gcc.cfg --threads 48 -n 3 -i ref -I all
```

3、补充参数说明

```shell
# 仅编译
runspec --action=build --config=gcc.cfg -i ref -I all
```

## 2 CPU性能测试（整型/浮点型）

>测试CPU整型计算性能，包括单线程基准值、满线程基准值
>
>测试CPU浮点计算性能，包括单线程基准值、满线程基准值

### 2.1 使用CPU2006

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

### 2.2 使用CPU2017

1、安装工具

获取镜像包：cpu2017-1_0_5.iso

挂载iso文件

```shell
mount -o loop -t iso9660 cpu2017-1_0_5.iso /xxx/cpu2017CD
```

安装工具

```shell
cd /xxx/cpu2017CD
./install.sh
```

2、测试步骤

spec2017主要分为四项测试：

- ​	intrate
- ​	fprate
- ​	intspeed
- ​	fpspeed

​	根据需要测试的cpu型号，可以到官网下载相应的cfg文件，修改icc、Jemalloc库、qkmalloc库的路径，放入config文件夹中。

测试命令如下：

```shell
source shrc
ulimit -s unlimited
# speed 测试
runcpu --config=icc-speed-official.cfg --threads=48 --define cores=48 -n 3 -i ref fpspeed intspeed

# rate 测试
runcpu --config=icc-rate-official.cfg -copies=48 -n 3 -i ref intrate fprate
```

补充参数说明

```shell
# 仅编译不跑测试
--action=build
# 重新编译
--rebuild
# 绑核跑
taskset -c 
```

对于Intel CPU的测试优化手段

```shell
1、Bios设置：
	开启超线程
	尝试开启LLC-PREFETCH
2、cfg文件中的submit可以设置绑核选项
```



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

>STREAM是一套综合性能测试程序集，通过fortran和C两种高级且高效的语言编写完成，由于这两种语言在数学计算方面的高效率， 使得 STREAM 测试例程可以充分发挥出内存的能力。Stream测试是内存测试中业界公认的内存带宽性能测试基准工具。
>
>用户测试内存单线程和并发读写性能（Mb/s）

1、编译安装

```shell
# 安装gfortran
sudo apt-get install gfortran
# 修改源码中的ArraySize
{最高级缓存X MB}×1024×1024×4.1×CPU路数/8，结果取整数
解释：由于stream.c源码推荐设置至少4倍最高级缓存，且STREAM_ARRAY_SIZE为double类型=8 Byte。所以公式为：最高级缓存(单位：Byte)×4.1倍×CPU路数/8
例如：测试机器是双路CPU，最高级缓存32MB，则计算值为32×1024×1024×4.1×2/8≈34393292
# 单线程编译
gcc -O3 stream.c -o stream
# 多线程编译
gcc -O3 -fopenmp stream.c -o stream_omp
```

测试

```shell
export OMP_NUM_THREADS=48/32/16/8/4  # 设置每个进程的线程数
./stream_omp 
```

如在intel平台，建议用ICC编译

```shell
icc -mtune=native -march=native -O3 -mcmodel=large -fopenmp -DSTREAM_ARRAY_SIZE=100000000 -DNTIMES=30 -DOFFSET=512 stream.c -o stream-gs
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

## 12 显卡测试-Glmark2

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

## 13 显卡测试-Glxgears

安装

```she
sudo apt-get install mesa-utils
```

测试

```shell
# 终端执行
glxgears
```

## 14 网络性能测试-iperf3

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

## 15 CPU、内存压测-stress

>stress是一款压力测试工具，可以用它来对系统CPU，内存，以及磁盘IO生成负载。

安装stress

```shell
apt-get install stress
```

使用stress

```shell
stress [option]

-? 显示帮助信息
-v 显示版本号
-q 不显示运行信息
-n，--dry-run 显示已经完成的指令执行情况
-t --timeout N 指定运行N秒后停止
   --backoff N 等待N微妙后开始运行
-c --cpu 产生n个进程 每个进程都反复不停的计算随机数的平方根
-i --io  产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上
-m --vm n 产生n个进程,每个进程不断调用内存分配malloc和内存释放free函数
   --vm-bytes B 指定malloc时内存的字节数 (默认256MB)
   --vm-hang N 指示每个消耗内存的进程在分配到内存后转入休眠状态，与正常的无限分配和释放内存的处理相反，这有利于模拟只有少量内存的机器
-d --hadd n 产生n个执行write和unlink函数的进程
   --hadd-bytes B 指定写的字节数，默认是1GB
   --hadd-noclean 不要将写入随机ASCII数据的文件Unlink
    
时间单位可以为秒s，分m，小时h，天d，年y，文件大小单位可以为K，M，G
```

使用stress进行CPU压测

```shell
# 128进程压测
stress -c 128
```

使用stress进行内存压测

```shell
# 创建三个进程，每个进程使用300M内存
stress -m 3 --vm-bytes 300M
```

使用stress进行磁盘压测

```shell
# stress -i N 会产生N个进程，每个进程反复调用sync()将内存上的内容写到硬盘上.
# stress -d N 会产生N个进程，每个进程往当前目录中写入固定大小的临时文件，然后执行unlink操作删除该临时文件。 临时文件的大小默认为1G，但可以通过 --hdd-bytes 设置临时文件的大小。比如
stress -i 2 -d 4 
```

同时对多个指标进行压力测试，只需要把上面的参数组合起来就行

## 16 NPB测试

NAS并行基准测试程序（NPB），是由美国航空航天局开发的一套代表流体动力学计算的应用程序集，它已经成为公认的用于测评大规模并行机和超级计算机的标准测试程序。NPB可用于常用的编程模型，如MPI和OpenMP。

1、工具安装

```shell
源包获取：
	NPB3.4.tar.gz         OpenMP || MPICH
	NPB3.4-MZ.tar.gz  OpenMP && MPICH
解压即可。
```

2、测试步骤

```shell
make suite
cd bin
mpirun -np 4 ./xxx
```

## 17 Linpack测试

> Linpack是国际上使用最广泛的测试高性能计算机系统浮点性能的基准测试。通过对高性能计算机采用高斯消元法求解一元 N次稠密线性代数方程组的测试，评价高性能计算机的浮点计算性能。Linpack的结果按每秒浮点运算次数（flops）表示。
>
>  N ^ 2 * 8 = 总内存

1、工具安装

```shell
源包获取：linpack.tgz
解压即可。
```

2、测试步骤

```shell
# 1进48线100G内存
./xhpl -n 1 -b 384 -p 1 -q 1 -m 100000
# 1进48线200G内存
./xhpl -n 1 -b 384 -p 1 -q 1 -m 200000
# 2进24线共100G内存
mpirun -n 2 ./xhpl -b 384 -p 1 -q 2 -m 50000
# 2进24线共200G内存
mpirun -n 2 ./xhpl -b 384 -p 1 -q 2 -m 100000
```

用mpirun提交xhplrun.sh效果更好

```shell
PRO_SIZE=${PMI_SIZE}
threads=`echo ${Cores}/${PMI_SIZE} | bc`
PMI_RANK_my=${PMI_RANK}
```

3、测试优化

>一、计算理论内存总带宽
>
>```shell
>计算方法：
>查看内存带宽：
>dmidecode | grep -A 16 "Memory Device"
>例如内存信息为：
>三星 DDR4 
>16G 每根
>2666 MT/s（Mhz）
>查看cpu支持的通道数，例如intel 6252N支持6通道，两个CPU支持12通道
>理论带宽计算DDR4：
>单条内存带宽 = 内存核心频率 x 内存总线位数 x 倍增系数
>	 = 2666 * 64 / 8 = 21328 MB/s = 21.3G/s
>12通道总内存带宽 = 21328 * 12 = 255936 = 250G/s
>```
>
>二、使用stream测试实际内存总带宽
>
>​	如果可以达到90%或以上，说明内存带宽正常, 继续Linpack测试
>
>三、计算CPU理论峰值
>
>```shell
>计算方法：
>	Mhz * 每个时钟周期执行浮点运算的次数 * CPU数目
>=
>	2.3 x ( 8 x 2 x 2 ) x 48 = 3532.8 Gflops
>说明：
>	峰值计算分为单精度和双精度浮点运算
>单精度：
>	32bit的指令长度的运算，对应32位操作系统
>双精度：
>	64bit的指令长度的运算，对应64位操作系统
>查找CPU可以处理什么样的指令集：
>	例如Intel官网查到Intel Xeon 6252N
>	支持AVX-512，
>	# of AVX-512 FMA Units = 2
>	(Fused Multiply Add instructions) 融合了 乘法 和 加法
>	即可以单个周期同时执行2条512bit的加法和2条512bit的乘法
>理解上述两个概念，可以开始计算（CPU单周期浮点计算能力）
>Intel 6252N (支持avx512，有512位)
>单精度：
>	2.3 x 512/32 x 2 x 2 = 147.2Gflops 
>双精度
>	2.3 x 512/64 x 2 x 2 = 73.6Gflops
>双精度共48核
>	74.6 x 48 = 3580
>
>FT2000+ 和 FT1500A (只有128位)，FT3000加入sve指令集(128-256位)
>单精度： 
>	2.2 x 128/32 * 2 = 4.4
>双精度:
>	2.2 x 128/64 * 2 = 8.8
>双精度共64核
>	8.8 x 64 = 563.2
>```
>
>四、使用Linapack测试CPU实际峰值
>
>```shell
>如果性能达到理论峰值的70%说明正常，如果未达到尝试以下优化
>```
>
>```shell
>1、使用mpirun运行xhpl, 按`lscpu` 中的NUMA分组绑定,`比如48核，分为两个Numa Node`, 则用mpirun`跑两个xhpl进程`，每个进程使用 OMP_NUM_THREADS=24, 跑24线程，这样可以将核用满。
>在`Intel`平台，两个进程分别使用 HPL_HOST_CORE='0-23' 以及 HPL_HOST_CORE='24-47' 绑定核
>在`Arm64`平台，则使用 GOMP_CPU_AFFINITY='0-23' 以及 GOMP_CPU_AFFINITY='24-47' 绑定
>2、内存不用分配太大，根据stream测试结果分配，从小开始测
>3、Intel平台编译器使用ICC，测试linpack使用icc自带的linpack，mpi使用icc自带的
>4、Bios中关闭超线程，开启超频(turbo/P-state)
>```
>
>**<font color="blue">先测stream,判断带宽是否正常，再测linpack，然后再测Spec Cpu</font>**

4、Intel平台

```shell
# Intel平台可以使用的控制频率的命令
cpupower -c all frequency-set -g performance
cpupower -c 0-95 frequency-set -g userspace
cpupower -c 0-95 frequency-set -f 2700000

# 查看当前频率控制是否为perfomance？
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# 跑xhpl之前设置指令集，或许有用
export MKL_ENABLE_INSTRUCTIONS=AVX512
```

5、HPL.dat修改

```shell
# 第1-2行，注释说明行
HPLinpack benchmark input file   
Innovative Computing Laboratory, University of Tennessee  

# 第 3-4 行，输出结果文件的形式
HPL.out      output file name (if any)  
6            device out (6=stdout,7=stderr,file) 

# 5-6行，求解矩阵的大小
1            # of problems sizes (N)  
1000         Ns   

# 7-8行 求解矩阵分块的大小
1            # of NBs
192 256      NBs

# 9行 处理器阵列的排列方式，行还是列
0：适用于节点较少，单个节点CPU较多的胖系统
1：适用于节点较多，单个节点CPU较少的瘦系统（机群上远好于按行排列）
1            PMAP process mapping (0=Row-,1=Column-major) 

# 10-12行 定义二维处理网格P/Q
# P X Q = 进程数，P尽量小于Q
# P=2^n,P最好选择2的幂
1            # of process grids (P x Q) 
1 2          Ps
1 2          Qs

# 13行 设置阀值，不用修改
16.0         threshold

# 14-21行 设置L的分解方式
1            # of panel fact
2 1 0        PFACTs (0=left, 1=Crout, 2=Right) # 对性能影响不大
1            # of recursive stopping criterium
4 8          NBMINs (>= 1)   # 4或8都不错
1            # of panels in recursion
2            NDIVs  # 选2比较理想
1            # of recursive panel fact.
1 0 2        RFACTs (0=left, 1=Crout, 2=Right) # 对性能影响不大

# 22-23行 设置L的横向广播方式
# 前四种，适用于快速网络，后两种适用于速度较慢的网络
# 小规模系统，选择0/1，大规模系统选择3
1            # of broadcast
0            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)

# 24-25行 设置横向通信的通信深度
# 小规模系统选择1/2，大规模系统2-5之间
1            # of lookahead depth
1            DEPTHs (>=0)

# 26-27 设置U的广播算法
# 小规模系统，使用缺省值即可
0            SWAP (0=bin-exch,1=long,2=mix)
1            swapping threshold

# 28-29行 L和U的数据存放格式
0：按列存放
1：按行存放
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form

# 30-31 缺省值即可
0            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
```

6、测试脚本编写

本地mpirun测试脚本（Intel版本）

```shell
# 本地单节点测试，需要根据`lscpu`中numa分区情况(node)的使用mpirun测试xhpl
# eg:在未开超线程的情况下，分为两个node，每个node中有12个核，则最佳测试方法为mpirun -np 2 ./myrun.sh
# 以下为myrun.sh的具体实现

#!/bin/bash
ulimit -s unlimited
Cores=48
ThreadsNum=`echo "${Cores}/${PMI_SIZE}" | bc` # 计算每个进程跑的线程数
case ${PMI_SIZE} in
2)
        case ${PMI_RANK} in
        0)
        export OMP_NUM_THREADS=${ThreadsNum}
        export HPL_HOST_CORE='0-23'
        ./xhpl -n 1 -b 384 -p ${1} -q ${2} -m ${3}
        ;;
        1)
        export OMP_NUM_THREADS=${ThreadsNum}
        export HPL_HOST_CORE='24-47'
        ./xhpl -n 1 -b 384 -p ${1} -q ${2} -m ${3}
        ;;
        esac
;;
esac
```

Slurm srun测试脚本(FT版本)

```shell
# xhplrun.sh
#!/bin/bash
ulimit -s unlimited
Cores=48


==================================================================
PMI_SIZE=$SLURM_NPROCS
PMI_RANK=$SLURM_PROCID
MPI_NUM_NODE=$SLURM_NNODES
MPI_PER_NODE=$((PMI_SIZE / MPI_NUM_NODES))
MPI_RANK_FOR_NODE=$((PMI_RANK % MPI_PER_NODE))

PRO_SIZE=${MPI_PER_NODE}
PMI_RANK_my=${MPI_RANK_FOR_NODE}
==================================================================

echo ${PRO_SIZE} ${PMI_RANK_my} ${threads} ${PMI_SIZE} ${PMI_RANK} ${MPI_NUM_NODES} ${MPI_PER_NODE} ${MPI_RANK_FOR_NODE}

export HPL_CMDLINE=1
case ${PRO_SIZE} in
1)
#numactl -i 0-1 -N 0-1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0,4 -N 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-63'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
case ${PMI_RANK_my} in
0)
#numactl -i 0-3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -m 3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-31'
numactl -i 4-7 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
#numactl -i 4-7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -m 7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-63'
numactl -i 0-3 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
#mpirun -n 1 numactl -i 0-3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3} : -n 1 numactl -i 4-7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
```

yhrun 提交作业 runpro.sh

```shell
#duqi

if [ $# -ne 5 ]
then
        echo Usage: ./runpro.sh nodes_list nodes_num proc_per_node mem_per_proc logdir
        exit 0
fi

export HPL_CMDLINE=1

export LD_LIBRARY_PATH=/usr/local/mpi3/lib:$LD_LIBRARY_PATH

nodes=${1}
nnodes=${2}
nprocs=${3}
nmem=${4}
logdir=${5}


mkdir -p ${logdir}

tnprocs=$((${nnodes}*${nprocs}))
P=1
Q=1
for i in `seq 1 ${tnprocs}`
do
        for j in `seq 1 ${tnprocs}`
        do
                PQ=$((${i}*${j}))
                if [ ${PQ} -eq ${tnprocs} ] && [ ${i} -le ${j} ]
                then
                        P=${i}
                        Q=${j}
                fi
        done
done

for i in `yhcontrol show hostname ${nodes}`
do
        scp HPL.dat $i:/root/linpack &
done
wait
sleep 1

j=0
runnodes=''

for i in `yhcontrol show hostname ${nodes}`
do
        runnodes+=${i},
        let j++
        if [ ${j} -eq ${nnodes} ]
        then
                echo "yhrun -p all -N ${nnodes} -n ${tnprocs} -w ${runnodes} -D /root/linpack /root/linpack/xhplrun.sh ${P} ${Q} ${nmem} &>> ${logdir}/${i}.log &"
                echo ${runnodes} &>> ${logdir}/${i}.log
                yhrun -p all -N ${nnodes} -n ${tnprocs} -w ${runnodes} -D /root/linpack /root/linpack/xhplrun.sh ${P} ${Q} ${nmem} &>> ${logdir}/${i}.log &
                j=0
                runnodes=''
        fi
done
```

bios设置

```shell
琦哥的：
    UPI Configuration
        Link Lop -> disable
        Link L1  -> disable 
        IO Directory Cache -> enable
        Isoc Mode -> enable

    Memory Configuration
        Eufsrce POR -> enable
        PPR Type -> Soft PPR
        Memory Frequency -> 2666
        Data Scrambling for NVMDIMM -> enable
        Data Scrambling for DDR4 -> enable
        Enable AOR -> enable
        Refresh Option -> enable
        Memory RAS
            Memory Rank Sparing -> enable

    IIO Configuration
        PCI-E Port Max Payload Size -> 256B

    CPU P-State
        SpeedStep -> disable/enable?
        PBF 
        Hardware P-State -> enable
        Package c state -> No limit 
	
网上找的系统Bios设置优化
    关闭超线程
    打开EIST
    打开Turbo Mode
    Boot Performance mode设置为max performance
    Energy Performance BIAS设置为Performance
    打开Monitor/Mwait
    Package C stat limit设置为C0/C1 state
    关闭CPU C3 report
    关闭CPU C6 report
    关闭Enhanced Halt State
    关闭Intel VT for Directed I/O
    Linux OS下CPU Power Management设置为max performance
    QPI及Memory Frequency保持为Max Frequency
    关闭NUMA功能 
```

7、结果反馈

```shell
1、grep FAILED 关键字,查看是否有节点计算错误
2、grep WR 查看节点性能是否正常
3、统计跑死的点

# 如何反馈？
一轮Linpack40分钟测试完成
	一、6个点本是drain*状态
	二、3个点跑死，cn[1,2,3]
	三、5个点Linpack计算FAILED
	四、其他点linpakc性能正常
可以考虑更换下一批
```

## 18 风冷系统linpack测试全过程记录

>##### 1、了解风冷系统的架构
>
>按批次测试，测完一批换下一批
>
>一批为192个节点，每个框32个节点，共6个框
>
>每个节点128G内存，芯片为FT2000+, 通过mhz获取频率为2000（降频），64物理核
>
>系统版本： 4.19.46-cn+
>
>##### 2、计算每个节点的理论峰值
>
>2000 * 128 / 64 * 2 * 64= 512000
>
>##### 3、开始测试
>
>runpro.sh脚本参数说明:
>
>```shell
>./runpro.sh NodeList NodeBind ProcessesPerNode MemSize
>NodeList 节点列表,例如cn[0-8]
>NodeBind 几个节点绑定运行，例如2，则计算效率时，需要将结果得到的效率/2/512
>ProcessesPerNode 每给节点运行几个xhpl进程，根据numa node来，FT一般是8
>MemSize 每个节点中，每个进程分配的内存，一般使用到约总内存的80%，例如单节点128G，填写12000（单位为Mb），则总使用96G内存
>```
>
>单节点测试：
>
>​	不需要互联测试网络联通性，仅对单节点进行压力测试
>
>```shell
>./runpro.sh cn0 1 8 12000 logdir
>```
>
>多节点测试：
>
>​	需要互联测试网络联通性， 2，4， 8， 16， 32， 64， 128，如果要求测多节点稳定性，测试的8点8进8线，结果按[章节6.8](#jump)反馈
>
>```shell
>./runpro.sh cn0 1 8 12000 logdir
>```
>
>问题解决一：
>
>​	测试到128节点，性能异常，效率仅有43%
>
>互联那边回应：
>
>​	节点数超过64，通信就跨框
>
>琦哥建议的做法：
>
>​	把128个节点分到4个框，测试一下，看是不是带宽的原因
>
>最后的解决方法：
>
>​	网络拓扑更换，矩阵值不能相等，使用原本32 x 32，改为16 x 64，但是换成160节点后还是有问题
>
>```shell
>./runpro.sh cn[xx-xx] 128 8 12000 dir
>其中执行的xhplrun.sh的参数为
>xhplrun.sh 32 32 12000
>```
>

## 19、附录

### 8.1 Linpack脚本

> runpro.sh
>
> ./runpro.sh nodes_list nodes_num proc_per_node mem_per_proc logdir
>
> node_list 节点列表
>
> nodes_num 几个点连在一起跑，单点 双点 多点
>
> proc_per_node 每个节点跑几个进程，看node分为几个，FT一般是8
>
> mem 12000 每个进程分配的内存，*8 大概= 总内存的80%
>
> logdir 日志文件
>
> 单点8进程，一轮≈35分钟

```shell
#! /bin/bash

#duqi

if [ $# -ne 5 ]
then
        echo Usage: ./runpro.sh nodes_list nodes_num proc_per_node mem_per_proc logdir
        exit 0
fi

export HPL_CMDLINE=1
export LD_LIBRARY_PATH=/usr/local/mpi3/lib:LD_LIBRARY_PATH

nodes=${1}
nnodes=${2}
nprocs=${3}
nmem=${4}
logdir=${5}


mkdir -p ${logdir}

tnprocs=$((${nnodes}*${nprocs}))
P=1
Q=1


num_max=1
for i in `seq 1 ${tnprocs}`
do
        j=$((${i}*${i}))
        if [ ${tnprocs} -le ${j} ]; then
                num_max=${i}
                break
        fi
done
for i in `seq 1 ${num_max}`
do
        for j in `seq ${num_max} ${tnprocs}`
        do
                PQ=$((${i}*${j}))
                if [ ${PQ} -eq ${tnprocs} ]
                then
                        P=${i}
                        Q=${j}
                fi
        done
done


for i in `yhcontrol show hostname ${nodes}`
do
        scp HPL.dat $i:/root/linpack &
done
sleep 1

j=0
runnodes=''

for i in `yhcontrol show hostname ${nodes}`
do
        runnodes+=${i},
        let j++
        if [ ${j} -eq ${nnodes} ]
        then
                echo "yhrun -p All -N ${nnodes} -n ${tnprocs} -w ${runnodes} -D /root/linpack /root/linpack/xhplrun.sh ${P} ${Q} ${nmem} &>> ${logdir}/${i}.log &"
                echo ${runnodes} &>> ${logdir}/${i}.log &
                echo "yhrun -p All -N ${nnodes} -n ${tnprocs} -w ${runnodes} -D /root/linpack /root/linpack/xhplrun.sh ${P} ${Q} ${nmem}" &>> ${logdir}/list.log
                yhrun -p All -N ${nnodes} -n ${tnprocs} -w ${runnodes} -D /root/linpack /root/linpack/xhplrun.sh ${P} ${Q} ${nmem} &>> ${logdir}/${i}.log &
                j=0
                runnodes=''
        fi
done

```

>xhplrun.sh

```shell
#! /bin/bash

#duqi

ulimit -s unlimited

Cores=64
#==========================================================
PMI_SIZE=$SLURM_NPROCS

PMI_RANK=$SLURM_PROCID

MPI_NUM_NODES=$SLURM_NNODES

MPI_PER_NODE=$((PMI_SIZE / MPI_NUM_NODES))

MPI_RANK_FOR_NODE=$((PMI_RANK % MPI_PER_NODE))
#==========================================================


threads=`echo ${Cores}*${SLURM_NNODES}/${PMI_SIZE} | bc`

PRO_SIZE=${MPI_PER_NODE}
PMI_RANK_my=${MPI_RANK_FOR_NODE}

echo ${PRO_SIZE} ${PMI_RANK_my} ${threads} ${PMI_SIZE} ${PMI_RANK} ${MPI_NUM_NODES} ${MPI_PER_NODE} ${MPI_RANK_FOR_NODE}

export HPL_CMDLINE=1
case ${PRO_SIZE} in
1)
#numactl -i 0-1 -N 0-1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0,4 -N 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-63'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
case ${PMI_RANK_my} in
0)
#numactl -i 0-3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -m 3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-31'
numactl -i 4-7 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
#numactl -i 4-7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -m 7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-63'
numactl -i 0-3 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
#mpirun -n 1 numactl -i 0-3 -N 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3} : -n 1 numactl -i 4-7 -N 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
4)
case ${PMI_RANK_my} in
0)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-15'
numactl -i 0-7 -N 0-1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 2-3 -N 0-1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='16-31'
numactl -i 0-7 -N 2-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 4-5 -N 2-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-47'
numactl -i 0-7 -N 4-5 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 6-7 -N 4-5 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
3)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='48-63'
numactl -i 0-7 -N 6-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0-1 -N 6-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
;;
8)
case ${PMI_RANK_my} in
0)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-7'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 4 -N 0 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='8-15'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 2 -N 1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='16-23'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 6 -N 2 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
3)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='24-31'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 1 -N 3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
4)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-39'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 0 -N 4 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
5)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='40-47'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 7 -N 5 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
6)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='48-55'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 5 -N 6 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
7)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='56-63'
numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#numactl -i 3 -N 7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
;;
16)
case ${PMI_RANK_my} in
0)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-3'
numactl -i 0-7 -C 0-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='4-7'
numactl -i 0-7 -C 4-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='8-11'
numactl -i 0-7 -C 8-11 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
3)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='12-15'
numactl -i 0-7 -C 12-15 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
4)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='16-19'
numactl -i 0-7 -C 16-19 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
5)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='20-23'
numactl -i 0-7 -C 20-23 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
6)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='24-27'
numactl -i 0-7 -C 24-27 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
7)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='28-31'
numactl -i 0-7 -C 28-31 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
8)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-35'
numactl -i 0-7 -C 32-35 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
9)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='36-39'
numactl -i 0-7 -C 36-39 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
10)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='40-43'
numactl -i 0-7 -C 40-43 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
11)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='44-47'
numactl -i 0-7 -C 44-47 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
12)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='48-51'
numactl -i 0-7 -C 48-51 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
13)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='52-55'
numactl -i 0-7 -C 52-55 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
14)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='56-59'
numactl -i 0-7 -C 56-59 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
15)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='60-63'
numactl -i 0-7 -C 60-63 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
;;
32)
#export OMP_NUM_THREADS=${threads}
#numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#;;
case ${PMI_RANK_my} in
0)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0-1'
numactl -i 0-7 -C 0-1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='2-3'
numactl -i 0-7 -C 2-3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='4-5'
numactl -i 0-7 -C 4-5 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
3)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='6-7'
numactl -i 0-7 -C 6-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
4)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='8-9'
numactl -i 0-7 -C 8-9 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
5)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='10-11'
numactl -i 0-7 -C 10-11 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
6)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='12-13'
numactl -i 0-7 -C 12-13 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
7)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='14-15'
numactl -i 0-7 -C 14-15 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
8)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='16-17'
numactl -i 0-7 -C 16-17 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
9)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='18-19'
numactl -i 0-7 -C 18-19 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
10)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='20-21'
numactl -i 0-7 -C 20-21 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
11)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='22-23'
numactl -i 0-7 -C 22-23 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
12)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='24-25'
numactl -i 0-7 -C 24-25 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
13)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='26-27'
numactl -i 0-7 -C 26-27 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
14)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='28-29'
numactl -i 0-7 -C 28-29 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
15)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='30-31'
numactl -i 0-7 -C 30-31 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
16)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32-33'
numactl -i 0-7 -C 32-33 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
17)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='34-35'
numactl -i 0-7 -C 34-35 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
18)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='36-37'
numactl -i 0-7 -C 36-37 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
19)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='38-39'
numactl -i 0-7 -C 38-39 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
20)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='40-41'
numactl -i 0-7 -C 40-41 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
21)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='42-43'
numactl -i 0-7 -C 42-43 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
22)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='44-45'
numactl -i 0-7 -C 44-45 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
23)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='46-47'
numactl -i 0-7 -C 46-47 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
24)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='48-49'
numactl -i 0-7 -C 48-49 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
25)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='50-51'
numactl -i 0-7 -C 50-51 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
26)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='52-53'
numactl -i 0-7 -C 52-53 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
27)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='54-55'
numactl -i 0-7 -C 54-55 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
28)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='56-57'
numactl -i 0-7 -C 56-57 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
29)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='58-59'
numactl -i 0-7 -C 58-59 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
30)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='60-61'
numactl -i 0-7 -C 60-61 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
31)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='62-63'
numactl -i 0-7 -C 62-63 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
;;
64)
export OMP_NUM_THREADS=${threads}
#export GOMP_CPU_AFFINITY='0-63'
#numactl -i 0-7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
#;;
case ${PMI_RANK_my} in
0)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='0'
numactl -i 0-7 -C 0 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
1)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='1'
numactl -i 0-7 -C 1 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
2)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='2'
numactl -i 0-7 -C 2 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
3)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='3'
numactl -i 0-7 -C 3 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
4)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='4'
numactl -i 0-7 -C 4 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
5)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='5'
numactl -i 0-7 -C 5 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
6)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='6'
numactl -i 0-7 -C 6 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
7)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='7'
numactl -i 0-7 -C 7 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
8)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='8'
numactl -i 0-7 -C 8 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
9)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='9'
numactl -i 0-7 -C 9 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
10)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='10'
numactl -i 0-7 -C 10 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
11)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='11'
numactl -i 0-7 -C 11 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
12)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='12'
numactl -i 0-7 -C 12 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
13)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='13'
numactl -i 0-7 -C 13 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
14)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='14'
numactl -i 0-7 -C 14 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
15)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='15'
numactl -i 0-7 -C 15 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
16)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='16'
numactl -i 0-7 -C 16 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
17)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='17'
numactl -i 0-7 -C 17 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
18)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='18'
numactl -i 0-7 -C 18 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
19)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='19'
numactl -i 0-7 -C 19 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
20)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='20'
numactl -i 0-7 -C 20 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
21)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='21'
numactl -i 0-7 -C 21 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
22)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='22'
numactl -i 0-7 -C 22 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
23)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='23'
numactl -i 0-7 -C 23 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
24)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='24'
numactl -i 0-7 -C 24 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
25)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='25'
numactl -i 0-7 -C 25 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
26)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='26'
numactl -i 0-7 -C 26 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
27)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='27'
numactl -i 0-7 -C 27 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
28)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='28'
numactl -i 0-7 -C 28 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
29)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='29'
numactl -i 0-7 -C 29 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
30)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='30'
numactl -i 0-7 -C 30 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
31)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='31'
numactl -i 0-7 -C 31 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
32)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='32'
numactl -i 0-7 -C 32 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
33)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='33'
numactl -i 0-7 -C 33 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
34)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='34'
numactl -i 0-7 -C 34 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
35)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='35'
numactl -i 0-7 -C 35 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
36)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='36'
numactl -i 0-7 -C 36 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
37)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='37'
numactl -i 0-7 -C 37 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
38)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='38'
numactl -i 0-7 -C 38 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
39)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='39'
numactl -i 0-7 -C 39 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
40)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='40'
numactl -i 0-7 -C 40 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
41)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='41'
numactl -i 0-7 -C 41 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
42)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='42'
numactl -i 0-7 -C 42 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
43)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='43'
numactl -i 0-7 -C 43 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
44)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='44'
numactl -i 0-7 -C 44 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
45)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='45'
numactl -i 0-7 -C 45 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
46)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='46'
numactl -i 0-7 -C 46 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
47)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='47'
numactl -i 0-7 -C 47 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
48)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='48'
numactl -i 0-7 -C 48 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
49)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='49'
numactl -i 0-7 -C 49 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
50)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='50'
numactl -i 0-7 -C 50 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
51)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='51'
numactl -i 0-7 -C 51 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
52)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='52'
numactl -i 0-7 -C 52 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
53)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='53'
numactl -i 0-7 -C 53 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
54)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='54'
numactl -i 0-7 -C 54 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
55)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='55'
numactl -i 0-7 -C 55 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
56)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='56'
numactl -i 0-7 -C 56 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
57)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='57'
numactl -i 0-7 -C 57 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
58)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='58'
numactl -i 0-7 -C 58 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
59)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='59'
numactl -i 0-7 -C 59 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
60)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='60'
numactl -i 0-7 -C 60 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
61)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='61'
numactl -i 0-7 -C 61 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
62)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='62'
numactl -i 0-7 -C 62 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
63)
export OMP_NUM_THREADS=${threads}
export GOMP_CPU_AFFINITY='63'
numactl -i 0-7 -C 63 ./xhpl -n 1 -b 192 -p ${1} -q ${2} -m ${3}
;;
esac
;;
esac
```

### 8.2 编译问题

```shell
-lc 
	解决找不到memcpy问题 
-lstdc++
	解决vtable for xxxx 问题
-lgcc_s
	undefined reference to `_Unwind_Resume'
	undefined reference to `__popcountdi2'
	
#include <sys/types.h>
#include <sys/types.h>
	 implicit declaration of function ‘fstat’

#include <sys/sysmacros.h>
	implicit declaration of function ‘major’
```

# 