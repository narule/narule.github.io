---
title: mysql常见操作语句,权限分配，建表，增删改查
author: Narule
date: 2019-11-27 15:00:00 +0800
categories: [Blogging, Technology^技术, SQL^数据库]
tags: [writing, Mysql]
---

## 用户操作

### 新建用户

`grant 权限 on 数据库.表名 to 用户名@'访问地址' identified by "密码";`

新建一个可以远程访问数据库的用户 test， 密码：ps12345 并且只赋予查询权限：

`grant select on *.* to test@'%' identified by "ps12345";`

用户格式： 用户名@'地址' , %表示所有，意思是所有远程主机都可以登录，root默认只能本地登录，可以上面的语句修改

### 赋予用户权限

#### 指定部分授权

```sql
grant insert,update,delete,select on *.* to test@'%';
```

#### 授权

　　所有权限

```sql　
　grant all privileges on *.* to test@'%'; 
```

　　后面加上 identified by "password" 可以修改密码；

　　*.* 表示所有数据库的所有表

　　mysql.* 表示mysql数据库的所有表

　　mysql.user 表示mysql数据库的user表

#### 撤销权限

```sql　　
revoke update on *.* from test@'%';  --(撤销更新数据库的权限)
revoke all on *.* from test@'%';　　--(撤销所有的权限)
```



#### 删除用户

　　`drop user test@'%';`

#### 查看权限

　　`show grants for test@'%';`

 

## 数据操作

### linux环境运行sql 文件

　　如 test.sql 文件在/root 目录下

　　登录mysql之后 

　　`source /root/test.sql; ` #即可导入sql文件内容

### 新建数据库

```sql
creat database database_name default character set utf8 collate utf8_general_ci;

creat database 数据库名 default character set utf8mb4 collate utf8mb4_unicode_ci; 
```

### 新建表

```sql
 creat table 表名 {

　　字段一 类型

　　字段二 类型

　　字段三 类型

　　...

} 
```





### 修改表字段

```sql
ALTER TABLE `domoment`.`leave_message` 
CHANGE COLUMN `public` `public_able` INT(1) NULL DEFAULT '1' COMMENT '是否公开' ;
```



### 增删改查 针对表

#### 　　增 insert

```sql
insert into 表名(字段1，字段2，...)value(值1,值2，...);

insert into 表名(值1，值2，值3)；
```



#### 　　删 delete

```sql
delete from 表名 where 条件 ；--不加条件会把表内所有的数据删除
```



#### 　　改 update

```sql
update 表名 set 字段 = 值 where 条件； --不加条件会更新表内每条数据
```



#### 　　查 select

```sql
select * from 表名 where 条件； --不加条件会查出表内所有数据
```



### 聚合函数
```sql
count(*)--数据总数,
min(字段)--最小值，
max(字段)--最大值 ，
sum(字段)--字段内容求和，
avg(字段)--字段内容求平均值　
```

### 综合性查询，条件查询

#### 　　group by (字段)  通过字段分组

#### 　　having

　　在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

#### 　　like 模糊查询

```sql
select 字段 from 表名 where 字段 like ‘st%’;  --查询字段以 'st' 开头的数据  

select 字段 from 表名 where 字段 like ‘%st%’; --查询字段包含 'st' 的数据　　　　

select 字段 from 表名 where 字段 like ’%st';  --查询字段以st 结尾的数据
```



#### 　　特殊字符处理

　　　java开发中，包含 '%' '_' '\' 特殊字符如何处理：

　　　　需要用到转义字符 \

　　　　%　　->　　\%

　　　　_　　->　　\_

　　　　\　　->　　\\\\

　　　　在java中可以如下处理：

```java
    /**
     * 处理 mysql 模糊查询语句 关键字keywords 包含 % \ _  特殊字符问题
     * @param keywords
     * @return
     */
    private static synchronized String HandleSqlLikeKey(String keywords) {
        keywords = keywords.replaceAll("\\\\", "\\\\\\\\\\\\\\\\");        //  \ -> \\\\
        keywords = keywords.replaceAll("%", "\\\\%");                    //    % -> \%    
        return keywords.replaceAll("_", "\\\\_");                        //    _ -> \_
        
    }
```

这里会觉得反斜杠“\"很多，这是因为 \\\\ 经过Java程序读取一次的时候，转义为 \\ ，到了数据库，再一次读取，第二次转义为 \

数据库某条数据的一个字段为 ’eg%cd‘ 如果想通过模糊查询找到（条件，包含%的字段），

那么在java开始接收参数的时候，查询条件的内容就应该是"\\\\%"

经过两次转义 \\\\%  变成 \% 拼接到sql中的样子就是

```sql
select * from tableName where value like '%\%%'

-- 不转义 sql拼接的结果是
 select * from tableName where value like '%%%' 
--会出差错  因为前两个%%  mysql就读完了 模糊查询
```

 

 

### 6 函数调用，存储过程
```

```