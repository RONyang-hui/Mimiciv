# mimiciv

## 下载数据库文件（>5G），下载数据库建库脚本，安装 7-zip 并设置环境变量，安装 postgreSQL，预留 60G 大小的硬盘空间。

## 环境变量入口：系统->高级系统设置->高级->环境变量->系统变量，在系统变量里添加 7-zip 的程序路径。然后使用 cmd 输入“7z”，如出现一大段英文介绍，则提示设置成功。

## PostgreSQL 设置 打开“SQL shell”，输入代码：

DROP DATABASE IF EXISTS mimiciii;
CREATE DATABASE mimiciii OWNER postgres;

## 连接到 mimic 数据库

\c mimiciii;

## 创建用于保存数据库的模式

CREATE SCHEMA mimiciii;

## 通知 postgres 它应该默认使用该 mimiciv 模式

set search_path to mimiciii;

## 在架构下创建表

\i E:/Mimic/mimic-code/mimic-iii/buildmimic/postgres/postgres_create_tables.sql

## 出现任何错误时停止执行

\set ON_ERROR_STOP 1

## 此命令指定包含数据的文件夹

\set mimic_data_dir 'E:/mimic-iii-clinical-database-1.4'
\i 'E:/Mimic/mimic-code/mimic-iii/buildmimic/postgres/postgres_load_data.sql'

## 此处会有 COPY 275。。。

## mimiciv=# \encoding 'UTF8'

## 索引

\i E:/Mimic/mimic-code/mimic-iii/buildmimic/postgres/postgres_add_indexes.sql

## 示例

SELECT \* FROM mimiciv_hosp.patients LIMIT 10;

# 导入物化视图

## 建立查询路径，注意要与自己在 navicat 设置的 schema 名字相同，我设置的是 mimiciii

set search_path to mimiciii;

## 将默认目录切换到 mimic-code-master 的 concepts 文件夹下

\cd 'E:/Mimic/mimic-code/mimic-iii/concepts_postgres'

## 运行建立视图文件

\i E:/Mimic/mimic-code/mimic-iii/concepts_postgres/postgres-functions.sql
\i E:/Mimic/mimic-code/mimic-iii/concepts_postgres/postgres-make-concepts.sql

## 导入别的视图,在 mimic 数据库下切换到 sepsis 目录

\cd sepsis;

## 直接运行 concept 里的 sepsis 内视图创建 sql 语句

\i angus.sql
