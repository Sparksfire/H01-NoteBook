为了支持emoji, mysql数据库的编码都改成了 utf8mb4, 由此也引发了mysqlbinlog命令行的错误. 比如执行 mysqlbinlog报错。

```bash
mysqlbinlog: [ERROR] unknown variable 'default-character-set=utf8mb4'
```

解决方法：

1. 配置 my.cnf 中将default-character-set=utf8mb4 修改为 character-set-server = utf8mb4，需要重启数据库。
2. 在mysqlbinlog后增加参数 --no-defaults

!> 注意: 在使用第二种方法的时候，binlog的文件路径要写完整, 否则识别不了。