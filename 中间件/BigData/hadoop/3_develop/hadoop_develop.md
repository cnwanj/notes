### 1.NameNode启动流程

- 启动9870端口服务。
- 加载镜像文件和编辑日志。
- 初始化NameNode的RPC服务。
  - 用于NameNode和DataNode。
- NameNode启动资源检查。
  - 默认100M存储元数据。
- DataNode心跳超时判断。
  - 超过10分30秒表示DataNode挂了。
- 开启安全模式。
  - 默认0.999为启动块数量占比，启动块低于这个占比则不能启动。

![image-20220213154420158](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154420158.png)

![image-20220213154337993](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154337993.png)

### 2.DataNode启动流程

- 初始化DataXceiverServer。
  - 用于接收块信息请求（socket方式）。
- 初始化HTTP服务。
- 初始化DataNodeRPC服务端。
  - 用于接收客户端的访问。
- DataNode向NameNode注册。
  - 通过RPC通讯注册，将DataNode添加到心跳管理。
- DataNode向NameNode发送心跳。
  - 更新存储状态和心跳时间。

![image-20220213154141570](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154141570.png)

![image-20220213154200494](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154200494.png)

### 3.DFS上传流程

- 创建NameNode的RPC客户端。
  - NameNode添加文件到目录树中。
- 创建了数据流，包含一个等待队列。
  - 请求NameNode可以往哪个DataNode进行写入数据。
- 通知队列发送数据包到DataNode。
  - 将数据包发送到DataNode（socket请求），同时备份一份到ackQueue中。
  - DataNode服务接收到数据持久化到磁盘，并将数据发送到下一个DataNode中。
  - 所有的NameNode服务接收完毕返回应答，并将ackQueue的数据移除。

![image-20220213154104910](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154104910.png)

![image-20220213154231967](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154231967.png)

### 4.Yarn流程

- Yarn客户端向ResourceManager提交作业。
  - 申请一个Application。
  - RM返回提交资源路径给客户端。
  - 客户端将Job资源提交并生成切片、配置文件、jar程序。
- ResourceManager启动MRAppMaster。
  - 资源申请完毕后将用户请求初始化为一个Task。
  - 将任务放入队列中（FIFO），等待执行。
- 调度器任务执行（YarnChild）。
  - NodeManager领取RM的任务，创建容器。
  - 读取任务配置信息，若需要开启多个NM，需要向RM申请多个。
  - 通过YarnChild进程执行任务。

![image-20220213153906886](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213153906886.png)

![image-20220213154510626](https://gitee.com/cnwanj/cloud_image/raw/master/image/image-20220213154510626.png)



