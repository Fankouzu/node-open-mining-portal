# NOMP ![NOMP Logo](http://zone117x.github.io/node-open-mining-portal/logo.svg "NOMP Logo")

#### Node开源挖矿门户

这个门户是一个极其高效、高度可扩展、一体化、易于设置的加密货币挖矿池，完全用Node.js编写。它包含一个stratum池服务器；奖励/支付/份额处理器；以及一个（*尚未完成的*）响应式用户友好的前端网站，提供挖矿说明、深入的实时统计和管理中心。

#### 生产使用注意事项

这是测试版软件。以下所有内容都可能会改变并破坏现有的NOMP设置：任何功能的功能、配置文件的结构和redis数据的结构。如果您在生产中使用此软件，请*不要*直接将新代码拉入生产使用，因为它可能会破坏您的设置，并要求您调整配置文件或redis数据等内容。

#### 付费解决方案

使用此软件需要系统管理、数据库管理、币种守护进程，有时还需要一些编程能力。运行一个生产池可能比全职工作还要繁重。

**向矿工支付BTC/LTC的币种切换和自动兑换**是一个很可能不会包含在这个项目中的功能。

#### 目录

* [特性](#特性)
  * [攻击缓解](#攻击缓解)
  * [安全性](#安全性)
  * [计划功能](#计划功能)
* [社区支持](#社区支持)
* [使用方法](#使用方法)
  * [要求](#要求)
  * [设置币种守护进程](#0-设置币种守护进程)
  * [下载安装](#1-下载安装)
  * [配置](#2-配置)
    * [门户配置](#门户配置)
    * [币种配置](#币种配置)
    * [池配置](#池配置)
    * [设置区块通知](#可选推荐-设置区块通知)
  * [启动门户](#3-启动门户)
  * [升级NOMP](#升级nomp)
* [捐赠](#捐赠)
* [致谢](#致谢)
* [许可证](#许可证)

### 特性

* 对于池服务器，它使用高效的[node-stratum-pool](//github.com/zone117x/node-stratum-pool)模块，支持vardiff、POW和POS、交易消息、反DDoS、IP封禁、[多种哈希算法](//github.com/zone117x/node-stratum-pool#hashing-algorithms-supported)。

* 该门户有一个[MPOS](//github.com/MPOS/php-mpos)兼容模式，可以作为[python-stratum-mining](//github.com/Crypto-Expert/stratum-mining)的即插即用替代品。这个模式可以在配置中启用，并将shares以MPOS期望的格式插入MySQL数据库。直接教程请参见wiki页面[为MPOS使用设置NOMP](//github.com/zone117x/node-open-mining-portal/wiki/Setting-up-NOMP-for-MPOS-usage)。

* 多池能力 - 这个软件从头开始设计，可以同时运行多个币种（可以有不同的属性和哈希算法）。它可以用来为单个币种或多个币种同时创建一个池。这些池使用集群来在多个CPU核心之间进行负载均衡。

* 对于奖励/支付处理，shares被插入Redis（一个快速的NoSQL键值存储）。使用PROP（比例）奖励系统和[Redis事务](http://redis.io/topics/transactions)进行安全和超快速的支付。对池运营者来说没有风险。来自导致孤立块的轮次的shares将被合并到当前轮次的share中，以便每一个share都能得到奖励。

* 这个门户没有用户账户/登录/注册。相反，矿工只需使用他们的币种地址进行stratum认证。一个极简的HTML5前端连接到门户的统计API，显示来自每个池的统计数据，如连接的矿工、网络/池难度/哈希率等。

* 使用币种网络和加密交易所API检测盈利能力的币种切换端口。矿工连接到这些端口时使用他们的公钥，NOMP使用它为任何需要支付的币种派生地址。

#### 攻击缓解

* 检测并阻止套接字泛洪（通过套接字发送垃圾数据以消耗系统资源）。
* 检测并阻止僵尸矿工（被僵尸网络感染的计算机连接到您的服务器以占用套接字但不发送任何shares）。
* 检测并阻止无效share攻击：
  * NOMP不容易受到其他池服务器发生的低难度share漏洞的影响。其他池服务器软件对新的哈希算法硬编码了估计的最大难度，而NOMP则根据币种源代码中找到的值动态生成每种算法的最大难度。
  * IP封禁功能，如果矿工提交超过可配置阈值的无效shares，将在可配置的时间内封禁该IP。
* NOMP用Node.js编写，使用单线程（异步）处理连接，而不是每个连接一个线程的开销，并实现了集群以利用所有CPU核心。

#### 安全性

NOMP为池运营者和矿工提供了一些隐含的安全优势：

* 没有注册/登录系统，不再需要担心不注重安全的矿工在多个池中重复使用密码。
* 默认自动支付，池利润被发送到另一个地址，因此池钱包不会充满硬币 - 给黑客很少的奖励，并使您的池不成为目标。
* 矿工可以注意到缺乏自动支付，作为运营者可能即将卷走他们的硬币的早期警告信号。

#### 计划功能

* NOMP API - 由网站用于在门户的前端网站上显示有关池的统计信息，并由NOMP桌面应用程序用于检索可用币种列表（以及用于本地钱包/地址生成的版本字节）。

* 为了减少刚起步的池的方差，计划了一个功能，允许您自己的池上游连接到更大的池服务器。它将从更大的池请求工作，然后将工作重新分配给我们自己的连接矿工。

### 社区支持

IRC

* 支持/一般讨论加入#nomp：<https://webchat.freenode.net/?channels=#nomp>
* 开发讨论加入#nomp-dev：<https://webchat.freenode.net/?channels=#nomp-dev>

加入我们的subreddit [/r/nomp](http://reddit.com/r/nomp)！

*由于某些模块依赖错误而无法运行门户？*这可能是因为您没有遵循本README中的说明。请__阅读使用说明__，包括[要求](#要求)和[下载安装](#1-下载安装)。如果您完全按照说明操作仍然遇到问题，请在github上开一个issue或加入我们的#nomp IRC频道并解释您的问题:)。

如果您的池使用NOMP，请告诉我们，我们会在这里列出您的网站。

##### 一些使用NOMP或node-stratum模块的池

* <http://clevermining.com>
* <http://suchpool.pw>
* <http://hashfaster.com>
* <http://miningpoolhub.com>
* <http://kryptochaos.com>
* <http://miningpools.tk>
* <http://umine.co.uk>

使用方法
=====

#### 要求

* 币种守护进程（找到币种的仓库并从源代码构建最新版本）
* [Node.js](http://nodejs.org/) v0.10+（[按照这些安装说明](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)）
* [Redis](http://redis.io/) 键值存储 v2.6+（[按照这些说明](http://redis.io/topics/quickstart)）

##### 严肃

这些都是合法的要求。如果您使用系统包管理器可能附带的旧版本Node.js或Redis，您将会遇到问题。按照链接的说明获取最新的稳定版本。

[**Redis安全警告**](http://redis.io/topics/security)：确保防火墙访问redis - 一个简单的方法是在您的`redis.conf`文件中包含`bind 127.0.0.1`。另外，了解和理解您正在使用的软件也是一个好主意 - 与redis相关的好的起点是[数据持久性](http://redis.io/topics/persistence)。
