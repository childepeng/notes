# Hadoop部署

Hadoop官网，https://hadoop.apache.org，建议下载版本：2.10.*

```shell
# 自行选择下载速度块的镜像
wget https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz
tar xvf hadoop-2.10.1.tar.gz
```

## 单机部署

机器环境：

> IP: 172.18.0.11
>
> 操作系统：　CentOS 7
>
> Java：　OpenJDK 8
>
> Hadoop部署目录：　/home/hadoop-2.10.1

部署说明：

1. 设置环境变量，当前用户建议修改".bashrc", 全局安装修改“/etc/profile”

   ```shell
   # 配置Java环境变量
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk
   export PATH=$PATH:$JAVA_HOME/bin
   
   # 配置Hadoop环境变量
   export HADOOP_HOME=/home/hadoop/hadoop-2.10.1
   export HADOOP_PREFIX=$HADOOP_HOME
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
   export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
   export HADOOP_INSTALL=$HADOOP_HOME
   export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib:$HADOOP_COMMON_LIB_NATIVE_DIR"
   ```

   

2. 修改配置文件，`./etc/hadoop`，以下配置仅为最简化配置，更多配置参考官方文档　[https://hadoop.apache.org/docs/r2.10.1/](https://hadoop.apache.org/docs/r2.10.1/)

   **hadoop-env.sh**

   ```shell
   # 修改JAVA_HOME
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk
   ```

   **core-site.xml**

   ```xml
   <!-- hdfs文件 -->
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://172.18.0.11:9000</value>
   </property>
   
   <!-- 临时目录 -->
   <property>
   	<name>hadoop.tmp.dir</name>
       <value>/home/hadoop/hadoop-2.10.1/tmp</value>
   </property>
   ```

   **hdfs-site.xml**

   ```xml
   <!-- 副本数量 -->
   <property>
   	<name>dfs.replication</name>
       <value>1</value>
   </property>
   <!-- namenode数据目录 -->
   <property>
       <name>dfs.namenode.name.dir</name>
       <value>/home/hadoop/hadoop-2.10.1/namenode</value>
   </property>
   <!-- datanode数据目录 -->
   <property>
       <name>dfs.datanode.name.dir</name>
       <value>/home/hadoop/hadoop-2.10.1/datanode</value>
   </property>
   <property>
       <name>dfs.permissions</name>
   	<value>false</value>
   </property>
   <!-- 开启web后台 -->
   <property>
       <name>dfs.webhdfs.enabled</name>
       <value>true</value>
   </property>
   
   ```

   **mapred-site.xml**

   ```xml
   <!-- 任务调度框架，默认使用yarn -->
   <property>
   	<name>mapreduce.framework.name</name>
       <value>yarn</value>
   </property>
   ```

   **yarn-site.xml**

   ```xml
   <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>172.18.0.11</value>
   </property>
   
   <property>
       <name>yarn.resourcemanager.webapp.address</name>
       <value>172.18.0.11:8088</value>
   </property>
   
   ```

3. 启动，后台地址：http://192.168.1.30:50070/

   ```shell
   # 1. 初始化
   ./bin/hdfs namenode -format
   # 2. 启动
   ./sbin/start-dfs.sh
   ./sbin/start-yarn.sh
   ```

## 集群部署

### 集群环境

至少准备3台机器或者容器实例；IP和hostname按如下设置

```
172.18.0.11	hdp01
172.18.0.12	hdp02
172.18.0.13	hdp03
```

基础配置

1. 修改hostname

   ```shell
   sudo hostname hdp01
   sudo echo "hdp01" > /etc/hostname
   ```

2. 修改hosts

   ```sh
   echo "hdp01 172.18.0.11\nhdp02 172.18.0.12\nhdp03 172.18.0.13" >> /etc/hosts
   ```

3. 设置免密登录

   ```bash
   # 1 安装ssh服务（已安装的跳过）
   yum install openssh-server openssh-clients
   
   # 2 创建密钥对，并将公钥复制到远程主机
   ssh-keygen -t rsa -C "peng@laop.cc"
   cat /root/.ssh/id_rsa.pub > /root/.ssh/authorized_keys
   ssh-copy-id -p 22 root@hostname[ip]
   
   # 3 修改远程主机ssh配置（/etc/ssh/sshd_config）
    RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile      .ssh/authorized_keys
   
   /usr/sbin/sshd reload
   
   # 4 测试
   ssh root@hostname[ip]
   ```
   
   
   
4. 配置环境变量

   略

### hadoop配置

 1. 修改配置文件

    大部分配置同上，以下只对新增配置进行说明

    **hdfs-site.xml**

    ```xml
    <!-- 副本数可以设置为２，集群有三台机器，副本设置为２可保证一台机器挂掉之后数据不丢失 -->
    <property>
    	<name>dfs.replication</name>
    	<value>2</value>
    </property>
    ```

    **slavers**

    ```
    # 添加集群中每台机器IP或者hostname
    172.18.0.11
    172.18.0.12
    172.18.0.13
    ```

 2. 包分发

    将修改完成的hadoop包分发到每台机器的指定目录

 3. 启动，后台地址：http://172.18.0.11:50070/

    ```shell
    # 1. 分别登录每台机器进行初始化
    ./bin/hdfs namenode -format
    
    # 2. 在主节点执行启动．首次启动时会提示ssh链接另外连台机器的授权提示
    ./sbin/start-dfs.sh
    ./sbin/start-yarn.sh
    
    # 3．确认服务是否启动成功，每台机器执行　"jps" 查看java进程
    # master
    1872 DataNode
    2065 SecondaryNameNode
    2354 NodeManager
    1720 NameNode
    2232 ResourceManager
    
    # slaver
    1024 NodeManager
    897 DataNode
    ```