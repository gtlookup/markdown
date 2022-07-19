# 1，安装 P30

- 下载地址：https://dev.mysql.com/downloads/mysql/

- 添加环境变量：复制 bin 路径，添加到环境变量 path 中

- root下新建my.ini配置文件

  ```bash
  [mysqld]
  basedir=D:\Program Files\mysql-5.7.29\
  datadir=D:\Program Files\mysql-5.7.29\data\
  port=3306
  #跳过密码认证
  skip-grant-tables #8+版本得注释掉 否则启动后自动关闭
  ```

- ==管理员模式==进cmd运行命令

- 安装到系统服务：**mysqld -install**

  - **删除：mysqld -remove**

- 手动新建文件夹 D:\Program Files\mysql-5.7.29\data\

- 初始化数据库文件：**mysqld --initialize-insecure --user=mysql**

  - db信息存放位置：mysql/data/库名/

  - ==centos 安装这步报错==

    >  error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or 

    ```bash
    yum install libaio
    yum install numactl.x86_64
    ```

- 启停mysql：**net start/stop mysql**==（centos：service mysqld start/stop）==

- 命令行进mysql：**mysql -uroot -p**

- 改密码：

  ```mysql
  update mysql.user set authentication_string=password('123456') where user='root' and host='localhost';
  ```

- 将 my.ini 的 skip-grant-tables 注释掉

- 重启 mysql 



# 2，拷贝表结构

```mysql
CREATE TABLE 新表名 like 旧表名;
```



# 3，填坑

## 3.1 only_full_group_by

> [Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column list which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

```mysql
select @@sql_mode
# 则在 my.ini 里加上
sql_mode=#@@sql_mode出来的内容删除 only_full_group_by 写在这里
#sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
# 重启 mysql
```

## 3.2 开启远程访问权限

> 当发生 'xxxxxx' is not allowed to connect to this MySQL server 错误时

```bash
# 1.新建一个供远程访问的用户
insert into user(host,user) values('%','admin');
create user '用户名'@'%' identified by '密码'; # mysql5.7创建用户
flush privileges; # 刷新到内存
# 2.开放admin用户可访问的库（空的就给空字符串）
grant all privileges on `要开放的库名`.* to 'admin'@'%' identified by '密码';
# 2.1 mysql8以上版本
# grant all privileges on *.* to 'root'@'%' with grant option;
flush privileges; # 刷新到内存
```

## 3.3 锁表及解决

```mysql
-- 查看被锁的表
-- 有 Waiting for table metadata lock 代表锁表
show processlist;
-- 杀掉锁表进程
kill id;
```

```mysql
-- 如果执行 delete from 发现 show processlist 的状态一直是 updating
-- 说明有事务没有结束
-- 通过该语句的 trx_mysql_thread_id 与 show processlist 的 id 对比，有相等的就 kill掉
select * from information_schema.innodb_trx;
```

> 2059 - authentication plugin 'caching_sha2_password

```mysql
# 修改密码模式
# 123 为密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123';
```

## 3.4 (> 8126) 错误

> Row size too large (> 8126). Changing some columns to TEXT or BLOB may help. In current row format, 

```bash
# my.ini 或 my.cnf
innodb_log_file_size=512M
innodb_strict_mode=0   # mysql8版本的话下面不用配，只配这两句

#innodb_file_per_table=1         # 默认可不写
#innodb_file_format=Barracuda    # 默认可不写
#innodb_file_format_check = ON   # 默认可不写
# mysql> SHOW GLOBAL VARIABLES LIKE '%innodb_file%';
# 结果如下：所以上面3项目不用加
+--------------------------+-----------+
| Variable_name            | Value     |
+--------------------------+-----------+
| innodb_file_format       | Barracuda |
| innodb_file_format_check | ON        |
| innodb_file_format_max   | Barracuda |
| innodb_file_per_table    | ON        |
+--------------------------+-----------+
4 rows in set (0.00 sec)

# 设置完保存后重启mysql
```

## 3.4 Error 1040

> ```ini
> Error 1040: Too many connections
> ```

```bash
show variables like 'max_connections'; # 发现默认是151
set global max_connections = 5000;     # 设置大一些就OK了
```



# 4，命令

```bash
SELECT @@version # 查看版本
SELECT @@basedir # 查看安装目录
SELECT @@datadir # 查看数据目录
show databases   # 查看所有库
show tables      # 查看所有表
describe tbname  # 查看表结构

#登陆 -u/-p后没空格
mysql -u用户 -p密码
#本地cmd远程登陆
mysql -u用户名 -p -h 远程ip地址 -P 端口 -D 数据库名
#导出db
mysqldump -u root -p 数据库 > 路径及文件.sql
#远程导出db
mysqldump -u 用户名 -p -h 远程ip地址 -P 端口 数据库名 > 路径及文件.sql
#导入db
mysql -u root -p 数据库名 < 路径及文件.sql
#查看用户名密码
use mysql;
select host,user,password from USER
#退出连接
exit;
```

# 5，mysql 四种语言

- DDL：定义语言==不支持事务（如 create table）==

- DML：操作语言，增删改

- DQL：查询语言，select

- DCL：控制语言

# 6，导入.sql文件

```bash
# 1. 管理员身份打开cmd
# 2. 登陆 mysql 并设置utf-8
mysql -uroot -p --default-character-set=utf8
#    输入密码
Enter password: ***
# 3. 导入
mysql> source d:/***.sql;
```

# 7，函数

## 7.1 类似 indexOf 功能

```mysql
# d 不在 abc 里，返回 0
# a 在 abc 里，返回 1
SELECT LOCATE('d','abc')
```

## 7.2 字符串连接

```mysql
SELECT CONCAT('a','b') # 结果：ab
# 有一个为null，则整个字符串都为null
SELECT CONCAT('a','b', null) # 结果：null
# arg1:连接后以什么字符分隔
#      null的不连接到字符串里
CONCAT_WS(',','a','b', null) # 结果：a,b
```

## 7.3 日期format

```mysql
DATE_FORMAT(NOW(),'%Y/%m/%d %H:%i:%S')
```

## 7.4 只取年月日

```mysql
date(now()) # 2020-09-27
```

## 7.5 uuid

```mysql
SELECT replace(uuid(),"-","")
```

## 7.6 IFNULL

```mysql
IFNULL(PARENT_ID,'') # 如果 PARENT_ID 是 null 则返回 ''，否则返回 PARENT_ID
```

## 7.6 字符转数字

```mysql
SELECT ID FROM m_mail_template order by (ID + 0) # 字符+0转成数字
```

## 7.7 GROUP_CONCAT



# 8，json

## 8.1 查询

```mysql
set @v = '{"a":1,"b":2,"c":"ccc","d":[1,2,3],"e":{"e1":"111","e2":"222"},"a1":"ccc"}';
# 单值列查询
SELECT JSON_EXTRACT(@v, '$.*') col           # 结果：[1, 2, "ccc", [1, 2, 3], {"e1": "111", "e2": "222"}]
SELECT JSON_EXTRACT(@v, '$.e') col           # 结果：{"e1": "111", "e2": "222"}
SELECT JSON_EXTRACT(@v, '$.d') col           # 结果：[1, 2, 3]
# 数组列查询
SELECT JSON_EXTRACT(@v, '$.d[*]') col        # 结果：[1, 2, 3]
SELECT JSON_EXTRACT(@v, '$.d[1 to 2]') col   # 结果：[2, 3]
SELECT JSON_EXTRACT(@v, '$.d[last ]') col    # 结果：3
SELECT JSON_EXTRACT(@v, '$.d[last - 1]') col # 结果：2
# 根据值查询key
# one：只返回第一个满足的key
SELECT JSON_SEARCH(@v, 'one', 'ccc') col     # 结果："$.c"
# all：返回所有满足的key
SELECT JSON_SEARCH(@v, 'all', 'ccc') col     # 结果：["$.c", "$.a1"]
```

```mysql
# json_test 表结构及数据
CREATE TABLE `json_test`  (
  `id` int(0) NOT NULL AUTO_INCREMENT,
  `json_context` json NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of json_test
-- ----------------------------
INSERT INTO `json_test` VALUES (1, '{\"params\": {}, \"default\": false, \"createBy\": \"13900000000\", \"dictType\": \"company_type\", \"parentId\": \"10680\", \"dictLabel\": \"中性人\"}');
INSERT INTO `json_test` VALUES (2, '{\"params\": {}, \"default\": false, \"createBy\": \"13900000000\", \"dictType\": \"company_type\", \"parentId\": \"10680\", \"dictLabel\": \"啊\"}');
INSERT INTO `json_test` VALUES (3, '{\"admin\": false, \"deptId\": \"\", \"params\": {}, \"userId\": 183892, \"deptName\": \"\", \"nickName\": \"专班用户1\", \"updateBy\": \"13900000000\", \"userName\": \"15610271018\", \"isSpecial\": \"0\", \"cityDeptId\": \"370100\", \"adminDeptId\": \"100\", \"phonenumber\": \"15610271018\", \"userSysType\": \"2\", \"districtDeptId\": \"370102\"}');
INSERT INTO `json_test` VALUES (4, '{\"msg\": \"操作成功\", \"code\": 200}');
INSERT INTO `json_test` VALUES (5, '{\"params\": {}, \"default\": false, \"createBy\": \"13900000000\", \"dictType\": \"company_type\", \"parentId\": \"10680\", \"dictLabel\": \"生产厂\"}');
INSERT INTO `json_test` VALUES (6, '{\"dictCode\": 10669}');

########################################### 查询 #########################################
SELECT id, json_context->'$.code' createBy 
FROM json_test WHERE json_context->'$.code'=200                                 # 结果：{"msg": "操作成功", "code": 200} 200

SELECT json_context->'$.createBy' createBy FROM json_test limit 1               # 结果："13900000000"
# JSON_UNQUOTE：取出来的值不带引号
SELECT JSON_UNQUOTE(json_context->'$.createBy') createBy FROM json_test limit 1 # 结果：13900000000
# ->>：取出来的值不带引号
SELECT json_context->>'$.createBy' createBy FROM json_test limit 1              # 结果：13900000000
# member of：查询 json_context.createBy = '13900000000' 的记录
SELECT * FROM json_test WHERE '13900000000' member of (json_context->'$.createBy')
# JSON_CONTAINS + JSON_OBJECT 实现 member of 相同查询
SELECT * FROM json_test WHERE JSON_CONTAINS(json_context, JSON_OBJECT('createBy', '13900000000'))
# JSON_CONTAINS_PATH：查询json里是否包含某几个key
# one：一行里只要包含一个属性就可以
SELECT * FROM json_test WHERE JSON_CONTAINS_PATH(json_context, 'one', '$.createBy', '$.code') # 结果：4行
# all：一行里全都包含才可以
SELECT * FROM json_test WHERE JSON_CONTAINS_PATH(json_context, 'all', '$.createBy', '$.code') # 结果：0行
```

## 8.2 修改

https://www.bilibili.com/video/BV1LD4y1m7Ej?from=search&seid=8112749730003389644  16:52

# 9，死锁

由唯一索引引起的死锁问题，归根到底，是由于并发请求造成的，无非是==由于前端重复请求或者是网络抖动产生的==

死锁一般是事务相互等待对方资源，最后形成环路造成的。对于死锁，数据库处理方法：牺牲一个连接，保证另外一个连接成功执行。

发现死锁后，InnoDB引擎会马上回滚一个事务，会返回1213错误

**mysql 锁分类：**

1. 全局锁（数据库级别，做逻辑备份使用）

2. 表级锁（意向共享锁（IS）、意向排他锁（IX））

3. 行锁

   - 共享锁（S)

   - 排他锁（X）

   - 间隙锁（GAP）

     > 当有id=1 和 7 两条数据，此时在事务A里 select where id=2（因为==没有2==，且2在1和7之间，所以1到7范围会被`间隙锁`）
     >
     > 然后在事务B里insert id=3，就会被阻塞，直到事务A提交或回滚后才能释放`间隙锁`
     >
     > 看上去1、2、3、7没什么关系不会行锁，但因为`间隙锁`还是会阻塞

     - https://www.jianshu.com/p/32904ee07e56

   - 间隙锁加行锁（next-lock-key）

mysql 锁：https://mp.weixin.qq.com/s?__biz=MzA5Mjg2MDQ5NQ%3D%3D&idx=1&mid=2452509174&scene=21&sn=600a3d3622927dd8234d6fd66f603665#wechat_redirect

**mysql 事务隔离级别**：

- 级别1（read uncommitted）：所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）
- 级别2（read committed）：读取已经提交的数据（可以读取到其他事务提交的update更新和insert新增），可以解决脏读
- 级别3（repeatable read）：保证同一事务的多个实例在并发读取数据时，会看到同样的数据行（==默认==）
- 级别4（serializable）：这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争

## 9.1 命令

```mysql
show engine innodb status; # 查看死锁日志
SELECT @@tx_isolation;     # 查看 mysql 事务的隔离级别
mysql -u root -p --execute="show engine innodb status \G" > D:/a.log # 导出死锁日志
```

## 9.2 死锁案例

### 9.2.1 行死锁

> 先来个简单例子，假如有个表 shop_order，id 是自增主键（==行锁锁的不是记录，而是索引==）

```mysql
# 1. 打开控制台1
mysql> begin; # 开启事务
Query OK, 0 rows affected (0.00 sec)
mysql> select * from shop_order where id=1 for update; # 加排它锁（X）
+----+---------+-----------+-------+--------+--------+
| id | user_id | user_name | name  | price  | number |
+----+---------+-----------+-------+--------+--------+
|  1 |       1 | admin     | 订单1 | 199.00 |      1 |
+----+---------+-----------+-------+--------+--------+
1 row in set (0.02 sec)
```

```mysql
# 2. 打开控制台2
mysql> begin; # 开启事务
Query OK, 0 rows affected (0.00 sec)
mysql> select * from shop_order where id=1 lock in share mode; # 加共享锁（S）。回车后在这阻塞了，直到控制台1 commit; 后才能往下走
```

https://www.bilibili.com/video/BV1KK4y1Q72f?from=search&seid=5348616453770224185

> 这是java里的行锁例子

```java
@GetMapping("a")
@Transactional
public String a() throws InterruptedException {
    new DDuplicate().setId(1).setName1("a").updateById();
    TimeUnit.SECONDS.sleep(8);
    new DDuplicate().setId(2).setName1("a").updateById();
    TimeUnit.SECONDS.sleep(8);
    return "a";
}

@GetMapping("b")
@Transactional
public String b() throws InterruptedException {
    new DDuplicate().setId(2).setName2("b").updateById();
    TimeUnit.SECONDS.sleep(8);
    new DDuplicate().setId(1).setName2("b").updateById();
    TimeUnit.SECONDS.sleep(8);
    return "b";
}
// 先访问a，再访问b
```

### 9.2.2 间隙死锁

假设表里有id=1和7两条数据

```mysql
# 案例一
begin; # 事务A
select * from xx where id=2 for update; # A。A获取1-7的间隙锁
begin; # 事务B
select * from xx where id=3 for update; # B。B获取1-7的间隙锁
insert into xx values(4,4,4)            # B
insert into xx values(5,5,5)            # A。此时发生死锁
```

```mysql
# 案例二
begin; # 事务A
update xx set a=2 where id=2;           # A。A获取1-7的间隙锁
begin; # 事务B
insert into xx values(3,3,3)            # B。此时阻塞
begin; # 事务C
insert into xx values(4,4,4)            # C。此时阻塞
insert into xx values(5,5,5)            # A。此时发生死锁
```

## 9.3 日志分析

```bash
=====================================
2021-06-25 06:08:20 2abed74c1700 INNODB MONITOR OUTPUT  # output时间
=====================================

------------------------
LATEST DETECTED DEADLOCK # 这段后面记录了死锁信息
------------------------
2021-06-25 10:32:36 2abe0163b700 # 最后一次死锁时间

*** (1) TRANSACTION: # 事务1信息
#           是此事务的id       活跃时间0秒
TRANSACTION 13443047642, ACTIVE 0 sec inserting
mysql tables in use 1, locked 1 # 表示此事务修改了一个表，锁了一行数据
LOCK WAIT 45 lock struct(s), heap size 376, 76 row lock(s), undo log entries 48
#        线程id                                                 查询id     数据库ip地址，账号，更新语句
MySQL thread id 560, OS thread handle 0x2abed806f700, query id 395306104 10.0.0.17 root update
INSERT INTO new_track_event2code...  # 正在执行的sql

*** (1) WAITING FOR THIS LOCK TO BE GRANTED: # 事务1正在等待的锁
RECORD LOCKS space id 5236 page no 9 n bits 35 index `idx_trackec_u` of table `cargo_copy`.`new_track_event2code` trx id 13443047642 lock_mode X waiting # X：排他锁
Record lock, heap no 35 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len=3; bufptr=0x2ababedd4501; hex= 4d534b; asc MSK;;
 1: len=0; bufptr=0x2ababedd4504; hex= ; asc ;;
 2: len=8; bufptr=0x2ababedd4504; hex= 47415445204f5554; asc GATE OUT;;
 3: len=4; bufptr=0x2ababedd450c; hex= 8000021c; asc     ;;

*** (2) HOLDS THE LOCK(S): # 事务2持有的锁
# 持有的是行级锁                                      锁的哪个索引                  锁的哪个库哪个表
RECORD LOCKS space id 5236 page no 9 n bits 35 index `idx_trackec_u` of table `cargo_copy`.`new_track_event2code` 
#       事务id       锁模式：（X：排他锁，S：共享锁）
trx id 13443047646 lock_mode X
Record lock, heap no 35 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
# 以下，0开头表示id列，1~3表示第1~第3个字段
 0: len=3; bufptr=0x2ababedd4501; hex= 4d534b; asc MSK;;
 1: len=0; bufptr=0x2ababedd4504; hex= ; asc ;;
 2: len=8; bufptr=0x2ababedd4504; hex= 47415445204f5554; asc GATE OUT;;
 3: len=4; bufptr=0x2ababedd450c; hex= 8000021c; asc     ;;
 
 *** WE ROLL BACK TRANSACTION (2) # 表示回滚了事务2
```

> **lock_mode X locks rec but not gap waiting Record lock** 非间隙死锁分析：https://www.jianshu.com/p/7655dc7884dc
>
> ==事务在以非主键索引为where条件进行Update的时候，会先对该非主键索引加锁，然后再查询该非主键索引对应的主键索引都有哪些，再对这些主键索引进行加锁==

## 9.4 尽量避免

1）对修改同一表的处理中，按统一的顺序来（更新或删除，如 order id）

2）修改或删除时，要保证 where 条件能命中数据，这样防止间隙锁的发生

# 附录

## 添加序号列

```mysql
-- 单表
SELECT (@i:=@i+1) i, T1.name FROM target_template T1,(SELECT @i:=0) T2
```

```mysql
-- 多表 join
SELECT T1.i,T1.id,T2.NAME
FROM (SELECT (@i:=@i+1) i,T.* FROM target_import T,(select @i:=0) TT) T1
LEFT JOIN target_template T2 ON T1.template_id = T2.id
WHERE 1 = 1
```

```mysql
-- mysql 里的变量不用事先声明，直接 @+变量名 用就行
```

## 给后添加的ID自增列赋值

```mysql
UPDATE d_oa_mjh T
INNER JOIN (
	SELECT (@i:=@i+1) v, T1.Id FROM d_oa_mjh T1,(SELECT @i:=0) T2
) TT ON T.Id = TT.Id
SET T.i = TT.v
```

## select now() 差8小时

```bash
# my.cnf
default-time-zone='+08:00'
```

## where 条件不区分大小写

- 方法一：加 ==binary==

```mysql
where binary xx_type = 'y'
```

- 方法二：把字段改成==utf8_bin==

![image-20201028165548330](mysql.assets/image-20201028165548330.png)

## 查看列名

```mysql
SELECT
	column_name,    # 列名
	column_comment, # 列说明
	TABLE_NAME      # 表名
FROM
	information_schema.`COLUMNS` 
WHERE
	TABLE_SCHEMA = 'nfh_sfa'  # db名
	AND TABLE_NAME = 'd_file'
	AND column_name = 'RECORD_ID'
```

## 查看表行数

```mysql
select NUM_ROWS from information_schema.INNODB_TABLESTATS where name = '库名/表名';
```

## 查指定库全部索引

```mysql
SELECT a.TABLE_SCHEMA,a.TABLE_NAME,a.index_name,GROUP_CONCAT(column_name ORDER BY seq_in_index) AS `Columns`
FROM information_schema.statistics a
WHERE a.TABLE_SCHEMA='sdtrace' # 哪个库的全部索引
GROUP BY a.TABLE_SCHEMA,a.TABLE_NAME,a.index_name
```

## 没有insert否则update

```mysql
# 方法一：ON DUPLICATE KEY UPDATE（当 id=1 在表里有时，就update）
insert into tb(id, name) values(1, 'a') ON DUPLICATE KEY UPDATE id=values(id),name=values(name)
# 方法二：replace into
replace into tb(id,name) values(1, 'a')

# 两者不同点在于，replace 会先删除原数据再 insert
# 比如表tb(id,a,b),有一条id=1、a=1、b=1的数据
insert into tb(id, a) values(1, 1) ON DUPLICATE KEY UPDATE id=values(id),a=values(a); # 执行后依然是：id=1、a=1、b=1
replace into tb(id, a) values(1, 1); # 执行后结果为：id=1、a=1、b=null
```

## insert ignore into

有数据不插入，没有就插

## FORCE INDEX

```mysql
SELECT * FROM TABLE1 FORCE INDEX (FIELD1) # 强制索引
```

## IGNORE INDEX

```mysql
SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2) # 忽略索引
```

## 误删/var/lib/mysql后无法启动

```bash
mkdir /var/lib/mysql             # 1
chown mysql:mysql /var/lib/mysql # 2
service mysqld start             # 3. 启动（mysql8 好用）
```

## 卸载mysql

```mysql
rpm -qa |grep mysql # 1. 查看都要卸载哪些文件
yum remove mysql*   # 2. 卸载
```

