# PostgreSQL 脚本收集

date: 2011-12-19 19:19:50

> Backup from <http://rongjih.blog.163.com/blog/static/33574461201110300454392>

管理端登录：`> psql postgres`

## 1. 数据库的命令行管理脚本

### 1.1. 创建数据库

`=> CREATE DATABASE 数据库名 WITH ENCODING '数据库编码'  OWNER 拥有者 TEMPLATE 模板名 TABLESPACE 表空间名;`

其中 WITH ENCODING、OWNER、TEMPLATE 和 TABLESPACE 都是可选参数，不设置就使用数据库的默认值。
如 `=> CREATE DATABASE testdb WITH ENCODING 'utf8'  OWNER test;`

另外默认情况下是根据模板 template1 创建的数据库（表空间为 pg_default），如果要创建一个真正空白的数据库，可以改用模板 template0，因为template1 在数据库安装后是允许维护修改的，而 template0 则在数据库安装初始化后一直保持不变的。特别是在使用 pg_dump 命令备份的数据库进行恢复时，使用模板 template0 创建一个空白的数据库，然后再恢复备份信息到这个数据库，就可以避免不必要的对象冲突。

查看已经创建的数据库的列表信息：`=> \l` 或 `=> select datname from pg_database;`

### 1.2. 删除数据库

`=> DROP DATABASE 数据库名;`

如果当前数据库存在会话，会提示你而无法删除。

### 1.3. 用户权限管理

PostgreSQL 的角色包括我们常说的数据库用户或者用户组。

创建角色：`=> create role 角色名;`  
删除角色：`=> drop role 角色名;` <--如果角色是某个数据库的属主，删除时会有提示而无法删除  
查看已存在的角色：`=> \du` 或 `select rolname from pg_roles;`  
创建用户：`=> create user 用户名;` 或 `create role 用户名 login;`  

注：

- 只有创建的角色包含 login 属性，才能使用该角色连接数据库。
- 如要指定用户的登录密码，使用 `=> create role 用户名 login password '登录密码';`
- 如要修改用户的密码，使用 `=> alter role 用户名 with password '登录密码';`

创建组(常用于某些用户的权限管理)：`=> CREATE ROLE group_role NOINHERIT;`  
向组中添加用户：`=> GRANT group_role TO role1, ...;`  
删除组中的用户：`=> REVOKE group_role FROM role1, ...;`  
将表的select权限分配给用户：`=> GRANT SELECT ON [表名] TO [用户名];`  
将表的select权限从用户中剔除：`=> REVOKE  SELECT ON [表名] FROM[用户名];`  
将数据库的select权限分配给用户：`=> GRANT SELECT ON  ALL TABLES IN SCHEMA public TO [用户名];`  
将数据库的select权限从用户中剔除：`=> REVOKE  SELECT ON  ALL TABLES IN SCHEMA public FROM [用户名];`  

有以下几种权限：SELECT（读）， INSERT（追加）， UPDATE（写），DELETE， RULE（规则），REFERENCES(外键)， TRIGGER， CREATE，TEMPORARY，EXECUTE， USAGE，和 ALL PRIVILEGES。

**alter default privileges 的使用：**

- a) 授予默认权限：`alter default privileges for user 【用户1】 in schema public grant select on tables to  【用户2】;`
    > 使用这个语句后， 【用户1】 再建新表时， 【用户2】 就会自动有对这些新表的select权限。  
    > 注意执行这个语句后，在这个语句之前 【用户1】 的老表， 【用户2】 仍然是没有select权限的。  
    > 这时需要手工把旧表的 select 权赋给 scott 用户，然后再使用 alter default privileges 就实现了只读用户 scott。
- b) 收回授予的默认权限：`alter default privileges for user 【用户1】 in schema public revoke select on tables from 【用户2】;`
    > 执行这个语句之前，`alter default privileges .... grant` 语句之后，【用户1】 建的表，【用户2】 已经有 select 权限的，这个语句不会收回这些 select 权限。
- c) 查看 `alter default privileges` 权限的方法是使用 `# /ddp;`

**总结：** `alter default privileges` 并不会改变已有用户的权限，只有在有对象创建时，才会根据 `alter default privileges` 定义的情况给指定的用户加权限，这个语句有点象是在建对象时加了一个权限触发器。


### 1.4. 关于表空间

数据库默认存在 pg_global、pg_default 两个表空间，pg_default 是 template0 和 template1 使用的也是新建数据库的默认表空间。

创建一个新的表空间：`=> CREATE TABLESPACE fastspace LOCATION '/mnt/sda1/postgresql/data';`

注：所指定的表空间路径必须存在、为空及为 PostgreSQL 系统帐号的拥有者。

查看现有的表空间信息：`=> \db` 或 `=> select spcname from pg_tablespace;`  
设置默认的表空间：`=> SET default_tablespace = space1;`  
将表创建到指定的表空间：`=> CREATE TABLE foo(i int) TABLESPACE space1;`

### 1.5. 其它

- 查看数据库的版本信息：`=> select version();`  
- 查看数据库的命令帮助：`=> \h 或 \?`  
    > `\h` 查看的是 SQL 命令，如 `create table` 等；`\?` 查看的是 psql 的元数据命令，如 `\db` 等。

### 1.6. 数据库的备份与恢复：

1. 备份数据库为 sql 文件：`> pg_dump -U bcsystem > bcsystem.sql` <-- 用户 bcsystem 是数据库 bcsystem 的属主
2. 恢复备份的sql文件：
    - `> psql -U bcsystem -f bcsystem.sql` <-- 用户 bcsystem 是数据库 bcsystem 的属主
    - `> psql -U bcsystem < bcsystem.sql` <-- 用户 bcsystem 是数据库 bcsystem 的属主
    - `> psql -h192.168.0.222 -U bcsystem -W -f bcsystem.sql` <-- 用户bcsystem是数据库bcsystem的属主
    - `=> \i 文件名.sql` <-- 如果要输出详细的执行信息，`=> set echo to all;`
3. 备份指定的表为压缩文件：`> pg_dump -dbcsystem -Ubcsystem -Fc -npublic -tmyTable -v -c -x --no-tablespaces > myTable.dump`  
    > -t 后的为要备份的表名，可使用通配符，也可使用多个 -t 参数指定多个表
4. 恢复备份的压缩文件：`> pg_restore -dbcsystem -Ubcsystem -npublic -tmyTable -v -a < myTable.dump`
    > -a 参数表示只恢复数据，不填此参数则会连表的结构一齐重建
5. [Postgres 数据表部分数据的备份与恢复](./2014-11-17-Postgres数据表部分数据的备份与恢复)

### 1.7 数据表维护脚本

- 添加列：`ALTER TABLE bs_case_advice ADD COLUMN is_invalid boolean DEFAULT false;`
- 删除列：`ALTER TABLE bs_case_advice DROP COLUMN is_invalid;`
- 修改列名：`ALTER TABLE bs_case_advice RENAME is_invalid  TO is_invalid1;`
- 修改列类型：`ALTER TABLE bs_case_advice ALTER COLUMN is_invalid TYPE boolean;`
- 修改列默认值：`ALTER TABLE bs_case_advice ALTER COLUMN is_invalid SET DEFAULT false;`
- 创建外键关联：`ALTER TABLE t1 ADD CONSTRAINT fk_t1_c1 FOREIGN KEY (c1) REFERENCES t2 (ID) ON UPDATE RESTRICT ON DELETE RESTRICT;`
- 创建索引：`CREATE [ UNIQUE ] INDEX idx_t1_c1 ON t1 (c1);`

## 2. SQL 注意事项

- 使用 `select distinct ... order by aa` 时，`order by` 后的字段必须在 select 的字段中包含，否则报错
- 查询语句中使用别名时，统一使用 as 关键字可以避免不必要的错误
    > 如 `t.name name`、`t.type type` 会报错，改为 `t.name as name`、`t.type as type` 就不会报错了
- 使用 hibernate 时，如果 domain 字段的类型为 boolean，数据库字段也必须用 boolean，不能用 integer 代替，如
 `'INNER_ boolean not null default false'`
- 官方文档中有说使用 integer 字段类型，速度是最快的，但 integer 类型不支持限定长度，如 integer(2) 的语法是错误的，如果要控制长度则只能使用 numeric(2) 来代替

## 3. 其它

- 启动：`$ sudo /etc/init.d/postgresql-9.1 start`
- 停止：`$ sudo /etc/init.d/postgresql-9.1 stop`
- 设置客户端编码：`=> SET CLIENT_ENCODING TO 'utf8';` 或 `=> \encoding 'utf8';`
- 查看客户端编码：`=> \encoding`
- 替换 bytea 字段内的部分值：`update act_ge_bytearray set bytes_ = overlay(bytes_ placing E'2013-04-25'::bytea from position(E'2013-03-25'::bytea in bytes_))
where id_ in ('398416','398420');`
- 替换字符串：`replace(string text, from text, to text)` - `replace('abcdefabcdef', 'cd', 'XX') = abXXefabXXef`
- 转换数据类型：`select cast(id as varchar(255)),... from ...` (id 原为 int 类型，这里将其转换为 varchar 类型)
- 数字格式化：小数格式化用 `to_char(16.50,'FM999999990D99')` -> 16.5, 整数格式化用 `to_char(16,'FM999999990')` -> 16
- 构建 1-12 的连续数据：`select * from generate_series(1,12) order by 1` 或 使用递归：
    ```
    WITH RECURSIVE T(N) AS ( 
      VALUES (1) 
      UNION ALL 
      SELECT N+1 FROM T WHERE N < 12
    ) SELECT N FROM T;
    ```
- 数组转为行：`SELECT UNNEST(ARRAY['a','b','c','d']) AS names`
- 行转为数组：`select array_agg(id) from T`
- 支持用 `$$` 将字符串括起来，如 `SELECT $$It's good$$ AS greeting;` 这样写， 对动态生成 SQL 语句很有用， 因为:
    1. 当嵌入字符串时, 能避免既繁琐, 又容易出错的手工引用和对特殊字符的转义.  
    2. 由于文本编辑器和 IDE 一般不把 `$$` 当作字符串分隔符, 动态生成的 SQL 语句依然根据语法高亮显示。
- 触发器函数里的几个特殊变量：<http://www.php100.com/manual/PostgreSQL8/plpgsql-trigger.html>

## SQL 函数参考

1. 获取行号: `SELECT row_number() over(),* FROM t;`
2. 返回第一个非空值：`select COALESCE(value [, ...])`
3. 查看sql的执行计划：`explain sql;`
4. 查看sql的实际执行计划：`explain  analyze sql;` 或 `explain (analyze) sql; `
5. 行列转换：crosstab、crosstabN
    > 9.0+ 版本默认已安装 tablefunc，在 `$PGHOME/share/extension/` 路径下会有 `tablefunc*` 的三个文件，分别是 `tablefunc.control`、`tablefunc--1.0.sql` 和 `tablefunc--unpackaged--1.0.sql`，安装后需要针对具体的某个数据库创建这个扩展才能使用：
    ```
    $ psql -U postgres -d mydb
    mydb=# create extension tablefunc;
    CREATE EXTENSION
    mydb=# \q
    ```
    之后就可以在 pgAdminIII 左侧的对象浏览器中看到 `.../数据库/mydb/拓展/tablefunc`
6. 计算两个日期之间的秒值（double）：`SELECT EXTRACT(EPOCH FROM timestamp '2013-01-01 00:01:10') - EXTRACT(EPOCH FROM timestamp '2013-01-01 00:00:10');` <-- EPOCH=1970-01-01 00:00:10
7. 判断元素是否在数组内：`select 1 = any (ARRAY[1,4,3])`
8. 跨数据库读取数据：[dblink 扩展](https://www.postgresql.org/docs/current/static/dblink.html)
    ```
    -- 安装扩展：
    $ psql -U postgres -d mydb
    mydb=# create extension dblink;
    CREATE EXTENSION
    mydb=# \q
    
    -- 读取另外一个 postgres 数据库的数据的例子：
    SELECT * FROM dblink(
      'hostaddr=192.168.0.2 port=5432 dbname=mydb user=reader password=reader',  
      'select id, name from table1'
    ) AS t(id int, name text);
    ```

## 参考：

- <http://wiki.ubuntu.org.cn/PostgreSQL>
- [Postgres 9.1 DEB 安装](http://wiki.openscg.com/index.php/Postgres_9.1_DEB)
- [字符集支持](http://www.pgsqldb.org/pgsqldoc-8.1c/multibyte.html)
- [一些 PostgreSQL 的操作](http://blog.chinaunix.net/space.php?uid=17260303&do=blog&id=2811307)
- [PostgreSQL9.0 新特性介绍： alter default privileges，解决只读用户的问题](http://blog.csdn.net/wh62592855/article/details/6337570)
- [postgresql9.1 sql-revoke](http://www.postgresql.org/docs/9.1/static/sql-revoke.html)
- [postgresql9.1 sql-grant](http://www.postgresql.org/docs/9.1/static/sql-grant.html)
- [postgres 创建只读用户 ，并且取消权限](http://hi.baidu.com/magicgrass/blog/item/591d60fa9556cc78034f568f.html)
- [中文环境下 PostgreSQL 的使用](http://bbs.chinaunix.net/thread-2324339-1-1.html)
- [Disqus 评论设计(一个递归例子)](http://www.oschina.net/translate/scaling-threaded-comments-on-django-at-disqus)
- [PostgreSQL 函数如何返回数据集](http://my.oschina.net/Kenyon/blog/108303)
- [PostgreSQL 的行转列应用](http://my.oschina.net/Kenyon/blog/54357) <http://www.postgresql.org/docs/9.1/static/tablefunc.html> <http://blog.163.com/digoal@126/blog/static/163877040201151253211186>
- [SQL UPDATE FROM 的使用方法](http://www.frogjumpjump.com/2011/09/sql-update-from.html)
- [Errors and Messages](http://www.postgresql.org/docs/9.1/static/plpgsql-errors-and-messages.html)
- [理解 Postgres 的执行计划 (Execution Plan)](http://pgguide.lxneng.com/performance/explain.html)
- [sql 聚合函数](http://my.oschina.net/u/1398304/blog/214262)
- [PostgreSQL vs. MS SQL Server](http://www.oschina.net/translate/postgresql-vs-ms-sql-server)
- [Postgres 访问其他 Postgres 数据库的 dblink 功能](http://blog.csdn.net/hantiannan/article/details/7553108)