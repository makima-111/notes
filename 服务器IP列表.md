| 公网IP          | 内网IP        | 描述                                            |
| --------------- | ------------- | ----------------------------------------------- |
| 172.16.0.41     | 172.16.0.41   | 各种本地测试服务部署地址                        |
| 34.213.133.231  |               | 海外gateway                                     |
| 34.214.255.18   |               | 海外服务器gateway1                              |
| 172.26.1.161    | 172.26.1.161  | 海外服务测试服务地址                            |
| 161.189.121.140 |               | gateway-1;adsService                            |
| 172.29.55.109   | 172.29.55.109 | 生产环境postgre数据库客户端服务器 需要gateway-1 |
| 211.145.33.146  |               |                                                 |
| -               | 172.16.0.52   | mfop-test-db                                    |
| -               | 172.16.0.251  | oper-test-web-01                                |
| -               | 172.16.0.252  | oper-test-web-02                                |
| -               | 172.16.0.171  | 192网段nginx                                    |
| 34.213.255.18   |               | gate-us-way-1 onesdk网关                        |
| 52.42.126.251   | 172.26.4.218  | mfop-us-dot-3-new                               |
| 52.25.180.239   |               | mfop-us-dot-2                                   |
| 50.112.36.164   |               | mfop-us-dot-1                                   |
|                 |               |                                                 |
|                 |               |                                                 |
|                 |               |                                                 |
|                 |               |                                                 |



# 172.16.0.41 41Service

1. 测试postgre数据库
2. ads本地测试服务 docker内 
3. 测试nginx服务器
4. gamecontrol 测试服务 docker内

# 34.213.133.231 adsTestGW

1. 0323测试ads 跳板机地址



# 34.214.255.18 adsTestService

1. 0323测试ads 服务部署地址



# 172.26.1.161 adsTestDB

1. 0323测试ads 数据库地址
   * 存储模板，tocken等数据
   * 为vps虚拟地址，需要从aws公网adsTestGW链接
   *  psql -h 172.26.1.161 -p 21001  -U op_sys -d oper_db
   *  psql -h 172.29.55.161 -p 21001  -U op_common -d oper_db



# 161.189.121.140 gateway-1 adsService

1. 链接ads生产环境数据库跳板机
2. ads国内生产环境服务器 p
3. 

# 172.29.55.109  链接adsDB

1. 链接ads生产环境数据库 psql -h 172.29.55.161 -p 21001  -U op_sys -d oper_db  mf_oper#sys#2019
   * 存储模板，tocken等数据

# 211.145.33.146 oper_db跳板机



# 172.16.0.52 mfop-test-db

* 部署test/oper_pgdb



# 172.16.0.251 oper-test-web-01 

* 部署打点服务 192.168.10.254
* dot.oper-test.corp.microfun.com 

# 172.16.0.252 oper-test-web-02 

# 172.16.0.171 nginx

* 能够访问到192ip nginx 



