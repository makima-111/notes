# 一、Docker相关

## 1、修改文件后docker push 显示already ex

* build脚本写错了 注意docker库名称

## 2、JAVA项目本地启动成功，docker启动失败

问题现象：

* 本地启动成功，docker启动失败

* 错误日志和正常日志对比

![image-20210429183340039](D:\notes\img\image-20210429183340039.png)

![image-20210429183355923](D:\notes\img\image-20210429183355923.png)



问题解决

* 经过对比，缺少日志模块加载过程
* 经测试，容器log目录映射错误



# 二、网络相关

## 1、POSTMAN 发送含有body的GET请求返回411

#### 1）问题描述：

* 使用GET请求body携带JSON字段向ctrl.oper-test.corp.lemonwarehouse.com/login?type=json和ctrl.oper-test.corp.microfun.com/login?type=json 发送请求时，后者能够到达域名映射的nginx，前者不能。POST请求都可以到达。

* 411为缺少长度字段，抓包显示存在长度字段，且目的地址为0.51，为poxy地址，请求没有发送到171nginx服务器：

![image-20210514142758090](C:\Users\yinghui.zhang\AppData\Roaming\Typora\typora-user-images\image-20210514142758090.png)

![image-20210514142624116](D:\notes\img\image-20210514142624116.png)

抓包现象：前一个url可以正常获取地址，后一个url请求的是poxy地址

![image-20210514142511612](D:\notes\img\image-20210514142511612.png)

DNS设置：

![image-20210514151025086](D:\notes\img\image-20210514151025086.png)

#### 2）问题分析

* 问题一：GET方法一个可以，一个不可以：可能跟dns后缀配置有关，所以出现了两个不同域名，一个可以成功，一个不能成功的问题。问题可能出现在网络上的某一环，由于ip地址被隐藏，无法追踪流量经过的路径，无法分析具体原因。
* 问题二：POST方法都可以，正常
* 总结：由于域名后缀不同，造成请求路由不同，其中一个路由不支持GET请求BODY携带数据，再转发的时候没有设置或者错误设置了Content-Length头部字段，造成411。



# 三、Java相关

## 1、gradle依赖冲突处理正确的情况下，加载了错误类

### 1）问题描述：

* 背景：
  * 实现Firebase cloud messaging 接入pushService的过程中。
  * FirebaseSDK依赖guava29.0版本
  * 项目原有hive依赖guava15版本
* 问题：
  * 观察gradle传递依赖树，正确加载29版本guava
  * hive标识不引入传递依赖
  * firebaseSDK仍然加载到了15版本的guava里的类，造成方法确实，无法通过。

### 2）分析

* hive模块将它所依赖的所有类一同编译在了jar包里，造成传递依赖相关处理失效。
* 正确的打包应该是只对源码进行编译，不编译依赖模块

### 3）解决过程

* 尝试通过sourceSets对对应包进行排除->失败
* 直接去maven仓库上把guava29版本下载到项目路径下，并在build.guradle 中指定从本地加载->解决问题



