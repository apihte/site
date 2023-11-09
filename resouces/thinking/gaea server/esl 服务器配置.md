+++
title = 'esl 服务器配置.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# 现有的云服务器列表

| 服务器名称     | 内网 ip      | 外网 ip         | 用户名   | 备注                                                         |
| -------------- | ------------ | --------------- | -------- | ------------------------------------------------------------ |
| 逗屋提审       | -            | 180.76.55.195   | dong.xia | 用逗屋的版本做的国内提审服                                   |
| 打包服         | 192.168.64.7 | -               | dong.xia | 打包用服务器，主干和分支都在上面进行打包                     |
| 亚洲IOS提审    | 10.7.0.38    | 124.156.117.185 | dong.xia | 用于亚洲IOS提审，在客户端上新包前需要根据对应分支进行更新    |
| 亚洲google提审 | 10.7.0.99    | 129.226.188.235 | dong.xia | 用于亚洲google提审，在客户端上新包前需要根据对应分支进行更新 |
| 天梯测试服     | 10.7.0.109   |                 | dong.xia | 用于进行天梯改动的测试，主要是需要用线上数据测试             |
| 备用测试服     | 10.7.0.13    | 129.226.142.86  | dong.xia | 暂无明确用途                                                 |
| 分支时间服     | 192.168.64.8 | -               | dong.xia | 原本计划用作分支时间服，实际上使用很少                       |

# 现有的游戏服务配置
| 服务器名称 | serverid | servername  | doname                                 |
| ---------- | -------- | ----------- | -------------------------------------- |
| 个人服务器 | 1        | -           | -                                      |
| 主干       | 100      | esl-trunk   | https://esl-dev.gaeamobile-inc.net/    |
| 分支       | 150      | esl-branch  | https://esl-branch.gaeamobile-inc.net/ |
| 正式       | 200      | esl-prod    | https://esl-prod.burninggame.net/      |
| 安卓提审服 | 161      | anchor-test | http://129.226.188.235                 |
| ios提审服  | 162      | Ios-trial   | http://124.156.117.185                 |
| 正式服预留 | 200+     | -           | -                                      |

serverid 和 servername 是用来记录 BI 日志的，客户端会拿到这两个值，并将 BI 日志上传到平台，所以这里大家的 serverid 不要瞎改，以免和主干，测试服，或正式服日志产生混杂，产生不必要的后果。

个人用的服务器的 serverid 暂时都用 1 就行了，这个大家就不用自己改动了。

# 平台各种包对应的gameId：
| 平台名称    | gameid | serverid 和充值回调                                          |
| ----------- | ------ | ------------------------------------------------------------ |
| ios         | 710045 | 0	https://esl-prod.burninggame.net/shop/finishGaeaTransaction<br/>100	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>150	https://esl-branch.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>161	http://129.226.188.235:5035/shop/finishGaeaTransaction<br/>162	http://124.156.117.185:5035/shop/finishGaeaTransaction |
| mycard      | 722045 | 0	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>100	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>150	https://esl-branch.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>161	http://129.226.188.235:5035/shop/finishGaeaTransaction<br/>162	http://124.156.117.185:5035/shop/finishGaeaTransaction<br/>200	https://esl-prod.burninggame.net/shop/finishGaeaTransaction |
| 日韩 google | 720045 | 0	https://esl-prod.burninggame.net/shop/finishGaeaTransaction<br/>100	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>150	https://esl-branch.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>161	http://129.226.188.235:5035/shop/finishGaeaTransaction<br/>162	http://124.156.117.185:5035/shop/finishGaeaTransaction |
| 港澳台谷歌  | 721045 | 0	https://esl-prod.burninggame.net/shop/finishGaeaTransaction<br/>100	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>150	https://esl-branch.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>161	http://129.226.188.235:5035/shop/finishGaeaTransaction<br/>162	http://124.156.117.185:5035/shop/finishGaeaTransaction |
| 东南亚      | 723045 | 100	https://esl-dev.gaeamobile-inc.net/shop/finishGaeaTransaction<br/>200	https://esl-prod.burninggame.net/shop/finishGaeaTransaction |

ios：710045 

mycard（com.gaea.g.sgjzcq.mycard）：722045

日韩 google（com.gaea.g.sgjzcq）：720045

港澳台谷歌（com.gaea.g.sgjzcq.t）：721045

东南亚（com.gaea.g.sgjzcq.seapay）：723045

# 现有的k8s集群namespace
| 服务器 | namespace  | context   |
| ------ | ---------- | --------- |
| 主干服 | esl-trunk  | tx-hk-esl |
| 分支服 | esl-branch | tx-cn-esl |
| 正式服 | esl-prod   | tx-hk-esl |


由于新增集群需要大量的配置已经硬件资源，所以原则上不会再新增集群，除非是像国内上线这种。

# Docker仓库
esl-repo.gaeamobile-inc.net   

用户：admin   

密码：aon0n1L433

# 集群部分配置
mariadb:

mysql用户：root 

mysql密码：Dnr1H8Xrh!@#esl

# 集群日志查看地址。
Log Browser

Log Browser tx-cn-esl

用户： esl

密码：123456