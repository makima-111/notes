# DotService

## 一、概况

* 技术栈：
  * java
  * springboot
  * docker
  * jetty
* 项目功能：
  * 接收并记录打点日志

## 二、笔记

## 三、更新记录

### 1、2021/05/11 更新支持UDP日志收集（公司防护墙日志）

* 增加UDPListener
* 增加类microfunFirewallLogServer实现接收UDP、记录日志、返回应答所有功能
* UDPListener启动根据传入参数控制启动 `UDPListener=on`
* 存放地址为data/logs/microfun/

```shell
#启动命令
#!/bin/bash
UDP_PORT=${1-8090}
OPERDOT_ENV=${2-dev}
HOST_NAME=${3-operdot}

docker pull docker.microfun.cn/oper/oper-dot

docker stop "$HOST_NAME" && docker rm "$HOST_NAME"

docker run -v /sys/fs/cgroup:/sys/fs/cgroup:ro --privileged=true --sysctl net.ipv6.conf.all.disable_ipv6=1 \
    -v /data/logs/dot:/data/logs \
    --name "$HOST_NAME" -h "$HOST_NAME" \
    -p 8080:8080 \
    -p 19090:19090 \
	-p ${UDP_PORT}:${UDP_PORT}/udp \
    -e "TZ=Asia/Shanghai" \
    --env OPERDOT_ENV="$OPERDOT_ENV" \
    --env UDP_PORT=${UDP_PORT} \
    -d docker.microfun.cn/oper/oper-dot || exit $?
```

### 2、2021/05/13 更新支持UDP日志收集配置功能

### 3、2021/06/07 支持从请求头部获取信息

#### 一、需求

能供从请求同步取出数据并存入日志中。

#### 二、要求

* 要支持从s3上读取配置。
* 支持接收请求刷新配置

#### 三、实现

* 暂时从内存读取配置，配置接口，之后从s3上读取配置。

##### 1、配置数据结构

采用JSON格式：

* gameToken1
  * configName1:[configArray]
  * configName2:[configArray]
* gameToken2
  * configName1:[configArray]

```json
{
    "3a9d70":{
        "headers":[
            "candy_client",
            "candy-client"
        ]
    }
}
```

##### 2、类实现结构

![image-20210621104543348](D:\notes\img\image-20210621104543348.png)

## 四、故障排查

### 1、海外服务器OOM 宕机 20210611~0618

#### 1）问题及背景

##### 背景

* candy网络更换
* dot服务更新新功能：从请求头中取参数并记录
* 打点请求来源有两个：1、oper-dot-alb=>dotService；2、oper-onesdk-alb =>oper-gateway-1(nginx) =>dotService

##### 问题

1. 从2021年6月11日开始，海外dot服务器出现oom宕机情况，每日1~2台。
2. 通常连续宕机的不是同一台服务器。
3. 宕机原因均为系统内存不足
4. 停用新功能后仍然发生宕机情况
5. dotService控制台爆出 Early EOF和idle timeout错误
6. dotService有几千条socket状态为TIME_WAITE

#### 2）排查过程

##### 通过切换两种打点来源观察发现：

1. 问题5(dotService控制台爆出 Early EOF和idle timeout错误)由来源1(oper-dot-alb=>dotService)产生
2. 问题6(dotService有几千条socket状态为TIME_WAITE)由来源2(oper-onesdk-alb =>oper-gateway-1(nginx) =>dotService)产生

##### 抓包观察idle timeout错误原因

1. tcpdump 抓包并用wireshark分析

```shell
tcpdump -i eth0 port 8080 -f -vvv -w /home/yinghui.zhang/tcpdumplog.cap
```

2. post请求由两个tcp包组成，分别发送：1、请求头（协议、方法、headers）；2、请求体（body）
3. 服务器收到请求的第一个包(请求头)，但未收到请求的第二个包(请求体)，产生错误异常，

##### 通过增加定期输出内存占用的监控脚本分析原因

1. oom时dotService和logService占用均不低，造成宿主机oom
2. 内存不足，造成虚拟内存频繁页面置换，产生系统盘io报警
3. 宿主机oom后杀掉两个进程
4. dotService重启成功，logService重启失败

#### 3）解决办法

1. logService更换为h2o服务