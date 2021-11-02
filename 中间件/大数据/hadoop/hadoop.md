# 一、Hadoop概述

## 1.大数据的特点（4V）

> 按顺序给出数据存储单位: bit、Byte、KB、MB、GB、TB、PB、EB、ZB、YB、BB、NB、DB。
> 1Byte = 8bit、1K = 1024Byte、1MB = 1024K、1G = 1024M、1T = 1024G、1P = 1024T。

### 1.1 Volumn（大量）

目前，人类产生的所有印刷材料的数据是200PB，而历史上全人类总共说过的话的数据大约是5EB。当前，普通的计算机存储容量为TB量级，一些大企业的数据量达到了EB量级。

### 1.2 Velocity（高速）

根据IDC的“数字宇宙”报告，预计到2025年，全球数据的使用量达到163ZB，数据量的增长越来越快。

### 1.3 Variety（多样）

类型的多样性使得数据被分为结构化数据和非结构化数据，相对于以往便于存储的以数据库/文本为主的结构化数据，非结构化数据越来越多，包括网络日志、音频、视频、图片、地理位置等，从而对数据的处理能力提出了更高要求。

### 1.4 Value（低价值密度）

随着数据总量快速上升，价值密度随之变低。如何快速对有价值的数据“提纯”，成为目前大数据背景下要解决的难题。

## 2.大数据发展前景

党的十九大提出“推动互联网、大数据、人工智能和实体经济深度融合”。

## 3.大数据部门间业务流程

- 产品人员提需求（统计双11实时交易额、各个第五销售排行TopN等）。

- 数据部门搭建数据平台、分析数据指标。

- 数据可视化（报表展示、邮件发送、大屏幕展示等）。

![image-20210808220848297](upload/image-20210808220848297.png)

## 4.Hadoop是什么？

- Hadoop是由Apache基金会开发的分布式系统基础架构。
- 实现对海量数据存储和海量数据分析计算的解决方案。
- 通常指Hadoop生态圈（HBase、Hive、Zookeeper等）。

## 5.Hadoop组成

### 5.1HDFS概述

#### 5.1.1NameNode(nn)

存储文件元数据，如**文件名、文件目录结构、文件属性**（生成时间、副本数、文件权限），以及每个文件的块列表和块所在DataNode等。

#### 5.1.2DataNode(dn)

本地文件系统**存储文件块数据**，以及**块数据的校验和**。

#### 5.1.3Secondary NameNode(2nn)

每隔一段时间对NameNode元数据**备份**。

> NameNode：目录；DataNode：目录对应的内容

![image-20210823205741176](upload/image-20210823205741176.png)

### 5.2Yarn架构

另一种资源协调者（Yet Another Resource Negotiator）简称Yarn，是Hadoop的资源管理器。

- ResourceManager（RM）：整个集群资源（内存、CPU等）的管理者。
- NodeManager（NM）：单个节点服务器资源的管理者。
- Container：容器，提供独立的服务，封装了任务所需的资源（内存、CPU、磁盘、网络等）。
- ApplicationMaster（AM）：单个任务运行的管理者，运行在容器中。

![image-20210818234046682](upload/image-20210818234046682.png)

> - 客户端可以有多个。
> - 每个NodeManager有多个Container。
> - 每个Container中可以运行多个ApplicationMaster。

### 5.3MapReduce架构

MapReduce分为Map和Reduce两个阶段

- Map阶段并行处理输入数据。
- Reduce阶段对Map结果进行汇总。

![image-20210818231618091](upload/image-20210818231618091.png)

### 5.4HDFS、Yarn、MapReduce之间的关系

![image-20210823210841865](upload/image-20210823210841865.png)

### 5.5大数据的生态体系

![image-20210823212307215](upload/image-20210823212307215.png)

- 业务模型层：业务模型、数据可视化、业务应用。
- 任务调度层：
  - Oozie任务调度。
  - Azkaban任务调度。
- 数据计算层：
  - MapReduce离线计算（Hive数据查询）。
  - Spark Core内存计算（Spark Mlib数据挖掘、Spark Sql数据查询、Spark Streaming实时计算）。
  - flink。
  - Storm实时计算。
- 资源管理层：Yarn资源调度管理。
- 数据存储层：
  - HDFS文件存储。
  - HBase非关系数据库。
  - Kafka消息队列。
- 数据传输层：
  - Sqoop数据传递。
  - Flume日志收集。
  - Kafka消息队列。
- 数据来源层：
  - 数据库（结构化数据）。
  - 文件日志（半结构化数据）。
  - 视频和ppt等（非结构化数据）。

# 二、Linux CentOS7环境准备

## 1.CentOS7连接外部网络

> 准备：在VMWare中安装上CentOS7

### 1.1VMWare配置

（1）选择“虚拟网络编辑器”

![image-20210901225107805](upload/image-20210901225107805.png)

（2）选择“VMnet8”，点击“更改设置”

![image-20210901225332637](upload/image-20210901225332637.png)

（3）输入子网IP：192.168.1.2，子网掩码：255.255.255.0

![image-20210909222329757](upload/image-20210909222329757.png)

（4）点击“NAT设置”

![image-20210909222411417](upload/image-20210909222411417.png)

（5）输入网关IP：192.168.1.2，点击确定，这样就完成了VMWare的配置了。

![image-20210909222434617](upload/image-20210909222434617.png)

### 1.2Win10的配置

（1）选择VMnet8，鼠标右键，选择属性

![image-20210901230025989](upload/image-20210901230025989.png)

![image-20210901230135739](upload/image-20210901230135739.png)

（2）配置如下，配置完成点击确定

IP地址：192.168.1.1

子网掩码：255.2525.255.0

默认网关：192.168.1.2

DNS(P)：192.168.1.2

DNS(A)：8.8.8.8

![image-20210909222528880](upload/image-20210909222528880.png)

### 1.3CentOS7配置

（1）进入root模式，输入密码

```shell
su root
```

（2）修改配置文件（注意，一定要进入root模式，否则打开空白）

```shell
vim /etc/sysconfig/network-script/ifcfg-ens32
```

（3）修改配置内容

```shell
# 修改BOOTPROTO为static，IP地址为静态模式
BOOTPROTO=static

# 加入以下地址
IPADDR=192.168.1.5		# IP地址，ifconfig命令查看
NETMASK=255.255.255.0	# 子网掩码
GATEWAY=192.168.1.2		# 网关
DNS1=192.168.1.2		# 域名解析器
```

![image-20210909222813273](upload/image-20210909222813273.png)

（4）重启后，验证网络

```shell
# 重启
reboot
# 验证网络
ping www.baidu.com
```

![image-20210901231342178](upload/image-20210901231342178.png)

这样网络就配置成功了！

### 1.4XShell连接

> 注意：设置的IP地址（IPADDR）第三位要与网关（GATWAY）一致，如我这里的第三位都是“1”。
>
> IPADDR=192.168.1.5
> GATEWAY=192.168.1.2
>
> 否则xshell不能通过ip地址连接上主机CentOS

![image-20210909223437022](upload/image-20210909223437022.png)

连接上输入用户名和密码就可以连接成功了。

## 2.hosts文件映射

- 进入win10路径：C:\Windows\System32\drivers\etc
- 打开hosts文件
- 将下面映射复制到hosts文件中

```
192.168.1.5 hadoop5
192.168.1.6 hadoop6
192.168.1.7 hadoop7
192.168.1.8 hadoop8
192.168.1.9 hadoop9
192.168.1.10 hadoop10
192.168.1.11 hadoop11
192.168.1.12 hadoop12
```

## 3.本地环境

### 3.1准备本地环境

（1）安装epel-release软件包

相当于一个软件仓库、大多数rpm包在官方的repository是找不到的。

```shell
yum install -y epel-release
```

安装成功如下：

![image-20210909225707142](upload/image-20210909225707142.png)

（2）关闭防火墙

```shell
# 关闭防火墙
systemctl stop firewalld

# 关闭开机自启
systemctl disable firewalld.service
```

> 注意切换到root权限。

（3）创建用户，并修改用户密码

```shell
useradd weiyh
passwd 123456
```

（4）给用户加上root权限，方便后期用sudo执行root命令

```shell
# 编辑配置文件
vim /etc/sudoers

# 在文件的%wheel后面加上，NOPASSWD: 表示免密码执行
weiyh ALL=(ALL) NOPASSWD:ALL
```

![image-20210915221450216](upload/image-20210915221450216.png)

（5）在opt创建文件，并修改所属用户和组

```shell
# 创建文件夹
mkdir tools
mkdir software

# 修改改所属用户和组
sudo chown weiyh:weiyh tools/ install/

# 查看tools所属用户和组
drwxr-xr-x. 2 root  root  41 4月  18 2020 tomcat
drwxr-xr-x. 2 weiyh weiyh  6 9月  15 22:18 tools
```

（6）卸载自带的jdk

```shell
rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
# rpm -qa: 查询安装的所有rpm软件包
# grep -i: 忽略大小写
# xargs -n1: 每次只传递一个参数
# rpm -e --nodeps: 强制卸载软件
```

> 注意切换到root权限。

### 3.1克隆虚拟机

（1）将已创建的虚拟机克隆出三个hadoop102、hadoop103、hadoop104。

![image-20210921141719322](upload/image-20210921141719322.png)

其他的默认选择，下面选择第二个：

![image-20210921141810227](upload/image-20210921141810227.png)

最后选择克隆的目录。

（2）重复按照以上操作克隆出3个虚拟机，并修改对应的IP和名称如下：

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens32
```



![image-20210921144942435](upload/image-20210921144942435.png)

```shell
vim /etc/hoatname
```



![image-20210921145027026](upload/image-20210921145027026.png)

修改完成后进行重启，IP地址修改成功如下：

![image-20210921145222336](upload/image-20210921145222336.png)

### 3.1安装jdk

```shell
# 解压jdk到install文件下
tar -zxvf jdk-8u241-linux-x64.tar.gz -C ../install

# 编辑文件，配置jdk
sudo vim /etc/profile.d/my_env.sh

# 添加内容如下
export JAVA_HOME=/opt/install/jdk1.8.0_241
export PATH=$PATH:$JAVA_HOME/bin

# 刷新配置文件
source /etc/profile
```

![image-20210921224932191](upload/image-20210921224932191.png)

![image-20210921224900782](upload/image-20210921224900782.png)

> 为什么刷新profile文件就可以了？
>
> 因为profile文件会去循环遍历读取profile.d目录下的*.sh文件，如下：

![image-20210921225204953](upload/image-20210921225204953.png)

### 3.1安装hadoop

> hadoop的安装配置和jdk差不多

```shell
# 解压hadoop到install安装目录下
sudo tar -zxvf hadoop-3.1.3.tar.gz -C ../install/

# 配置/etc/profile.d/my_env.sh文件
export HADOOP_HOME=/opt/install/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

# 刷新profile文件
source /etc/profile
```

配置成功如下：

![image-20210921230215969](upload/image-20210921230215969.png)

# 三、运行模式

## 1.三种运行模式

- Local（Standalone）Mode：本地模式，单机运行，数据存储在linux本地。
- Pseudo-Distributed Mode：伪分布式，单机运行，具备Hadoop的多有功能，数据存储在HDFS。
- Fully-Distribute Mode：完全分布式，数据存储在HDFS，并且多台服务器工作。

## 2.本地运行模式

### 2.1统计词数

```shell
# 进入hadoop安装目录下新建文件
mkdir wcinput

# 在wcinput文件夹中新建文本
vim word.txt
# 内容如下：
zs zs
ls ls
banzhang
fanfan
yangge

# 回到hadoop目录下，统计词数到wcoutput文件中
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount wcinput/ ./wcoutput
	-- 通过hadoop命令运行jar包hadoop-mapreduce-examples-3.1.3.jar
	-- 统计wcinput中的词数到wcoutput中
	-- 注意：若输出文件wcoutput存在，会报错
	
# 查看wcoutput中的文件
cat wcoutput/part-r-00000
# 内容如下：
banzhang	1
fanfan	1
ls	2
yangge	1
zs	2
```

运行成功输出到wcoutput文件如下：

![image-20210923224456819](upload/image-20210923224456819.png)

查看统计文件如下：

![image-20210923224701973](upload/image-20210923224701973.png)

## 3.完全分布式运行模式

### 3.1编写虚拟机分发脚本xsync

#### 3.1.1scp安全拷贝

> scp（secure copy）可以实现服务器之间的数据拷贝（from server1 to server2）

基本语法：

（1）从当前服务器推到其他服务（copy to）

```shell
# 在hadoop102中操作，拷贝jdk到hadoop103服务中的目录
scp -r /opt/install/jdk1.8.0_241 weiyh@hadoop103:/opt/install/
	-- -r：递归
	-- hadoop103：服务名称
	-- weiyh：用户名
	
# 拷贝当前目录下的hadoop到hadoop103中
scp -r /opt/install/hadoop-3.1.3 weiyh@hadoop103:/opt/install/
```

（2）从其他服务器拉到当前服务（copy from）

```shell
# 在hadoop103中操作，从hadoop102拉取文件到当前服务
scp -r weiyh@hadoop102:/opt/install/hadoop-3.1.3 /opt/install/
```

（3）在hadoop103中，将102拷贝到104中

```shell
# 在hadoop103中操作，将hadoop102的文件拷贝到hadoop104中
scp -r weiyh@hadoop102:/opt/install/* weiyh@hadoop103:/opt/install/
```

> 若报错如下：
>
> Warning: Permanently added 'hadoop102,192.168.10.102' (ECDSA) to the list of known hosts.
> weiyh@hadoop102's password: 
> .//jdk1.8.0_241: Permission denied
>
> 需要给install文件夹设置权限如下：
>
> sudo chwon weiyh:weiyh install/

#### 3.1.2rsync远程同步工具

> - rsync（remote sync）主要用于备份和镜像，具有速度快、避免复制相同内容和支持符号链接的有点
> - rsync和scp区别：scp相对比较慢，rsync只对差异文件做更新同步，scp是把福哦呦都复制过去。

```shell
# 在hadoop102中，将hadoop102中的hadoop-3.1.3文件同步到hadoop103中的hadoop-3.1.3文件
rsync -av hadoop-3.1.3/ weiyh@hadoop103:/opt/install/hadoop-3.1.3
	-- a：复制归档
	-- v：显示过程
```

#### 3.1.3xsync集群分发脚本

> 用于循环复制文件到所有节点的相同目录下。

将如下脚本复制到系统的bin目录下

```shell
#!/bin/bash

# 1.判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

# 2.遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3.遍历所有目录，挨个发送

    for file in $@
    do
        # 4.判断文件是否存在
        if [ -e $file ]
            then
                # 5.获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                # 6.获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```

执行如下命令变可执行脚本：

```shell
# 将目录变成可执行脚本
chmod 777 xsync

# 将bin目录及文件复制到其他节点
xsync bin/
```

执行成功后如下：

![image-20211102222607540](upload/image-20211102222607540.png)

若需要复制权限文件（profile.d/下的配置文件），可以执行命令：

```shell
# 将my_env.sh文件复制到其他系统
sudo ./bin/xsync /etc/profile.d/my_env.sh

# 刷新配置文件
source /etc/profile
```

#### 3.1.4SSH免密访问

![image-20211102224645874](upload/image-20211102224645874.png)

- 将A的公钥放到B中，B将允许A免密访问本机。
- 当A访问B时，A携带私钥访问B，B解密A的私钥与其公钥进行匹配，成功后将加密数据返回给A，A再用私钥进行解密。

```shell
# 查看root目录下的隐藏文件
ls -al

# 在.ssh目录下，生成秘钥和公钥（执行后记性3次回车）
ssh-keygen -t rsa
```

![image-20211102230343161](upload/image-20211102230343161.png)

id_rsa：秘钥，id_rsa.pub：公钥

```shell
# 将秘钥和公钥拷贝到hadoop103和104
ssh-copy-id hadoop103
ssh-copy-id hadoop104
```

执行后，hadoop103和104的.ssh目录下存在如下文件，可以直接使用"ssh hadoop103"命令免密登录操作。

![image-20211102230718735](upload/image-20211102230718735.png)

> 添加秘钥后，可以直接使用xsync命令来复制文件

























