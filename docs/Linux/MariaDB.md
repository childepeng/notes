# yum安装MariaDB
1. 创建MariaDB.repo文件
```
vi /etc/yum.repos.d/MariaDB.repo
```
2. 设置数据源，可以从MariaDB官网上查询：https://downloads.mariadb.org/mariadb/repositories, 根据系统选择合适的版本
```properties
# MariaDB 10.2 CentOS repository list - created 2017-09-28 02:46 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
3. 执行安装命令
```bash
yum -y install MariaDB-server MariaDB-client
```

# 数据库配置
1. 启动数据库
```bash
# 查看mysql状态;关闭数据库  
# service mysql status  
# service mysql stop  
# 启动数据库  
service mysql start
```
2. 设置数据库密码，初次安装后默认没有密码
```bash
mysqladmin -u root password 'root'
```
3. 或者进入数据库
```bash
# 默认没有密码，直接回车
mysql -uroot -p
set password = password('root');
```

