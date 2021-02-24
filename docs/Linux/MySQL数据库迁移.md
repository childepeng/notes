# MySQL数据库迁移方案

## mysqldump

`mysqldump`可以将数据库导出为SQL脚本，然后通过`source` 命令进行导入；适合数据量不大的数据库迁移（数据库超过百万导入数据会很慢）

```bash
# 整库备份
mysqldump -uroot -p --opt -R --all-database > mysql_bak.sql

# 备份指定数据库
mysqldump -uroot -p --opt -R --databases [database1 database2] > mysql_bak.sql

# 还原数据库
mysql -uroot -p -e "source mysql_dak.sql;"
```

## 数据库文件迁移

通过Copy数据库库文件进行整库迁移，适合数据量大的数据库；

linux 环境MySQL默认数据库目录：/var/lib/mysql；需要迁移的文件包括：用户数据库文件夹、ibdata1；

文件迁移到指定服务器之后，放到MySQL数据库目录；然后重启MySQL；

需要注意文件copy之后的文件权限，需要将文件授权给mysql用户；