# Hbase 部署

hbase部署官网： [https://hbase.apache.org/](https://hbase.apache.org/)

## 单机部署

1. 下载软件包，建议下载 2.2.6

   ```shell
   # 如果下载速度慢，可以官方找其他镜像的下载地址
   wget https://mirror.bit.edu.cn/apache/hbase/2.2.6/hbase-2.2.6-bin.tar.gz
   tar xvf hbase-2.2.6-bin.tar.gz
   ```

2. 设置环境变量，

   ```shell
   # 根据实际路基配置，如果只需要对当前用户有效，修改用户目录下".bashrc"文件；如果全局配置，修改 "/etc/profile"
   # 设置Java环境变量
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export PATH=$PATH:$JAVA_HOME/bin
   # 设置Hbase环境变量
   export HBASE_HOME=/home/aisys/hbase/hbase-2.2.6
   export PATH=$PATH:$HBASE_HOME/bin
   ```

3. 修改配置文件

   **hbase-env.sh** 

   ```shell
   # 1. 修改 JAVA_HOME
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
   
   # 2. 设置是否使用及维护自带zookeeper
   export HBASE_MANAGES_ZK=true
   # false 表示hbase需要连接独立部署或者其他服务提供的zk
   # true 表示使用内置的zk
   ```

   **hbase-site.xml**

   ```xml
     <property>
       <name>hbase.unsafe.stream.capability.enforce</name>
       <value>false</value>
     </property>
   
     <!-- 临时文件目录 -->
     <property>
   	<name>hbase.tmp.dir</name>
       <value>./tmp</value>
     </property>
   
     <!-- zookeeper地址，如果hbase-env[HBASE_MANAGES_ZK]配置了false，这个配置可忽略，多地址使用","分隔 -->
     <property>
       <name>hbase.zookeeper.quorum</name>
       <value>192.168.1.6:2181</value>
     </property>
   
     <!-- hbase数据目录，此处可设置 hadoop路径 -->
     <property>
        <name>hbase.rootdir</name>
        <value>file:///home/aisys/hbase/data-2.2.6</value>
     </property>
   
     <!-- hbase集群模式，false表示单机模式，true表示集群模式 -->
     <property>
        <name>hbase.cluster.distributed</name>
        <value>false</value>
     </property>
   
   ```

4. 启动， 后台地址： http://localhost:16010

   ```shell
   # bin目录
   ./start-hbase.sh
   ```

## 集群部署



