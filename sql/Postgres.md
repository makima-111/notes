# Postgres

## 一、存储相关

show data_directory;

show all;

## 二、COPY 

copy 和\copy命令效果类似，只不过执行权限不通，使用\copy即可，

### 1、导入数据

```sql
\copy op_common.set_user_collection (user_id,tag) from '/root/test/test.csv' with (FORMAT csv,DELIMITER ',',escape '\',header false,quote '"',encoding 'UTF8')
```

### 2、数据格式

```csv
224117494,"tag_user_8"
224318405,"tag_user_8"
```

### 3、导出csv格式数据

```sql
\COPY (SELECT * FROM ads_channels ) TO '/home/yinghui.zhang/ads_channels_file.csv' csv

\COPY (SELECT * FROM ads_api_authorize ) TO '/home/yinghui.zhang/ads_api_authorize_file.csv' csv
```



