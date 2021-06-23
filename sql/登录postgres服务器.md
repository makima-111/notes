## 登录postgres服务器 

### 一、硬件配置

1. 插key
2. 运行pageant.exe 点 Add CAPI Cert

### 二、链接putty

1. 运行putty
2. 配置Session:目标IP
3. Date:用户名称 
4. open打开命令行

### 三、打开psql

1. 切换到root `sudo su`
2. 打开psql `su - postgres`
3. psql -U op_sys oper_db