# Game-control-login

## 一、概况

* 技术栈：
  * java
  * springboot
  * mybatis
  * docker
  * jetty
* 项目功能：
  * 玩家登录发送求情，采取相关动作。

## 二、笔记

### 1、项目结构

#### 1）Login入口：UserLoginAction，（/userLogin)

* /user-agent：login( )
  * 获取userAgent和IP
  * ->[uaLoginService.login](#UALoginService.login)
* /updateLevel：updateLevel( )
  * ->userBaseService.updateLevel
* /json：login( )
  * 解析用户基本信息
  * ->[jsonLoginService.login](#JSONLoginService.login)
  * 记录日志

#### 2）*JSONLoginService

* init()：启动时初始化
  
  * 初始化GAME_SERVICE：Map(gametoken,登录服务)
* <span id = "JSONLoginService.login">login</span>
  1. 设置gameToken

  2. 标准化设备名

  3. ->[DeviceServiceImpl.queryMFPIdByUserInfo](#queryMFPIdByUserInfo)  //获取mfpId并设置

  4. 根据token加载游戏登录服务

  5. ->IGameLoginService.execute 执行所有服务
* [userBaseService](#UserBaseServiceImpl)  更新用户基本数据
  
* [bulletinService](#BulletinService) 发布公告，公告领奖等
  
* [configService](#LConfigServiceImpl)  向response中插入登录基础配置和gane_event配置(adjust)
  
6. 返回应答

#### 3）DeviceServiceImpl

*  <span id = "queryMFPIdByUserInfo">queryMFPIdByUserInfo</span>
  * ->blackIdentifier.contain 黑名单过滤(初始化时查询[identifier_blacklist](#identifier_blacklist)表)
  * 根据用户信息创建deviceinfo->查数据库device_base 获得mfpId
  * 如果mfpId为空，则根据deviceinfo创建新的mfpId并插入
  * 如果老用户采用了新设备，则更新deviceinfo

#### 4）<span id = "UserBaseServiceImpl ">UserBaseServiceImpl </span>

* execute->addOrUpdateUserBase
* addOrUpdateUserBase
  * 通过user_id从数据库获取用户基本信息([user_base](#user_base))
    * 如果没有数据，则插入一条
    * 如果有，则更新变化

#### 5）<span id = "BulletinService ">BulletinService</span>

* init() 创建时初始化
  * 从数据库获取所有公告配置（[bulletin_config](#bulletin_config)）
  * 对于每个公告
    * 判断过滤类型
      * 条件过滤：查询bulletin_user_filter获取过滤条件
      * 固定用户集：查询用户bulletin_uid_list
    * 将公告按用户分组
* execute（）
  * ->[getUserBulletinList](#getUserBulletinList)  获取用户公告列表
  * 获取公告配置，并设置返回值及url
  * ->bi.[bi](#bi) bi打点
  * 返回信息

* <span id = "getUserBulletinList ">getUserBulletinList</span>
  * 获取游戏对应公告
  * 查询用户公告状态([user_bulletin_status](#user_bulletin_status))
  * 将有效公告加入用户公告列表（是否查看过，是否领过奖，条件过滤，固定用户集等）
  * 返回用户公告列表
* updateStatus 更新用户读状态
  * 查询用户公告状态([user_bulletin_status](#user_bulletin_status))
  * 更新读状态
* rewardUser 更新用户发奖状态
  * 根据唯一ID获取用户状态
  * 根据游戏token获取对应rewardService(ie JellyBulletinRewardService)
  * ->rewardService.rewardToUser 发奖
  * 更新用户公告状态

#### 6）BI

* <span id = "bi">bi</span>
  * 获取打点服务url
  * new BITASK
    * dot()：将自身提交给线程池
    * BITASK.run：拼接打点地址，记录日志，并请求bi打点。

#### 7）<span id = "LConfigServiceImpl">LConfigServiceImpl</span>

* init()  定时触发初始化
  * 获取登录配置（[login_base_config](#login_base_config)）
  * 根据APP和配置类型分配
* execute
  * ->addBaseConfig
* addBaseConfig
  * ->[getGameLoginConfig](#getGameLoginConfig)  获取有效login配置
  * 将配置信息加入response内
  * ->[gameEventService.injectGameEvent](#GameEventService)
* <span id = "getGameLoginConfig">getGameLoginConfig</span>
  * 根据用户信息获取游戏、渠道login配置
  * ->addToResponse 检测配置是否有效

#### 8）<span id = "GameEventService">GameEventService</span>

* init()
  * 海外不启用直接return
  * 获取活动信息（[game_event_setting](#game_event_setting)）
* injectGameEvent
  * 海外游戏暂时继续从login_base表读取event数据
  * response中插入(adjust,活动信息)

#### 9）UALoginService

* <span id = "UALoginService.login">login</span>
  * ->UALoginParamConvertor.initUserSnsInfo 解析userAgent->userSnsInfo
  * ->[DeviceServiceImpl.queryMFPIdByUserInfo](#queryMFPIdByUserInfo)  //获取mfpId并设置
  * ->[userBaseService.execute](#UserBaseServiceImpl)  更新用户基本数据
  * ->[cpService.execute](#CPServiceImpl.execute) 交叉推广
  * ->[bulletinService.execute](#BulletinService) 向response中插入发布公告，公告领奖等
  * ->[userStatusService.addUserStatus](#UserStatusServiceImpl.addUserStatus) 向response中插入用户状态
  * ->shakeUserValueService.execute 向response中插入用户价值挖掘信息 //todo 未看
  * ->[configService.execute](#LConfigServiceImpl) 向response中插入登录基础配置和gane_event配置(adjust)
  * response

#### 10）CPServiceImpl (已废弃)

* init()
  * 获取交叉推广计划([cp_plan](*cp_plan))
  * 获取CP开关配置([cp_app_switch](#cp_app_switch))
  * 获取CP基础([cp_base](#cp_base)) 并根据pid appid创建map
  * 获取用户过滤配置([cp_user_filter](#cp_user_filter))
  * 初始化任务 //todo
* <span id = "CPServiceImpl.execute">execute</span>
  * ->addCrossPromo;
* addCrossPromo
  * 黑名单过滤
  * 根据mfpID获取用户任务信息(cp_user_mission)
  * 如果有用户任务信息
    * 判断用户任务是否有效
    * ->reachedDayLimit 判断是否达到每日上限(cp_status)
    * ->checkMissionStatus根据用户b游戏信息更新用户任务列表及状态
    * 返回信息
  * 如果没有
    * 根据appid 获取任务列表
    * 更新用户任务表

#### 11）UserStatusServiceImpl

* <span id = "UserStatusServiceImpl.addUserStatus">UserStatusServiceImpl.addUserStatus</span>
  * 获取用户状态([user_status](#user_status))
  * 解析用户状态并加入response

#### 12）CPServiceV2Impl

* init()
  * 获取交叉推广计划([cp_plan](*cp_plan))
  * 根据计划进行分类
  * 获取所有任务信息([cp_plan_mission](#cp_plan_mission))
  * 初始化任务
* <span id = "CPServiceImpl.execute">execute</span>
  * ->addCrossPromo;
* addCrossPromo
  * 根据mfpID获取用户任务信息([user_cp_mission](#user_cp_mission))
  * 如果有用户任务信息
    * 判断用户任务是否有效
    * 查询用户绑定状态
    * 获取uid
  * 如果没有
    * 根据token获取任务列表
    * 创建用户任务表
  * 获取用户信息
  * ->checkMission 检查任务是否完成
  * ->saveUserCPMission 保存用户任务表
  * response

### 2、数据库

#### 1）<span id = "identifier_blacklist ">identifier_blacklist </span>黑名单表

| 列名       | 简介         | 描述                |
| ---------- | ------------ | ------------------- |
| identifier | 黑名单识别符 | “idf(imei)”和id拼成 |

#### 2）<span id = "device_base ">device_base </span>设备device_info<->mfp_id n:1

| 列名        | 简介                                         | 描述          |
| ----------- | -------------------------------------------- | ------------- |
| device_info | 设备信息（idfaid，imeiid等，每个id对应一条） | 主键、b树索引 |
| mfp_id      | 用户id标识                                   |               |
| create_time | 创建时间                                     |               |

#### 3）<span id = "user_base ">user_base </span>用户基本信息

| 列名            | 简介   | 描述                         |
| --------------- | ------ | ---------------------------- |
| user_id         | 用户id | 与token组成唯一约束，b树索引 |
| install_date    |        |                              |
| ip              |        |                              |
| device_id       |        |                              |
| vendor_id       |        |                              |
| openudid        |        |                              |
| idfa            |        |                              |
| imei            |        |                              |
| mfp_id          |        | 与token组成，b树索引         |
| last_login      |        |                              |
| level           |        |                              |
| channel         |        |                              |
| client_ver      |        |                              |
| current_os_name |        |                              |
| install_os_name |        |                              |
| game_token      | token  |                              |

#### 4）<span id = "bulletin_config ">bulletin_config </span>公告配置表

| 列名              | 简介     | 描述 |
| ----------------- | -------- | ---- |
| id                | id       | 自增 |
| title             | 标题     |      |
| content           | 内容     |      |
| rewards           | 奖励     |      |
| link              | 链接     |      |
| button_text       |          |      |
| begin_date        | 开始时间 |      |
| end_date          | 结束时间 |      |
| order_num         | 排序id   |      |
| pop_max_count     | 弹出次数 |      |
| pop_interval_time |          |      |
| always_top        |          |      |
| delete_on_read    |          |      |
| user_filter_type  |          |      |
| status            |          |      |
| creater           |          |      |
| create_time       |          |      |
| game_token        |          |      |

#### 5）<span id = "bulletin_user_filter ">bulletin_user_filter </span>公告过滤条件表

| 列名        | 简介                        | 描述                     |
| ----------- | --------------------------- | ------------------------ |
| id          | id                          | 主键                     |
| field       | 计算数据类型（user_id等等） | getUserSnsInfoFieldValue |
| operator    | 计算方式（in,not in等等）   | OperatorUtils.valid      |
| val         | 计算值                      |                          |
| bulletin_id | 公告id                      |                          |

#### 6）<span id = "bulletin_uid_list ">bulletin_uid_list </span>公告用户集

| 列名        | 简介   | 描述 |
| ----------- | ------ | ---- |
| id          | id     | 主键 |
| uid         | 用户id |      |
| bulletin_id | 公告id |      |

#### 7）<span id = "user_bulletin_status ">user_bulletin_status </span>公告用户状态

| 列名        | 简介         | 描述                         |
| ----------- | ------------ | ---------------------------- |
| id          | 唯一id       | userId+OSName+gametoken 生成 |
| read_list   | 读列表       |                              |
| reward_list | 获取奖励列表 |                              |

#### 8）<span id = "login_base_config ">login_base_config	 </span>登录配置

| 列名          | 简介 | 描述 | 示例   |
| ------------- | ---- | ---- | ------ |
| id            | id   | 主键 | 6      |
| key           |      |      | reward |
| value         | 值   |      | 1,6,1  |
| belong        | 类型 |      | share  |
| app           |      |      | 11     |
| cids          |      |      | 29.26  |
| start_version |      |      | 7.4.8  |
| end_version   |      |      | 7.5.8  |
| odd_even      |      |      | -1     |

#### 9）<span id = "game_event_setting  ">game_event_setting	</span>游戏活动信息

| 列名        | 简介      | 描述 |
| ----------- | --------- | ---- |
| event_token | 活动token |      |
| event_name  | 活动名    |      |
| type        | 类型      |      |
| extend_val  | 扩展值    |      |
| desc        |           |      |
| valid       | 是否有效  |      |
| os_name     | 系统名    |      |
| game_token  | token     |      |
| create_time |           |      |
| operator    |           |      |

#### 10）<span id = "cp_app_switch">cp_app_switch</span> 交叉推广开关表

| 列名            | 简介   | 描述 |
| --------------- | ------ | ---- |
| app_id          | app_id |      |
| pswitch         | 开关   |      |
| conversion_rate | 转化率 |      |

#### 11）<span id = "cp_base ">cp_base </span> cp	基础

| 列名                  | 简介 | 描述 |
| --------------------- | ---- | ---- |
| pid                   |      | 自增 |
| pcode                 |      |      |
| app_id                |      |      |
| bapp_id               |      |      |
| start_time            |      |      |
| end_time              |      |      |
| pop_type              |      |      |
| pop_times             |      |      |
| target_install_of_day |      |      |
| status                |      |      |
| duration              |      |      |
| mission_limit         |      |      |
| install               |      |      |
| pri_level             |      |      |
| cdn                   |      |      |
| pop_poor_energy       |      |      |
| pop_radio             |      |      |
| pop_pe_times          |      |      |
| enable                |      |      |

#### 10）<span id = "user_status">user_status</span> 用户状态表

| 列名  | 简介 | 描述                                   |
| ----- | ---- | -------------------------------------- |
| id    |      | 453351112                              |
| value |      | {"cs":{"message":1},"oa":{"follow":1}} |

#### 11）<span id = "cp_user_filter">cp_user_filter</span>  交叉推广用户过滤

| 列名               | 简介 | 示例                | 描述 |
| ------------------ | ---- | ------------------- | ---- |
| id                 |      | 10218               |      |
| pid                |      | 218                 |      |
| pay                |      | 1                   |      |
| level              |      | 30                  |      |
| install_limit_date |      | 2019-01-01 00:00:00 |      |
| channels           |      |                     |      |
| versions           |      |                     |      |
| uid_type           |      | 2                   |      |
| uid_endswitch      |      |                     |      |
| limit_uids         |      |                     |      |
| enable             |      | 1                   |      |

#### 12）<span id = "cp_plan">cp_plan</span> 交叉推广计划

| 列名           | 简介    | 示例                | 描述 |
| -------------- | ------- | ------------------- | ---- |
| id             |         | 1                   |      |
| token          | a-token | 89741e              |      |
| cp_token       | b-token | 51bcd0              |      |
| mission_set_id |         | cake-jelly-plan2020 |      |
| start_time     |         | 2020-09-27 00:00:00 |      |
| end_time       |         | 2020-10-27 00:00:00 |      |
| status         |         | 1                   |      |
| priority       |         | 1                   |      |
| cdn            |         |                     |      |

#### 13）<span id = "cp_plan_mission">cp_plan_mission</span> 交叉推广任务

#### 14）user_cp_mission 用户cp任务表

#### 15）user_cp_bind 用户cp绑定表

## 三、更新记录

#### 1、2021/05/11 更新Candy渠道

* 增加candy渠道
* 增加统一接口方法
* 去除.do后缀要求
* 增加candy adjust打点配置

```sql
INSERT INTO login_base_config (key,value,belong,app)
values
('chop','{"token": "2ehm0r"}','adjust',183),
('get_box','{"token": "menzxb"}','adjust',183),
('iap','[{"pay_id": "extraisland_1", "token": "rowiyb" },{"pay_id": "getgem_1", "token": "9kan6v" },{"pay_id": "gift_energy_3", "token": "1ycglj" },{"pay_id": "gift_tb_hr", "token": "f3gfum" },{"pay_id": "gift_welcome_1", "token": "kstgat" },]','adjust',183),
('levelup','[{"level": 5, "token": "8iervz","fben": "level_up_5"},{"level": 6, "token": "kzy8g4","fben": "level_up_6"}]','adjust',183),
('merge','[{"target": 3, "token": "s0wt24"}, {"target": 5, "token": "psxuom", "fben": "merge_5"}, {"target": -1, "token": "ff5gei"}]','adjust',183),
('start_order','{"token": "t2v3fd"}','adjust',183),
('end_order','{"token": "td028v"}','adjust',183),
('first_pay','{"token": "5btlo5", "fben": "first_pay"}','adjust',183),
('start_purchase','{"token": "i9su17"}','adjust',183),
('task','[{"id": "D2_SuperMerge", "token": "b8zvlb", "fben": "task_super_merge"}]','adjust',183),
('unlock_element','{"token": "v8z07c"}','adjust',183),
('unlock_fog','{"token": "ppb1rp"}','adjust',183),
('useitem','[{"type": "gem", "token": "bt8e0l", "fben": "use_gem"}, {"type": "gold", "token": "xhc015"}]','adjust',183),
('watch_ad','{"token": "3lsnpy"}','adjust',183),


('chop','{"token": "8xg5xu"}','adjust',184),
('get_box','{"token": "c4u6kd"}','adjust',184),
('iap','[{"pay_id": "extraisland_1", "token": "tf4ts0" },{"pay_id": "getgem_1", "token": "lr29gz" },{"pay_id": "gift_energy_3", "token": "n66pxw" },{"pay_id": "gift_tb_hr", "token": "1pcvqx" },{"pay_id": "gift_welcome_1", "token": "sock5r" },]','adjust',184),
('levelup','[{"level": 5, "token": "d93id5","fben": "level_up_5"},{"level": 6, "token": "tn2zsu","fben": "level_up_6"}]','adjust',184),
('merge','[{"target": 3, "token": "woo8gu"}, {"target": 5, "token": "hu88o6", "fben": "merge_5"}, {"target": -1, "token": "vu3uwb"}]','adjust',184),
('start_order','{"token": "sg6pj2"}','adjust',184),
('end_order','{"token": "lbvv6b"}','adjust',184),
('first_pay','{"token": "1oxq9q", "fben": "first_pay"}','adjust',184),
('start_purchase','{"token": "pjicjx"}','adjust',184),
('task','[{"id": "D2_SuperMerge", "token": "wnllcy", "fben": "task_super_merge"}]','adjust',184),
('unlock_element','{"token": "84oz1v"}','adjust',184),
('unlock_fog','{"token": "wmwibq"}','adjust',184),
('useitem','[{"type": "gem", "token": "879up7", "fben": "use_gem"}, {"type": "gold", "token": "vyczh2"}]','adjust',184),
('watch_ad','{"token": "f5mvtu"}','adjust',184);
```

* 更新candy打点配置

```sql
UPDATE  login_base_config SET value='[{"pay_id": "extraisland_1", "token": "rowiyb" },{"pay_id": "getgem_1", "token": "9kan6v" },{"pay_id": "gift_energy_3", "token": "1ycglj" },{"pay_id": "gift_tb_hr", "token": "f3gfum" },{"pay_id": "gift_welcome_1", "token": "kstgat" }]' where key='iap'and app=183;
UPDATE  login_base_config SET value='[{"pay_id": "extraisland_1", "token": "tf4ts0" },{"pay_id": "getgem_1", "token": "lr29gz" },{"pay_id": "gift_energy_3", "token": "n66pxw" },{"pay_id": "gift_tb_hr", "token": "1pcvqx" },{"pay_id": "gift_welcome_1", "token": "sock5r" }]' where key='iap'and app=184;
```

* 更新candy打点配置

```sql
UPDATE  login_base_config SET value='{"token": "i9su17","fben": "start_purchase"}'where key='start_purchase'and app=183;
UPDATE  login_base_config SET value='{"token": "3lsnpy","fben": "watch_ad"}'where key='watch_ad'and app=183;
UPDATE  login_base_config SET value='{"token": "pjicjx","fben": "start_purchase"}'where key='start_purchase'and app=184;
UPDATE  login_base_config SET value='{"token": "f5mvtu","fben": "watch_ad"}'where key='watch_ad'and app=184;
```

#### 2、2021/05/13 从大数据拷贝user_base表数据（未完成）

```csv
user_id,"install_date","ip","device_id","vendor_id","openudid","idfa","imei","mfp_id","last_login","level","channel","client_ver","current_os_name","install_os_name","game_token"


\copy op_common.user_base (user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,channel,client_ver,current_os_name,install_os_name,game_token) from '/root/test.csv' with (FORMAT csv,DELIMITER ',',escape '\',header false,quote '"',encoding 'UTF8')

101012,"2020-11-05 18:19:19.094",,"4EF4598C-7157-4E6B-B032-45B5BCCDABF4","e1a2284f503bf169e2ebf4764de4963881f13902","6AED3514-2FD6-4D6D-9E8D-3BB6CBE9F117","imei","f290a92d5f7c66b90ff243d3a06a4c23","2020-11-05 18:19:19.096085",1,"4","7.8.5","Android","Android","3a9d70"
```

#### 3、2021/05/14 数据库user_base新增3个字段，userBase增加对应属性

1）增加字段

```sql
###增加三列
ALTER TABLE user_base
ADD network_name character varying(128);
ALTER TABLE user_base
ADD tracker_name character varying(256);
ALTER TABLE user_base
ADD adid character varying(64);
```

#### 4、2021/05/17 海外数据库更新配置

```sql
INSERT INTO login_base_config (key,value,belong,app)
values
('chop','{"token": "2ehm0r"}','adjust',183),
('get_box','{"token": "menzxb"}','adjust',183),
('iap','[{"pay_id": "extraisland_1", "token": "rowiyb" },{"pay_id": "getgem_1", "token": "9kan6v" },{"pay_id": "gift_energy_3", "token": "1ycglj" },{"pay_id": "gift_tb_hr", "token": "f3gfum" },{"pay_id": "gift_welcome_1", "token": "kstgat" }]','adjust',183),
('levelup','[{"level": 5, "token": "8iervz","fben": "level_up_5"},{"level": 6, "token": "kzy8g4","fben": "level_up_6"}]','adjust',183),
('merge','[{"target": 3, "token": "s0wt24"}, {"target": 5, "token": "psxuom", "fben": "merge_5"}, {"target": -1, "token": "ff5gei"}]','adjust',183),
('start_order','{"token": "t2v3fd"}','adjust',183),
('end_order','{"token": "td028v"}','adjust',183),
('first_pay','{"token": "5btlo5", "fben": "first_pay"}','adjust',183),
('start_purchase','{"token": "i9su17","fben": "start_purchase"}','adjust',183),
('task','[{"id": "D2_SuperMerge", "token": "b8zvlb", "fben": "task_super_merge"}]','adjust',183),
('unlock_element','{"token": "v8z07c"}','adjust',183),
('unlock_fog','{"token": "ppb1rp"}','adjust',183),
('useitem','[{"type": "gem", "token": "bt8e0l", "fben": "use_gem"}, {"type": "gold", "token": "xhc0l5"}]','adjust',183),
('watch_ad','{"token": "3lsnpy","fben": "watch_ad"}','adjust',183),

UPDATE login_base_config set value='[{"type": "gem", "token": "bt8e0l", "fben": "use_gem"}, {"type": "gold", "token": "xhc0l5"}]' where

('chop','{"token": "8xg5xu"}','adjust',184),
('get_box','{"token": "c4u6kd"}','adjust',184),
('iap','[{"pay_id": "extraisland_1", "token": "tf4ts0" },{"pay_id": "getgem_1", "token": "lr29gz" },{"pay_id": "gift_energy_3", "token": "n66pxw" },{"pay_id": "gift_tb_hr", "token": "1pcvqx" },{"pay_id": "gift_welcome_1", "token": "sock5r" }]','adjust',184),
('levelup','[{"level": 5, "token": "d93id5","fben": "level_up_5"},{"level": 6, "token": "tn2zsu","fben": "level_up_6"}]','adjust',184),
('merge','[{"target": 3, "token": "woo8gu"}, {"target": 5, "token": "hu88o6", "fben": "merge_5"}, {"target": -1, "token": "vu3uwb"}]','adjust',184),
('start_order','{"token": "sg6pj2"}','adjust',184),
('end_order','{"token": "lbvv6b"}','adjust',184),
('first_pay','{"token": "1oxq9q", "fben": "first_pay"}','adjust',184),
('start_purchase','{"token": "pjicjx","fben": "start_purchase"}','adjust',184),
('task','[{"id": "D2_SuperMerge", "token": "wnllcy", "fben": "task_super_merge"}]','adjust',184),
('unlock_element','{"token": "84oz1v"}','adjust',184),
('unlock_fog','{"token": "wmwibq"}','adjust',184),
('useitem','[{"type": "gem", "token": "879up7", "fben": "use_gem"}, {"type": "gold", "token": "vyczh2"}]','adjust',184),
('watch_ad','{"token": "f5mvtu","fben": "watch_ad"}','adjust',184);


#升级user_base
user_base_2
CREATE TABLE op_common.user_base_2 (
    user_id bigint NOT NULL,
    install_date timestamp(6) without time zone,
    ip character varying(20),
    device_id character varying(100),
    vendor_id character varying(100),
    openudid character varying(100),
    idfa character varying(100),
    imei character varying(100),
    mfp_id character(32),
    last_login timestamp(6) without time zone,
    level integer,
    channel character varying(20),
    client_ver character varying(20),
    current_os_name character varying(10),
    install_os_name character varying(10),
    game_token character(6),
    network_name character varying(128),
    tracker_name character varying(256),
    adid character varying(64)
);
ALTER TABLE op_common.user_base_2 OWNER TO op_common;
ALTER TABLE ONLY op_common.user_base_2
    ADD CONSTRAINT user_base_user_id_game_token_key UNIQUE (user_id, game_token);

#导数据 Android iOS
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','token' from user_base where user_id=10101 and game_token='3a9d70';

#51bcd0
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT -user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','51bcd0' from user_base where app_id=101;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','51bcd0' from user_base where app_id=102;

#d7a935
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT -user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','d7a935' from user_base where app_id=103;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','d7a935' from user_base where app_id=104;

#633b2d
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','633b2d' from user_base where app_id=111;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','633b2d' from user_base where app_id=112;

#ba3005
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT -user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','ba3005' from user_base where app_id=113;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','ba3005' from user_base where app_id=114;

#31dfe6
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT -user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','3ad472' from user_base where app_id=143;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','3ad472' from user_base where app_id=144;

#7ee6c8
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT -user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'iOS','iOS','7ee6c8' from user_base where app_id=163;
INSERT INTO user_base_2(user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,current_os_name,install_os_name,game_token) 
SELECT user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,client_ver,'Android','Android','7ee6c8' from user_base where app_id=164;

alter table user_base rename to user_base_13;
alter table user_base_2 rename to user_base;

alter table user_base rename to user_base_2;
alter table user_base_13 rename to user_base;

INSERT INTO login_base_config_2(id,key,value,belong,app,cids,odd_even,check_manufacturer) 
SELECT id,key,value,belong,app,cids,-1,false from login_base_config;

alter table user_base rename to login_base_config_3;
alter table login_base_config_2 rename to login_base_config;
```

```shell
\copy op_common.user_base (user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,channel,client_ver,current_os_name,install_os_name,game_token) from '/root/test0518.csv' with (FORMAT csv,DELIMITER ',',escape '\',header false,quote '"',encoding 'UTF8')
\copy op_common.user_base (user_id,install_date,ip,device_id,vendor_id,openudid,idfa,imei,mfp_id,last_login,level,channel,client_ver,current_os_name,install_os_name,game_token) from '/home/yinghui.zhang/candy_all_users_exports.csv' with (FORMAT csv,DELIMITER ',',escape '\',header false,quote '"',encoding 'UTF8')
```

#### 5、2021/5/19 切换海外服务

##### 1）需求：

* 避免出现数据库报错
* 避免服务长时间停止
* 海外现有域名切换到新服务指向
* 修改表名

##### 2）操作：

1. 重传正确images

2. 切换nginx配置 去mf-ctrl 改端口 

3. 停容器

4. 改表名

   ```sql
   alter table user_base rename to user_base_3;
   alter table user_base_2 rename to user_base;
   alter table login_base_config rename to login_base_config_3;
   alter table login_base_config_2 rename to login_base_config;
   ```

5. 重新rundocker

##### 3)测试

```

```

#### 6、2021/06/02 candy配置更新

adjust配置：n_pay_2

```json
#ios
n_pay:'[{"times":2, "type":"equal", "token":"hfq4h2"},{"times":2,  "type":"over", "token":"iqv3mg"}]' 
#Android
n_pay:'[{"times":2, "type":"equal", "token":"5jd219"},{"times":2,  "type":"over", "token":"sme1q0"}]' 
sme1q0
```

```sql
#插入新配置时间
INSERT INTO login_base_config (key,value,belong,app)
values
('n_pay','[{"equal": 2, "token": "hfq4h2"},{"times": 2, "token": "iqv3mg"}]','adjust',183),
('n_pay','[{"equal": 2, "token": "5jd219"},{"times": 2, "token": "sme1q0"}]','adjust',184);
UPDATE  login_base_config SET value='[{"times":2, "type":"over", "token":"hfq4h2"},{"times":5,  "type":"equal", "token":"iqv3mg"}]'  where key='n_pay' and app=183;
UPDATE  login_base_config SET value='[{"times":2, "type":"over", "token":"5jd219"},{"times":5,  "type":"equal", "token":"sme1q0"}]'  where key='n_pay' and app=184;

UPDATE  login_base_config SET value='[{"times":2, "type":"over", "token":"hfq4h2","fben":"n_pay_over_2"},{"times":5,  "type":"equal", "token":"iqv3mg","fben":"n_pay_equal_5"}]'  where key='n_pay' and app=183;
UPDATE  login_base_config SET value='[{"times":2, "type":"over", "token":"5jd219","fben":"n_pay_over_2"},{"times":5,  "type":"equal", "token":"sme1q0","fben":"n_pay_equal_5"}]'  where key='n_pay' and app=184;
```

#### 7、2021/06/25 candy更新 DEBUG log配置

* 增加于adjust同级Debug log

```sql
INSERT INTO login_base_config (key,value,app)
values
('debug_log','1',183),
('debug_log','1',184);
```

