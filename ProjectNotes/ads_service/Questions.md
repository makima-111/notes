# Question

## 一、服务器日志下载时间不正常

### 1）问题描述

1. 从凌晨到下午一直在重复更新本日和昨日日志；下午更新前几日日志

   <img src="D:\notes\img\ads_1.png" alt="7" style="zoom:50%;" />

   ![ads_2](D:\notes\img\ads_2.png)

2. 从凌晨到下午只出现一条开始下载日志

   ![](D:\notes\img\image-20210426181043004.png)

3. 没有api被调用的记录

4. 凌晨25写日志文件流被重复设置

   ![image-20210426182744016](D:\notes\img\image-20210426182744016.png)

5. 任务触发时间与完成时间存在较大延迟![image-20210427102043067](D:\notes\img\image-20210427102043067.png)

6. 

### 2）排查过程：

1. 怀疑是接口被调用（未调用）
2. 怀疑配置中重复获取前一天数据（没有相关逻辑）
3. 下载接口被其他方法触发
4. 下载过程过慢，造成请求阻塞。
5. 

### 3）待测试方法

- [x] 调整定时器时间，加大间隔，减小凌晨实时数据读取。0 */20 5-23
- [x] 增加taskID和tasktime
- [x] 增加锁

### 4）问题原因

* 执行时间有问题，cron采用的是utc时间

## 二、数据下载速度慢 400次异步数据下载耗时接近20分钟

### 1、2021-05-08-10:42

#### 1）问题描述：

* 经过打点测试记录观察时间戳，400条request发出方法同步进入（1s内）
* 400条request为一个promise，回调在20分钟内依次触发
* 回调顺序与启动顺序不完全一致。
* 本地测试20条同渠道数据，1s完成下载。
* 本地测试20条*10天同渠道数据，20s完成下载。

#### 2）疑问：

1. 由于超时时间的问题，20分钟返回有两种可能
   *  实际get request发送时间与进入方法时间不一致，发送依次接近同步产生。
   * getrequest返回数据很快，实际接收处理时有问题，依次同步处理。
2. 下载速度缓慢是出现在哪个层面的：
   *  centos7系统
   * docker
   * node js

3. 本地如何复现缓慢问题
   * 整体拷贝数据库

#### 3）排查方法

1. 针对疑问3，对源数据库响应表进行拷贝测试

   ```bash
   #在172.16.0.52数据库上测试dump命令
   pg_dump -h 172.16.0.41 -n oper_db -p 5432 -U  -t ads_api_authorize -f /home/yinghui.zhang/copy_ads_api_authorize.sql
    ads_api_template ads_channels 
   psql -h 172.16.0.52 -d oper_db -p 5432 -U op_sys -t ads_api_authorize ads_api_template ads_channels -f /home/yinghui.zhang/copy_ads_DB.sql
   pg_dump -h 172.16.0.52 oper_db -p 15432 -U op_sys -t ads_api_authorize   -f /home/yinghui.zhang/copy_ads_DB.sql
   pg_dump -h 172.16.0.41 -n oper_db -p 5432 -U  -t ads_api_authorize -f /home/yinghui.zhang/copy_ads_api_authorize.sql
    ads_api_template ads_channels 
   #
   
   pg_dump -h 172.29.55.161 oper_db -p 21001 -U op_sys -t ads_api_authorize   -f /home/yinghui.zhang/copy_ads_DB.sql
   ```

   客户端与服务器数据不一致，无法使用pg_dump命令跨版本获取线上服务器数据

#### 4）猜测

1. 猜测问题是出现在node.js 发送http.request时的链接数量上。

   

### 2、2021-05-08-10:42

#### 1）猜测：

猜测问题是出现在node.js 发送http.request时的链接数量上。

#### 2）测试方法

- [x] 设置最大链接数到无限大
- [x] 设置最大链接数到10
- [ ] 设置链接状态为keepalive=true

### 3)测试结果

* 默认为无限大
* 无效

### 3、2021-05-11

#### 1）问题：

* 默认代码环境下加入任务log只显示前10条左右（第一个渠道和游戏的任务）
* 任务结束时log显示为300多个任务
* 注释掉await刷新token后，显示前200条左右

#### 2）猜测

1. 与linux的journalctl有关
2. 与nodejs的console.log机制及事件队列 调用堆栈有关