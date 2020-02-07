# Valine Admin

## 简介
Valine Admin 项目是一个对 [Valine](https://valine.js.org) 评论系统的拓展应用，可增强 Valine 的邮件通知功能。基于 Leancloud 的云引擎与云函数，主要实现评论邮件通知、评论管理、自定义邮件通知模板等功能，而且还可以提供邮件 `通知站长` 和 `@ 通知` 的功能。

<a href="/高级配置.md#邮件通知展示" target="_black">点击查看演示</a>

## 部署
1. 需要确保 Valine 的基础功能是正常的，参考 [Valine Docs](https://valine.js.org)。
2. 进入 [Leancloud](https://leancloud.cn/dashboard/applist.html#/apps) 对应的 Valine 应用中。
3. 点击 `云引擎 -> 设置` 填写代码库：`https://github.com/hongweifuture/Valine-Admin`，保存
![代码库](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207142608.png)
4. 设置`自定义环境变量`，需要设置云引擎的环境变量以提供必要的信息，变量参数参考下面的`配置项`
![自定义环境变量](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207155409.png)
**配置项**

变量 | 示例 | 说明
--- | ------ |:-----
SITE_NAME | HONGWEI'S Blog | [必填] 网站名称
SITE_URL  | https://www.zhwei.cn| [必填] 网站地址，**最后不要加 `/`**
SMTP_SERVICE | QQ | [必填] 邮件服务提供商，支持 QQ、163、126、Gmail 以及 [更多](https://nodemailer.com/smtp/well-known/#supported-services)。 --- *如这里没有你使用的邮件提供商，请查看[自定义邮件服务器](/高级配置.md#自定义邮件服务器)*
SMTP_USER | xxxx@qq.com | [必填] SMTP登录用户，一般为邮箱地址
SMTP_PASS | xxxx | [必填] SMTP登录密码，一般为授权码，而不是邮箱的登陆密码，请自行查询对应邮件服务商的获取方式
SENDER_NAME | HONGWEI'S Blog Valine 评论提醒 | [可选] 发件人
ADMIN_URL | https://xxx.leanapp.cn/ | [建议] Web主机二级域名，用于自动唤醒
TO_EMAIL  | xxxxx@gmail.com | [可选] 指定站长收信邮箱，默认值为`SITE_USER`。用于 SMTP 发件人与站长收件人不一致的情况下使用。
TEMPLATE_NAME | rainbow | [可选] 通知邮件的模板（default和rainbow），参考高级功能

5. 点击 `云引擎 -> 部署`，选择`Git源码部署`，分支或版本号输入`master`，下载最新依赖（可选），部署
![Git](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207143035.png)
![master](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207143205.png)


## 后台管理
1. 点击 `云引擎 -> 设置`，在`Web主机域名`位置点击`申请`，获取二级域名，现在的二级域名不支持自定义，如果想好记请参考高级功能
![Web主机域名](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207143315.png)
2. 设置后台管理登录信息，点击 `存储 -> 结构化数据`，选择`_User -> 添加行`，只需要填写`password`、`username`、`email`这三个字段即可, 使用 email 作为账号登陆、password 作为账号密码、username 任意即可。（为了安全考虑，此 email 必须为配置中的 SMTP_USER 或 TO_EMAIL， 否则不允许登录）
![后台管理](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207151339.png)
3. 此后，可以通过`https://二级域名.leanapp.cn/`管理评论

## 定时任务
免费版的 LeanCloud 容器，是有强制性休眠策略的，不能 24 小时运行：
- 每天必须休眠 6 个小时
- 30 分钟内没有外部请求，则休眠
- 休眠后如果有新的外部请求实例则马上启动（但激活时此次发送邮件会失败）。


分析了一下上方的策略，如果不想付费的话，最佳使用方案就设置定时器，目前基于 LeanCloud 自带定时器实现了两种云函数定时任务：
- 自动唤醒，定时访问Web APP二级域名防止云引擎休眠（推荐）
- 定时检查，每天定时检查24小时内漏发的邮件通知


**配置**
1. 首先需要添加环境变量，点击 `云引擎 -> 设置`，配置`自定义环境变量`，变量名`ADMIN_URL`，变量值`Web 主机域名，即二级域名地址`，添加后重启容器环境变量才会生效
2. 配置定时任务，击 `云引擎 -> 定时任务`
- 配置自动唤醒（推荐），创建定时任务，名称任意，生产环境选择`self-wake`云函数，Cron表达式填入`0 */20 7-23 * * ?`，表示每天 7 - 23 点每 20 分钟访问一次，这样可以保持每天的绝大多数时间邮件服务是正常的。
- 配置定时检查，创建定时任务，名称任意，生产环境选择`resend-mails`云函数，Cron表达式填入`0 0 8 * * ?`，表示每天早8点检查过去24小时内漏发的通知邮件并补发
![](https://cdn.jsdelivr.net/gh/hongweifuture/jsDelivrCDN/img/20200207153831.png)

## 高级配置

[自定义邮件模板](/高级配置.md#自定义邮件模板)

[自定义收件邮箱](/高级配置.md#自定义收件邮箱)

[自定义邮件服务器](/高级配置.md#自定义邮件服务器)

[Web 评论管理](/高级配置.md#web-评论管理)

[好记的二级域名](/高级配置.md#好记的二级域名)

[Leancloud 休眠策略(必看)](/高级配置.md#leancloud-休眠策略)

[开发指南](/高级配置.md#开发)

## 更新历史

* 12.01 新增自助添加定时器方式。详见: [LeanCloud 自带定时器[推荐方式]](/高级配置.md#leancloud-自带定时器推荐)

* 7.30 修复 @ 邮件通知出错 bug (需 [Valine 1.3.0](https://valine.js.org/changelog.html#v1-3-0-2018-07-29) 支持)，优化发件逻辑，站长发的评论不再收到邮件通知。
* 7.7 兼容 `valine v1.2.0-beta ` 版本对 at 的更改 [点击查看](https://valine.js.org/changelog.html#v1-2-0-beta-2018-06-30)
* 7.1 修复 `Web` 后台登录安全 `bug`
* 6.14 添加自定义邮件服务器功能. [点击查看](/高级配置.md#自定义邮件服务器)

## 常见问题

### 为什么我收不到邮件？

* 请确认评论时留下的邮箱不是环境变量里的 `SMTP_USER` 或 `TO_EMAIL` 里的邮箱，原因详见 7.30 更新日志。
* 请确认修改环境变量后已重启容器。
* 对于 QQ / 网易 163 邮箱，请确认你输入的是 SMTP 的授权码，而不是登陆密码。[QQ邮箱获取授权码](https://service.mail.qq.com/cgi-bin/help?subtype=1&id=28&no=1001256)  [网易邮箱获取授权码](http://help.mail.163.com/faqDetail.do?code=d7a5dc8471cd0c0e8b4b8f4f8e49998b374173cfe9171305fa1ce630d7f67ac2cda80145a1742516)

### 为什么我刚开始测试的时候是正常的，但后面的邮件没有通知？

请确认已针对 `LeanCloud` 的**免费容器休眠策略**配置了定时器，详见：[LeanCloud 休眠策略](https://github.com/zhaojun1998/Valine-Admin/blob/master/%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE.md#leancloud-休眠策略)。

### 如何重启容器？

<img width="500" src="https://cdn.jun6.net/201807081507_968.png"/>

> **注: 更新新版本与更改环境变量均需要重启容器后生效。**


**注：本项目修改于 panjunwen 的项目 : [Valine-Admin](https://github.com/panjunwen/Valine-Admin) (部分逻辑于功能不同，还请读者不要搞混配置项.)**
