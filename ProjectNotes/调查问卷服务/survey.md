# Survey-调查问卷服务

### 一、概况

* 技术栈：Vue
* 项目简介：
  * 调查问卷页面部分，需与主控系统SurveyAction搭配使用。
  * 需要手动修改主控系统访问地址
  * 需要手动修改奖励显示内容
  * 需要手动增加问题内容
* 更新步骤：
  * 修改数据库配置（由主控系统读取）
    1. 添加问卷配置信息 wv_survey_info
    2. 配置用户id和tag set_user_collection
    3. 配置访问地址
  * 修改Survey项目
    1. 配置奖励页面
       1. reward.vue中修改显示内容（发奖在数据库配）
    2. 配置问卷（增加question.json 文件夹和文件）
    3. 修改请求url（线上、线下不一样）
* 项目运行流程
  1. 用户访问页面
  2. 根据访问参数记录用户信息
  3. 向game-control发送请求，验证用户是否有访问权限
  4. 用户答题
  5. 用户提交
  6. 向game-control发送提交请求
  7. 结束退出

### 二、笔记

#### 1、问卷配置

* 配置目录public/mock/51bcd0/[id]
* [id]要等与数据库中配置的order

#### 2、奖励配置

* 目录src/components/reward/Reward.vue
* 修改页面配置文件数量
* 方法暂未启用

#### 3、game-control 地址配置

* src/App.vue
* submit 和 getSurveyPageConfig

#### 4、项目结构

* App.vue:主文件
  * mouted()钩子方法初始化
    * BI日志
    * 获取参数配置
    * 请求调查问卷配置
  * template
    * MFMask模板显示loading信息
    * PrivacyWindow显示隐私信息
    * PoPWindow问卷窗口
      * 展示奖励
      * 动态模板从quetionPage中获取各个题目类型，并调用对应模板。
      * 点击他提交出发submit()方法
    * MFMASK显示奖励信息
      * 确定按钮点击触发->hiddenConfirm->rewardClose,发日志；->close()
  * submit()
    * 获取用户信息
    * post提交问卷信息
    * 根据返回值展示提示信息
  * getSurveyPageConfig()
    * 获取用户信息
    * post获取问卷配置
    * 获取问卷信息
    * 展示问卷页面
  * close()
    * 发送closeme
    * out设为true

