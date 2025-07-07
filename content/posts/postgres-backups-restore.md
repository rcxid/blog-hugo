+++
title = "PostgreSQL数据库备份与恢复"
date = "2025-07-07T16:27:49+08:00"
summary = "PostgreSQL数据库备份与恢复"
categories = ["数据备份"]
tags = ["数据库", "postgres"]
draft = false
+++

## 1.使用 `pg_dump` 命令行工具

`pg_dump` 是一个用于备份和恢复PostgreSQL数据库的命令行工具。

要进行全量备份，可以使用以下命令：
```bash
pg_dump -U username -h hostname -p port database_name > backup_file.sql
```
例如：
```bash
pg_dump -U postgres -h localhost -p 5432 postgres > postgres_backup.sql
```

## 2. 使用 `psql` **恢复** `.sql` **文件**

如果你使用的是 `pg_dump` 的纯文本格式（例如 `.sql` 文件），可以使用 `psql` 来恢复：
```bash
psql -U username -h hostname -p port -d database_name < backup_file.sql
```
例如：
```bash
psql -U postgres -h localhost -p 5432 -d postgres < postgres_backup.sql
```