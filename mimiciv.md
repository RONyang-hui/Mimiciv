# mimiciv
## 下载数据库文件（>5G），下载数据库建库脚本，安装7-zip并设置环境变量，安装postgreSQL，预留60G大小的硬盘空间。

## 环境变量入口：系统->高级系统设置->高级->环境变量->系统变量，在系统变量里添加7-zip的程序路径。然后使用cmd输入“7z”，如出现一大段英文介绍，则提示设置成功。

## PostgreSQL设置 打开“SQL shell”，输入代码：
DROP DATABASE IF EXISTS mimic;
CREATE DATABASE mimic OWNER postgres;

## 连接到mimic数据库
\c mimiciv; 

## 创建用于保存数据库的模式
CREATE SCHEMA mimiciv;

## 通知 postgres 它应该默认使用该mimiciv模式
set search_path to mimiciv;

## 在架构下创建表
\i E:/mimic-code-main/mimic-iv/buildmimic/postgres/create.sql

## 出现任何错误时停止执行
\set ON_ERROR_STOP 1

## 此命令指定包含数据的文件夹
mimiciv=# \set mimic_data_dir 'F:/mimic-iv-clinical-database-demo-2.2'

mimiciv=# \i F:/Mimic/mimic-code/mimic-iv/buildmimic/postgres/load_7z.sql

## 此处会有COPY 275。。。

## mimiciv=# \encoding 'UTF8'

## 索引
mimiciv=# \i F:/Mimic/mimic-code/mimic-iv/buildmimic/postgres/index.sql

## 示例
mimiciv=# SELECT * FROM mimiciv_hosp.patients LIMIT 10;