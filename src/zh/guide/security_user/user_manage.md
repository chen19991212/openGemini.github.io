---
title: 用户管理
order: 3
---

本部分内容主要介绍在openGemini内部如何创建用户、删除用户、授权等。  
openemini无内置账号，开启鉴权，并成功启动后，需要主动创建系统唯一的管理员账号。openGemini的账号分为管理员账号和普通账号，对应角色分别是admin和user，管理员账号在系统内只能创建一次，不能删除，并且不能重命名。普通账号可以创建多个，由管理员创建，并且需要管理员授权才可以访问数据库。  
普通用户的权限和DB关联，权限分为READ、WRITE、ALL三种，分别表示可读、可写、可读写。如果一个普通用户被授予对DB的READ权限，则这个普通用户只能查询该DB的元数据以及DB内表中数据。

## 初始状态创建管理员账号

```sql
> curl -i -XPOST "https://ip:8086/query" -k --insecure --data-urlencode "q=CREATE USER admin WITH PASSWORD 'admin-passwd' WITH ALL PRIOVILEGES"
```
| 该命令创建了一个管理员账号，账号名称admin

:::tip
1. 执行时，需要将命令中的IP地址和端口替换为实际环境中的ts-sql的ip和port，同时设置管理员账号名和密码
2. 密码必须由大小写字母、数字、特殊字符组成，长度限制为8-256位字符
3. config/weakpasswd.properties为若密码配置文件，默认支持若密码校验，如果设置的密码与配置文件中的密码一致，则视为若密码，不允许使用。
4. 密码字符串必须用单引号引起来，验证请求时，请包含单引号。
5. <font color=red>不建议</font>在密码中使用单引号（‘）和反斜杠（\）字符，对于包含这些字符\’的密码，在创建密码和提交身份验证请求时，请使用反斜杠对特殊字符进行转义
:::

::: danger

出于安全考虑，openGemini的管理员账号在系统内只能创建一次，不能删除，并且不能重命名。创建管理员账户前，请认真考虑用户名和密码`<username>`，并做好账号和密码的保存。

:::

## 账号登陆

```sql
> auth
username: admin  
password: 
```

## 创建普通用户
需要使用管理员登陆，才可以创建普通用户

```sql
> CREATE USER user0 WITH PASSWORD 'your_pwd' WITH PARTITION PRIVILEGES
```

## SHOW USERS
查看所有现有用户及其管理员状态

```sql
> SHOW USERS
+-------+-------+
| user  | admin |
+-------+-------+
| user0 | admin | 
+-------+-------+
2 columns, 1 rows in set
```

## DROP USER
删除用户

**语法**

```sql
DROP USER <username>
```

**示例**

```sql
DROP USER "user0"
```

## GRANT
授予现有用户对DB的 `READ`, `WRITE` or `ALL` 权限

**语法**

```sql
GRANT [READ,WRITE,ALL] ON <database_name> TO <username>
```

**示例**

授权`user0`对 `NOAA_water_database` 数据库读权限:

```sql
GRANT READ ON "NOAA_water_database" TO "user0"
```

授权`user0`对`NOAA_water_database` 数据库所有权限:

```sql
GRANT ALL ON "NOAA_water_database" TO "user0"
```

## SHOW GRANTS
查看指定用户现有权限

**语法**

```sql
> SHOW GRANTS FOR <username>
```

**示例**

```sql
> SHOW GRANTS FOR "user0"
+----------------------+-----------+
| database             | privilege |
+----------------------+-----------+
| NOAA_water_database  | READ      |
+----------------------+-----------+
2 columns, 1 rows in set
```

## REVOKE
回收用户权限

**语法**

```sql
> REVOKE [READ,WRITE,ALL] ON <database_name> FROM <username>
```

**示例**

取消 `user0`用户对 `NOAA_water_database` 数据库的写权限:

```sql
> REVOKE WRITE ON "NOAA_water_database" FROM "user0"
```

取消 `user0`用户对 `NOAA_water_database` 所有权限：

```sql
> REVOKE ALL ON "NOAA_water_database" FROM "user0"
```

## SET PASSWORD
重置密码

**语法**

```sql
> SET PASSWORD FOR <username> = '<password>'
```

**示例**

```sql
> SET PASSWORD FOR "user0" = 'your_pwd'
```

::: tip

密码字符串必须用单引号引起来，验证请求时，请包含单引号

建议避免在密码中使用单引号（‘）和反斜杠（\）字符，对于包含这些字符\’的密码，在创建密码和提交身份验证请求时，请使用反斜杠对特殊字符进行转义，（例如（））

:::
