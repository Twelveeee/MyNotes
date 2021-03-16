# Oracle

## 创建表空间

```sql
create tablespace waterboss
datafile 'c:\waterboss.dbf'
size 10m
autoextend on
next 10m;
```

## 创建用户

```sql
create user wateruser 
identified by password
default tablespace waterboss;
```

create user 用户名
identified by 密码
default tablespace 默认表空间;

备注：账户wateruser2 ，密码public

## 授权

```sql
grant dba to wateruser;
```

dba = databaseAdmin

## 数据类型

**字符型**
CHAR: 固定长度，2000字节
VARCHAR2: 可变长度，4000字节
LONG: 大文本类型，最大2G

**数值型**
NUMBER
NUMBER(5)最大存5位数
NUMBER(5,2)五位数，其中两个小数

**日期型**
DATE：精确到秒
TIMESTAMP：精确到秒的小数点后9位

**二进制型**（大数据类型）
CLOB：存储字符，最大4G
BLOB：存储图像、声音、视频最大4G

## 表操作

### 创建表

```sql
create table t_owners
(
id number primary key,
name varchar2(30),
addressid number,
housenumber varchar2(30),
watermete varchar2(30),
adddate date,
ownertypeid number
)
```

### 添加字段

```sql
alater table t_owners add
(
	remark varchar2(20),
    outdate date
)
```

### 修改字段

```sql
alater table t_owners modify
(
	remark char2(20),
    outdate timestamp
)
```

修改字段名

```sql
alater table t_owners rename colnumn outdate to exitdate
```

删除字段名

```sql
alater table t_owners drop colnumn outdate
```

删除表

```sql
drop table tablename
```

## 数据操作

数据操作完了，一定要再执行commit

### **插入数据**

```sql
insert into T_OWNERS values(2,'asd',1,'2-3','1234',sysdate,1);
commit;
```

### **修改数据**

```sql
update T_OWNERS set adddate = adddate-3 where id=1;
commit;
```

### **删除数据**

```sql
delete from T_OWNERS where id=2;
commit;
```

```SQL
truncate table tablename;
```

delete删除的数据可以rollback
delete删除可能产生碎片，并且不释放空间
truncate是先摧毁表结构，再重构表结构
truncate不用commit直接执行，不能rollback
truncate清除表的所有数据，释放存储空间

## jdbc 链接数据库

1，创建工程，引入驱动包
创建java工程，建立lib文件夹，将ojdbc.jar拷贝到此文件夹，然后add build path

2.写BaseDao
写一个BaseDao负责加载驱动，获取数据库连接，关闭资源

oracle的jdbc连接方式:oci和thin
thin是一种瘦客户端的连接方式，即采用这种连接方式不需要安装oracle客户端,只要求classpath中包含jdbc驱动的jar包就行.thin就是纯粹用Java写的ORACLE数据库访问接口。
oci是一种胖客户端的连接方式，即采用这种连接方式需要安装oracle客户端。oci是Oracle Call Interface的首字母缩写，是ORACLE公司提供了访问接口，就是使用Java来调用本机的Oracle客户端，然后再访问数据库，优点是速度快，但是需要安装和配置数据库。