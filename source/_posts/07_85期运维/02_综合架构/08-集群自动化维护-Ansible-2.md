---
title: 08-集群自动化维护-Ansible(二)
date: 2024-5-4 16:35:52
categories:
- 运维
- （二）综合架构
tags: 
---

# Ansible集群自动化维护（二）

主要内容：剧本和变量

# 1 剧本

## 1.1 剧本介绍

什么是ansible的剧本(play book)？

- 剧本是一个yaml格式的文件，用于长久保存并且实现批量管理、维护、部署的文件。
- 类似于脚本存放命令和变量，剧本中存放各模块的使用命令
- 剧本是未来我们批量管理、运维必会的内容  

剧本和ansible命令的区别：

|                    | ansible剧本              | ans ad-hoc（单条命令） |
| ------------------ | ------------------------ | ---------------------- |
| 共同点             | 批量管理,使用模块        | 批量管理,使用模块      |
| 区别               | 方便重复调用             | 不是很方便，不容易重复 |
| 应用建议(应用场景) | 部署服务，多个步骤的任务 | 测试模块，临时性任务   |

## 1.2 剧本的书写格式

剧本的书写格式

![image-20240504152707076](../../../img/image-20240504152707076.png)

>书写playbook注意事项:
>
>- 同一个层级的内容对齐的.
>- 不同层级的通过2个空格对齐
>- 不能使用tab键  

## 1.3 第一个剧本

使用剧本在对象机器创建文件

```shell
# 01、创建剧本文件
[root@mn01[ /server/scripts/playbook]#cat 01-show.yml
---
- hosts: all
  tasks:
    - name: 01 打开冰箱门
      shell: echo 01 >> /tmp/bingxiang.log
    - name: 02 把大象放进冰箱
      shell: echo 02 >> /tmp/bingxiang.log
    - name: 03 关上冰箱门
      shell: echo 03 >> /tmp/bingxiang.log
      
# 02、执行剧本
[root@mn01[ /server/scripts/playbook]#ansible-playbook -i /etc/ansible/hosts 01-show.yml
 ____________
< PLAY [all] >
 ------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
...
ok: [172.16.1.7]
ok: [172.16.1.31]
ok: [172.16.1.41]
 _________________
< TASK [01 打开冰箱门] >
 -----------------
...

# 03、查看结果
[root@mn01[ /server/scripts/playbook]#ansible all -m shell -a "cat /tmp/bingxiang.log"
172.16.1.31 | CHANGED | rc=0 >>
01
02
03
172.16.1.7 | CHANGED | rc=0 >>
01
02
03
172.16.1.41 | CHANGED | rc=0 >>
01
02
03
```

>执行的时候有奶牛：
>
>可以删除软件 或 修改ansible.cfg配置进行关闭 #nocows = 1去掉注释即可  

## 1.4 剧本案例

### 1.4.1 案例01-创建目录

创建目录并分发文件，要求:

1. 创建目录/server/files/
2. /etc/hosts文件发送过去/server/files/  

剧本编写：

```shell
[root@mn01[ /server/scripts/playbook]#cat 02-disk-file.yml
- hosts: all
  tasks:
    - name: 01 创建目录
      file: path=/server/files/ state=directory
    - name: 02 分发文件
      copy: src=/etc/hosts dst=/server/files
```

以上剧本path和state仍写在一行，更像是一条命令，改进后如下：

```shell
[root@mn01[ /server/scripts/playbook]#cat 02-disk-file.yml
- hosts: all
  tasks:
    - name: 01 创建目录
      file:
        path: /server/files/
        state: directory
    - name: 02 分发文件
      copy:
        src: /etc/hosts
        dest: /server/files/
```

执行

```shell
[root@mn01[ /server/scripts/playbook]#ansible-playbook 02-disk-file.yml
 ____________
< PLAY [all] >
 ------------
...

ok: [172.16.1.31]
ok: [172.16.1.7]
ok: [172.16.1.41]
 ________________
< TASK [01 创建目录] >
 ----------------
...

```

### 1.4.2 案例02-安装软件

分发软件包、安装软件包、启动服务

步骤:

- zabbix-agent软件包(下载)
- 安装软件包
- 配置(略)
- 启动开机自启动    

实现如下：

```shell
# 01、创建剧本
[root@mn01[ /server/scripts/playbook]#cat 03-install-zabbix-agent.yml
- hosts: all
  tasks:
    - name: 01. 下载安装包到/tmp
      get_url:
        url: "https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/6.0/rhel/7/x86_64/zabbix-agent-6.0.7-1.el7.x86_64.rpm"
        validate_certs: no
        dest: /tmp/
    - name: 02. 安装软件包
      yum:
        name: /tmp/zabbix-agent-6.0.7-1.el7.x86_64.rpm
        state: present
    - name: 03. 配置
      debug:
        msg: "进行配置zabbix-agent"
    - name: 04. 启动
      systemd:
        name: zabbix-agent
        enabled: yes
        state: started
```

运行剧本

```shell
# 02、运行脚本
[root@mn01[ /server/scripts/playbook]#ansible-playbook 03-install-zabbix-agent.yml
 ____________
< PLAY [all] >
...
ok: [172.16.1.7]
ok: [172.16.1.31]
ok: [172.16.1.41]
 _______________________
< TASK [01. 下载安装包到/tmp] >
...
ok: [172.16.1.31]
ok: [172.16.1.41]
ok: [172.16.1.7]
 __________________
< TASK [02. 安装软件包] >
...
ok: [172.16.1.41]
ok: [172.16.1.31]
changed: [172.16.1.7]
 _______________
< TASK [03. 配置] >
 ---------------
...
ok: [172.16.1.7] => {
    "msg": "进行配置zabbix-agent"
}
ok: [172.16.1.31] => {
    "msg": "进行配置zabbix-agent"
}
ok: [172.16.1.41] => {
    "msg": "进行配置zabbix-agent"
}
 _______________
< TASK [04. 启动] >
...
ok: [172.16.1.31]
ok: [172.16.1.41]
changed: [172.16.1.7]
 ____________
< PLAY RECAP >
 ------------
```

### 1.4.3 案例03-部署服务

NFS服务

- nfs服务端:在backup上部署nfs服务,共享/backup-nfs目录,all_squash,匿名用户:nfsnobody
- nfs客户端:web挂载 /ans-upload目录挂载nfs服务端共享的/backup-nfs(永久挂载)  

流程梳理：

- 服务端流程:
  1. 部署nfs-utils,rpcbind
  2. 修改配置文件
  3. 创建共享目录并改所有者
  4. 启动服务rpcbind,nfs(注意顺序)
- 客户端流程:
  1. 安装nfs-utils
  2. 挂载与永久挂载  

实现：

```shell
[root@mn01[ /server/scripts/playbook]#cat 04-deploy-nfs.yml
# nfs服务端部署
- hosts: backup01
  tasks:
    - name: 01 部署nfs-utils, rpcbind
      yum:
        name: nfs-utils,rpcbind
        state: present
    - name: 02 修改配置文件
      lineinfile:
        path: /etc/exports
        line: "/backup-nfs 172.16.1.0/24(rw,all_squash)"
        create: true
    - name: 03 创建共享目录所有者并改权限
      file:
        path: /backup-nfs
        owner: nfsnobody
        group: nfsnobody
        state: directory
    - name: 04 启动服务rpcbind（注意顺序）
      systemd:
        name: rpcbind
        enabled: yes
        state: started
    - name: 05 启动服务nfs
      systemd:
        name: nfs
        enabled: yes
        state: started

# nfs客户端部署
- hosts: web01
  tasks:
    - name: 01 部署nfs-utils
      yum:
        name: nfs-utils
        state: present
    - name: 02 挂载nfs
      mount:
        src: 172.16.1.41:/backup-nfs
        path: /ans-upload
        fstype: nfs
        state: mounted
```



# 2 变量

ansible中额的变量无处不在,，ans中大部分地方都可以定义变量.

| 可以定义变量的地方                          | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| 在剧本文件中定义                            | 比较常用. 仅仅限于当前的play使用.                            |
| register变量(注册变量)                      | 实现例如脚本中`反引号`的功能,可以获取命令结果                |
| 变量文件（根据主机清单分分组进行定义变量 ） | 如果多个剧本,使用相同的变量,大型的剧本roles                  |
| inventory主机清单中定义变量                 | 未来可以用于批量修改主机使用,其他很少用了.                   |
| 命令行中                                    | 几乎不用.                                                    |
| facts变量                                   | 一般用于获取主机基本信息:ip,主机名,系统 (centos/ubuntu) 如果不需要可以关闭,用于加速剧本的执行 |

## 2.1 剧本中的变量

在剧本中使用变量：

1. 仅仅在当前play生效.
2.  一般用来存放路径,用户名,ip地址,类似于之前使用的脚本.
3. 注意引号使用.  

案例：批量创建/oldboy/lidao/upload/  

```shell
[root@mn01[ /server/scripts/playbook]#cat 05-vars.yml
- hosts: all
  # 定义变量
  vars:
    dir: /oldboy/lidao/upload
  tasks:
    - name: mkdir
      file:
        # 使用变量
        path: "{{dir}}"
        state: directory
```

>关于双引号的注意事项：
>
>```shell
>dir: /oldboy/lidao/upload/
>file:
>  path: "{{ dir }}" #这种要添加,变量是开头.
>file:
>  path: /oldboy-new/{{ dir }} #这种可以不加引号,变量不是开头.
>```

## 2.2 共用变量（变量文件）

将变量写入新建的变量文件vars.yml，然后在剧本中引用

```shell
[root@m01 /server/scripts/playbook]# cat 05.vars.yml
- hosts: all
  vars_files: ./vars.yml
  tasks:
    - name: file
      file:
        path: "{{ dir }}/{{ user }}-{{ file }}"
        state: touch
        
[root@m01 /server/scripts/playbook]# cat vars.yml
dir: /tmp/
file: lidao.txt
user: lidao996kkk
```

## 2.3 共用变量（根据主机组）

group_vars根据主机清单的分组去匹配变量文件，可以根据主机组创建变量文件，例如：

```shell
xxxx-check.yml
group_vars/
	lb/vars.yml #存放lb组的变量
	web/vars.yml #存放web组的变量
	data/vars.yml #存放xxx组的变量
	all/vars.yml #所有主机共用的变量
```

>未来一般使用all分组即可,把所有变量存放在一起,供剧本使用

案例：

```shell
# 创建主机组变量文件
[root@mn01[ /server/scripts/playbook]#cat group_vars/all/vars.yml
user: www
nfs_dir: /nfs_backup
web_mount_dir: /web_nfs
nfs_server: 172.16.1.41
rsync_pass: 1

# 调用变量文件
[root@mn01[ /server/scripts/playbook]#cat 07.group_vars.yml
- hosts: all
  tasks:
  - name: 测试group变量
    debug:
      msg: "变量内容 {{user}} {{rsync_pass}}"
- hosts: web01
  tasks:
  - name: 测试web组是否识别group变量
    debug:
      msg: "web组识别的变量变量内容 {{user}} {{rsync_pass}}"
```

执行：

```shell
[root@mn01[ /server/scripts/playbook]#ansible-playbook 07.group_vars.yml
 ____________
< PLAY [all] >
 ------------
...
ok: [172.16.1.7]
 __________________
< TASK [测试group变量] >
 ------------------
...
ok: [172.16.1.7] => {
    "msg": "变量内容 www 1"
}
...
< TASK [测试web组是否识别group变量] >
...
ok: [172.16.1.7] => {
    "msg": "web组识别的变量变量内容 www 1"
}
 ____________
< PLAY RECAP >
 ------------
```

## 2.4 facts变量  

什么是facts变量？

- 运行剧本的时候ans会收集每个主机的基本信息，这些信息形成的变量叫做facts变量。
- facts变量可以通过setup模块获取  

常用的fact变量

```she
ansible_hostname #主机名
ansible_memtotal_mb #内存大小(总计) 单位mb
ansible_processor_vcpus #cpu数量
ansible_default_ipv4.address #默认的网卡ip eth0
ansible_distribution #系统发行版本名字
CentOS Ubuntu Debian ՎՎʢ
ansible_processor_vcpus
ansible_processor_cores
ansible_date_time.date
```

案例：批量修改系统/etc/motd文件，登录的时候输出系统的基本信息，如

主机名、内存总大小、ip地址、发行版本、cpu数、核心数  

```shell
# 创建fatcs变量文件
[root@mn01[ /server/scripts/playbook]#cat templates/motd.j2
#######################################
welcome to oldboy elastic linux system
操作需谨慎,删根弹指间.
主机名: {{ ansible_hostname }}
ip地址: {{ ansible_default_ipv4.address }}
内存大小: {{ ansible_memtotal_mb }}
CPU数量: {{ ansible_processor_vcpus }}
核心总数: {{ ansible_processor_cores }}
发行版本: {{ ansible_distribution }}

# 分发
[root@mn01[ /server/scripts/playbook]#cat 08-change-motd.yml
- hosts: all
  tasks:
    - name: 分发motd文件
      template:
        src: templates/motd.j2
        dest: /etc/motd
        backup: yes

    - name: 分发motd文件
      copy:
        src: templates/motd.j2
        dest: /tmp/motd
        backup: yes
```

>**温馨提示: template vs copy模块**
>
>copy仅仅传输数据,复制文件
>
>template 传输数据,复制文件的时候,文件中的变量会被解析和运行  
>
>
>
>关于facts变量实际应用案例:
>
>1. 通过facts变量获取系统的基本信息
>2. 通过facts变量获取信息并进行判断
>3. 如果不需要可以进行关闭,加速剧本的运行( gather_facts: no)  

## 2.5 register变量

本质上就是用来实现脚本中的*反引号*功能，用户通过命令获取的内容都存放到Register变量中  

```shell
# 编写
[root@mn01[ /server/scripts/playbook]#cat 09-regvars.yml
- hosts: all
  tasks:
    - name: 获取日期
      shell: date +%F
      register: result
    - name: print result 变量内容
      debug:
        msg: |
             "register变量的全部内容是:{{ result.stderr }}"
             "register变量的精确的内容是:{{ result.stdout }}"
               
# 执行
[root@mn01[ /server/scripts/playbook]#ansible-playbook 09-regvars.yml
 ____________
< PLAY [all] >
 ------------
...
ok: [172.16.1.7] => {
    "msg": "\"register变量的全部内容是:\"\n\"register变量的精确的内容是:2024-05-04\"\n"
}
ok: [172.16.1.31] => {
    "msg": "\"register变量的全部内容是:\"\n\"register变量的精确的内容是:2024-05-04\"\n"
}
ok: [172.16.1.41] => {
    "msg": "\"register变量的全部内容是:\"\n\"register变量的精确的内容是:2024-05-04\"\n"
}
```

>符号说明：
>
>msg:中的`|`表示下面的内容是多行. `|`也可以用于其他模块中  