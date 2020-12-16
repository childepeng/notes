# Hadoop部署

Hadoop官网，https://hadoop.apache.org，建议下载版本：2.10.*

```shell
# 自行选择下载速度块的镜像
wget https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz
tar xvf hadoop-2.10.1.tar.gz
```

## 单机部署

1. 设置环境变量，当前用户建议修改".bashrc", 全局安装修改“/etc/profile”

   ```shell
   # 配置Java环境变量
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk
   export PATH=$PATH:$JAVA_HOME/bin
   
   # 配置Hadoop环境变量
   export HADOOP_HOME=/home/hadoop/hadoop-2.10.1
   export PATH=$PATH:$HADOOP/bin:$HADOOP/sbin
   ```

   

2. 修改配置文件，`./etc/hadoop`

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
       <value>hdfs://[hostname]:9000</value>
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
       <value>/home/peng/dev/hadoop/hadoop-2.10.1/namenode</value>
   </property>
   <!-- datanode数据目录 -->
   <property>
       <name>dfs.datanode.name.dir</name>
       <value>/home/peng/hadoop/hadoop-2.10.1/datanode</value>
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

> 172.17.0.11	hdp01
>
> 172.17.0.12	hdp02
>
> 172.17.0.13	hdp03

1. 修改hostname

   ```shell
   sudo hostname hdp01
   sudo echo "hdp01" > /etc/hostname
   ```

2. 修改hosts

   ```sh
   echo "hdp01 172.17.0.10\nhdp02 172.17.0.12\nhdp03 172.17.0.13" >> /etc/hosts
   ```

3. 设置免密登录

   略

### hadoop配置



