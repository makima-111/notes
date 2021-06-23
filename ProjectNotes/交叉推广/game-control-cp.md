# game-control-cp

## 一、概况

* 技术栈：
  * java
  * springboot
  * mybatis
  * docker
  * jetty
* 项目功能：
  * 交叉推广
  * 玩家绑定
  * 发奖

## 二、笔记

### 1、项目结构

#### 1）入口CPAction

* initCofnig （/initCofnig） 初始化配置
  * ->cpService.init()
    * 获取交叉推广计划([cp_plan](*cp_plan))
    * 获取CP开关配置([cp_app_switch](#cp_app_switch))
    * 获取CP基础([cp_base](#cp_base)) 并根据pid appid创建map
    * 获取用户过滤配置([cp_user_filter](#cp_user_filter))
    * 初始化任务 //todo
* rewardUser（/{gameToken}/reward）
  * ->cpService.rewardMission
    * 
* bindUser(/bindUser)
* rewardMission(/rewardMission)
* 

### 2、数据库