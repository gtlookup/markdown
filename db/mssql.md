# 备份

```mssql
backup database test to disk='d:\test.bak' -- 备份test库
```

# 恢复

```mssql
restore database test from disk='d:\test.bak' -- 恢复test库，执行命令前先关闭 test 库
```

<h3>需要改名恢复</h3>

```mssql
-- 1. 先设置离线
ALTER DATABASE flaw SET OFFLINE WITH ROLLBACK IMMEDIATE
-- 2. 改名恢复
restore database flaw from disk='d:\flaw' with 
move 'flaw' to 'D:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\new_flaw.mdf',         -- 重命名个.mdf
move 'flaw_log' to 'D:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\new_flaw_log.ldf', -- 重命名个.ldf
replace
-- 3. 设置上线
ALTER DATABASE flaw SET ONLINE
```

==以上是SQL2019亲测==

# 全局操作

```mssql
SELECT NAME FROM MASTER..SYSDATABASES       --读取所有库
SELECT NAME FROM SYSOBJECTS WHERE TYPE='U'  --读取所有表
```

```mssql
-- 指定表所有字段信息
SELECT
    表名 = case when a.colorder = 1 then d.name else '' end,
    表说明 = case when a.colorder = 1 then isnull(f.value, '') else '' end,
    字段序号 = a.colorder,
    字段名 = a.name,
    是否自增 = case when COLUMNPROPERTY(a.id, a.name, 'IsIdentity')= 1 then '√'else '' end,
    主键 = case when exists(
        SELECT 1 FROM sysobjects 
        where xtype = 'PK' and parent_obj = a.id and name in (
            SELECT name FROM sysindexes WHERE indid in(
                SELECT indid FROM sysindexkeys WHERE id = a.id AND colid = a.colid
            )
        )
    ) then '√' else '' end,
    类型 = b.name,
    占用字节数 = a.length,
    长度 = COLUMNPROPERTY(a.id, a.name, 'PRECISION'),
    小数位数 = isnull(COLUMNPROPERTY(a.id, a.name, 'Scale'), 0),
    允许空 = case when a.isnullable = 1 then '√'else '' end,
    默认值 = isnull(e.text, ''),
    字段说明 = isnull(g.[value], '')
FROM syscolumns a
left join systypes b on a.xusertype = b.xusertype
inner join sysobjects d on a.id = d.id  and d.xtype = 'U' and d.name <> 'dtproperties'
left join syscomments e on a.cdefault = e.id
left join sys.extended_properties g on a.id = G.major_id and a.colid = g.minor_id
left join sys.extended_properties f on d.id = f.major_id and f.minor_id = 0
where d.name = '表名'
order by a.id,a.colorder
```

```mssql
-- 列名、列类型、列说明
SELECT
	T1.name,         -- 列名
	T3.name type,    -- 列类型
	T4.value comment -- 列说明
FROM syscolumns T1
INNER JOIN sysobjects T2 ON T1.id=T2.id AND T2.type='U'
LEFT JOIN systypes T3 ON T1.usertype=T3.usertype
LEFT JOIN sys.extended_properties T4 ON T1.id=T4.major_id AND T1.colid=T4.minor_id
WHERE T2.name='data1_202104'
ORDER BY T1.colorder -- 按表结构出现的顺序排序
```

