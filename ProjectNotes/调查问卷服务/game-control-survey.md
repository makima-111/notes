# game-control-survey 主控服务-问卷相关

### 一、概况

* 技术栈：
  * java
  * springboot
  * mybatis
  * docker
  * jetty
* 项目功能：
  * 调查问卷服务器部分，与survey项目共同使用
  * 提供用户资格验证功能
  * 调用jelly发奖接口
  * 求取入口src/main/java/com/mf/oper/gamecontrol/web/SurveyAction.java
  * 路径配置完成后需要重启服务器
* 更新步骤
  * 配置数据库
    * 添加问卷配置信息 wv_survey_info
    * 配置用户id和tag set_user_collection
    * 配置访问地址
  * 重启服务加载配置

* 项目运行流程
  1. jelly前端登录时调用JellyCPService.execute方法
  2. 转到SurveyService.execute方法，查询用户信息，验证用户资格，初始化用户配置，查询问卷页面（survey-client）url并返回
  3. 前端访问问卷页面
  4. 问卷页面发送请求到服务，调用surveyPage方法，查询用户响应的问卷信息（那个问卷）并返回
  5. 问卷显示->用户答题->用户提交
  6. 问卷页面发送求情，调用submitSurvey方法，提交问卷
  7. 服务器验证用户信息，调用JellyRewardService.webviewReward方法为用户发奖
  8. 记录日志，返回发奖状态。

### 二、笔记

#### 1、数据库结构

##### 1）wv_survey_info 问卷信息配置表

* 配置调查问卷标题、时间、编号、玩家标签等

| 类名                      | 简介                                                         | 描述 |
| ------------------------- | ------------------------------------------------------------ | ---- |
| id                        | id                                                           |      |
| title                     | 标题                                                         |      |
| rewards                   | 发奖内容，需要策划确认                                       |      |
| pre_text                  |                                                              |      |
| start_time                | 开始时间                                                     |      |
| end_time                  | 结束时间                                                     |      |
| pop_max_count             |                                                              |      |
| pop_interval_time         |                                                              |      |
| user_set_tag              | 用户标签，与set_user_collection表中tag对应                   |      |
| enable_test_user          |                                                              |      |
| user_filter_condition_tag |                                                              |      |
| question_page_id          | 暂未使用                                                     |      |
| desc                      |                                                              |      |
| status                    |                                                              |      |
| order                     | 与survey-client 中question.json存放的最后一级目录名一致（数字） |      |
| created                   |                                                              |      |
| updated                   |                                                              |      |
| creator_id                |                                                              |      |
| game_token                | 游戏token                                                    |      |

##### 2）set_user_collection 用户集配置表

* 配置问卷访问玩家集合

| 类名    | 简介                                                     | 描述 |
| ------- | -------------------------------------------------------- | ---- |
| user_id | 玩家真实id                                               |      |
| tag     | 玩家标签，与wv_survey_info对应调查问卷的user_set_tag对应 |      |

##### 3）user_survey_status 玩家调查问卷状态信息

* 记录玩家访问问卷的状态

| 类名       | 简介                                          | 描述 |
| ---------- | --------------------------------------------- | ---- |
| user_id    | 玩家真实id                                    |      |
| read       | 标识是否阅读问卷，与wv_survey_info中order对应 |      |
| rewarded   | 标识是否领奖                                  |      |
| game_token | 游戏token                                     |      |

##### 4）cfg_enviroment 问卷访问地址配置

* 配置各种环境变量，包括survey-client访问地址

| 类名        | 简介 | 描述 |
| ----------- | ---- | ---- |
| name        | 名称 |      |
| environment |      |      |
| value       | 值   |      |
| desc        | 描述 |      |

#### 2、项目结构

* 入口地址：java/com/mf/oper/gamecontrol/web/SurveyAction.java

##### SurveyService

* init() 
  * ScheduleService.refreshCP定时刷新
  * 初始化读取数据库配置
* submitSurvey
  * 由survey-client 提交请求触发调用；
  * 查询数据库用户状态，
  * 记录玩家基本信息
  * 调用jelly发奖 ->JellyRewardService.webviewReward ->HttpUtils.responseForPostData，根据返回值生成日志
  * 更新数据库用户发奖状态
  * 记录问卷日志
  * 返回retCode
* execute
  * JellyAction.execReq->JellyService.exec->JellyCPService.execute->SurveyService.execute
  * jelly前端调用
  * 查询用户基本信息
  * 初始化玩家问卷状态
  * 返回问卷地址及信息
* surveyPage
  * 由survey-client 访问页面请求触发调用；
  * 查询数据库，验证玩家信息是否存在
  * 调用->SurveyService.chooseUserSurveyInfo查询玩家对应的问卷信息
  * 返回问卷信息给survey-client
* chooseUserSurveyInfo
  * surveyPage->chooseUserSurveyInfo
  * 验证用户客户端版本
  * 获取token
  * 查询数据库玩家是否处于问卷集中
  * 动态查询玩家信息（逻辑写死在代码中，动态用户集时使用）
  * 返回问卷信息