# 一键部署


```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBASE及其他子系统的版本兼容说明！`请查看 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```

​	一键部署可以在 **同机** 快速搭建WeBASE管理台环境，方便用户快速体验WeBASE管理平台。

​	一键部署会搭建：节点（FISCO-BCOS 3.0+）、管理平台（WeBASE-Web）、节点管理子系统（WeBASE-Node-Manager）、节点前置子系统（WeBASE-Front）、签名服务（WeBASE-Sign）。其中，节点的搭建是可选的，可以通过配置来选择使用已有链或者搭建新链。一键部署架构如下：

<img src="../../_images/one_click_structure.png" width="700">


## 前提条件

| 环境   | 版本                   |
| ------ | ---------------------- |
| Java   | JDK 8 至JDK 14 |
| MySQL | MySQL-5.6及以上 |
| Python | Python3.6及以上 |
| PyMySQL | |

### 检查环境

#### 平台要求

推荐使用CentOS 7.2+, Ubuntu 16.04及以上版本, 一键部署脚本将自动安装`openssl, curl, wget, git, nginx, dos2unix`相关依赖项。

其余系统可能导致安装依赖失败，可自行安装`openssl, curl, wget, git, nginx, dos2unix`依赖项后重试

#### 检查Java

推荐JDK8-JDK13版本，使用OracleJDK[安装指引](#jdk)：

```
java -version
```

*注意：不要用`sudo`执行安装脚本*


#### 检查mysql

MySQL-5.6或以上版本：
```
mysql --version
```

- Mysql安装部署可参考[数据库部署](#mysql-install)

#### 检查Python
<span id="checkpy"></span>
使用Python3.6或以上版本：
```shell
python --version
# python3时
python3 --version
```

如已安装python3，也可通过`python3 --version`查看，在运行脚本时，使用`python3`命令即可

- Python3安装部署可参考[Python部署](#python3)

#### PyMySQL部署（Python3.6+）

Python3.6及以上版本，需安装`PyMySQL`依赖包

- CentOS

  ```
  sudo yum -y install python36-pip
  sudo pip3 install PyMySQL
  ```


- Ubuntu

  ```
  sudo apt-get install -y python3-pip
  sudo pip3 install PyMySQL
  ```

 CentOS或Ubuntu不支持pip命令的话，可以使用以下方式：

  ```
  git clone https://github.com/PyMySQL/PyMySQL
  cd PyMySQL/
  python3 setup.py install
  ```

### 检查服务器网络策略

网络策略检查：
- **开放WeBASE管理平台端口**：检查webase-web管理平台页面的端口`webPort`(默认为5000)在服务器的网络安全组中是否设置为**开放**。如，云服务厂商如腾讯云，查看安全组设置，为webase-web开放5000端口。**若端口未开放，将导致浏览器无法访问WeBASE服务页面**
- 开放节点前置端口：如果希望通过浏览器直接访问webase-front节点前置的页面，则需要开放节点前置端口`frontPort`（默认5002）；由于节点前置直连节点，**不建议对公网开放节点前置端口**，建议按需开放

## 拉取部署脚本

获取部署安装包：
```shell
wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v3.1.1/webase-deploy.zip
```
解压安装包：
```shell
unzip webase-deploy.zip
```
进入目录：
```shell
cd webase-deploy
```

## 修改配置

① mysql数据库需提前安装，已安装直接配置即可，还未安装请参看[数据库部署](#mysql)；

② 修改配置文件（`vi common.properties`）；

③ 一键部署支持使用已有链或者搭建新链。通过参数`if.exist.fisco`配置是否使用已有链，以下配置二选一即可：

- 当配置`yes`时，需配置已有链的sdk证书路径`sdk.dir`。路径下存放sdk证书，包含以下两种之一：

  非国密：`ca.crt`、`sdk.crt`、`sdk.key`

  国密：`sm_ca.crt`、`sm_sdk.crt`、`sm_sdk.key`、`sm_ensdk.crt`、`sm_ensdk.key`

- 当配置`no`时，需配置节点fisco版本和节点安装个数，未修改时搭建的新链默认两个群组

​    如果不使用一键部署搭建新链，可以参考FISCO BCOS官方文档搭建 [FISCO BCOS部署流程](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html#fisco-bcos)；

- 如果使用的`liquid`合约的链（`fisco.wasm=1`），并在WeBASE管理台或WeBASE-Front的合约IDE中编译Liquid合约，要求**手动**在WeBASE-Front所在主机[配置Liquid环境](https://liquid-doc.readthedocs.io/zh_CN/latest/docs/quickstart/prerequisite.html)后，才能通过WeBASE编译Liquid合约

- 启用链的权限管理功能时（`fisco.auth=1`），建链后会在`webase-deploy/nodes/ca`目录生成一个链管理员私钥（包含国密与非国密），需要在WeBASE管理台的权限管理页面中**导入该私钥**才能进行权限管理。


④ 服务端口不能小于1024

```shell
[common]
# WeBASE子系统的最新版本（v3.0版本）
webase.web.version=v3.1.1
webase.mgr.version=v3.1.1
webase.sign.version=v3.1.1
webase.front.version=v3.1.1

# 节点管理子系统mysql数据库配置
mysql.ip=localhost
mysql.port=3306
mysql.user=dbUsername
mysql.password=dbPassword
mysql.database=webasenodemanager

# 签名服务子系统mysql数据库配置
sign.mysql.ip=localhost
sign.mysql.port=3306
sign.mysql.user=dbUsername
sign.mysql.password=dbPassword
sign.mysql.database=webasesign

# 节点前置子系统h2数据库名
front.h2.name=webasefront

# WeBASE管理平台服务端口
web.port=5000
# 节点管理子系统服务端口
mgr.port=5001
# 节点前置子系统端口
front.port=5002
# 签名服务子系统端口
sign.port=5004

# SDK连接加密类型 （0: ECDSA SSL, 1: 国密SSL）
encrypt.type=0

# 是否使用已有的链（yes/no）
if.exist.fisco=no

# 搭建新链时需配置[if.exist.fisco=no]
# 节点监听Ip
node.listenIp=127.0.0.1
# 节点p2p端口
node.p2pPort=30300
# 节点rpc端口
node.rpcPort=20200
# FISCO-BCOS版本（v3.0.0或以上版本）
fisco.version=v3.5.0
# 搭建节点个数（默认两个）
node.counts=nodeCounts
# 是否搭建Liquid合约链（Solidity和Liquid合约需要二选一，默认Solidity
# 如果使用Liquid，要求在webase-front所在主机配置Liquid环境才能在WeBASE中编译合约
# [0: solidity, 1: liquid]
fisco.wasm=0
# 是否启用链的权限管理，FISCO BCOS >=3.3.0后默认启用
# 如果启用权限，一键部署自动在'webase-deploy/nodes/ca'目录生成一个随机的管理员私钥（包含国密与非国密）
# [0: disabled, 1: enable]
fisco.auth=1

# 使用已有链时需配置[if.exist.fisco=yes]
# 已有链节点rpc端口列表
node.rpcPeers=['127.0.0.1:20200','127.0.0.1:20201']
# sdk目录，需包含已有链SSL所需的证书（如: ca.crt,sdk.crt,sdk.key）
sdk.dir=/data/app/nodes/127.0.0.1/sdk
```


## 部署

* 执行installAll命令，部署服务将自动部署FISCO BCOS节点，并部署 WeBASE 中间件服务，包括签名服务（sign）、节点前置（front）、节点管理服务（node-mgr）、节点管理前端（web）

**备注：** 
- 部署脚本会拉取相关安装包进行部署，需保持网络畅通
- 首次部署需要下载编译包和初始化数据库，重复部署时可以根据提示不重复操作
- 部署过程中出现报错时，可根据错误提示进行操作，或根据本文档中的[常见问题](#q&a)进行排查
- **不要用sudo执行脚本**，例如`sudo python3 deploy.py installAll`（sudo会导致无法获取当前用户的环境变量如JAVA_HOME）

```shell
# 部署并启动所有服务
python3 deploy.py installAll
```

部署完成后可以看到`deploy  has completed`的日志：

```shell
$ python3 deploy.py installAll
...
============================================================
              _    _     ______  ___  _____ _____ 
             | |  | |    | ___ \/ _ \/  ___|  ___|
             | |  | | ___| |_/ / /_\ \ `--.| |__  
             | |/\| |/ _ | ___ |  _  |`--. |  __| 
             \  /\  |  __| |_/ | | | /\__/ | |___ 
              \/  \/ \___\____/\_| |_\____/\____/  
...
...
============================================================
==============      deploy  has completed     ==============
============================================================
==============    webase-web version  v...        ========
==============    webase-node-mgr version  v...   ========
==============    webase-sign version  v...       ========
==============    webase-front version  v...      ========
============================================================
```

* 服务部署后，需要对各服务进行启停操作，可以使用以下命令：

```shell
# 一键部署
部署并启动所有服务        python3 deploy.py installAll
停止一键部署的所有服务    python3 deploy.py stopAll
启动一键部署的所有服务    python3 deploy.py startAll
# 各子服务启停
启动FISCO-BCOS节点:      python3 deploy.py startNode
停止FISCO-BCOS节点:      python3 deploy.py stopNode
启动WeBASE-Web:          python3 deploy.py startWeb
停止WeBASE-Web:          python3 deploy.py stopWeb
启动WeBASE-Node-Manager: python3 deploy.py startManager
停止WeBASE-Node-Manager: python3 deploy.py stopManager
启动WeBASE-Sign:        python3 deploy.py startSign
停止WeBASE-Sign:        python3 deploy.py stopSign
启动WeBASE-Front:        python3 deploy.py startFront
停止WeBASE-Front:        python3 deploy.py stopFront
```


## 状态检查

成功部署后，可以根据以下步骤**确认各个子服务是否启动成功**

#### 检查各子系统进程

通过`ps`命令，检查各子系统的进程是否存在
- 包含：节点进程`nodeXX`，节点前置进程`webase.front`，节点管理服务进程`webase.node.mgr`，节点管理平台`webase-web`的`nginx`进程，以及签名服务进程`webase.sign`

检查方法如下，若无输出，则代表进程未启动，需要到该子系统的日志中[检查日志错误信息](#checklog)，并根据错误提示或本文档的[常见问题](#q&a)进行排查

- 检查节点进程，此处部署了两个节点node0, node1
```
$ ps -ef | grep node
```

输出如下
```
root     29977     1  1 17:24 pts/2    00:02:20 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node1/../fisco-bcos -c config.ini
root     29979     1  1 17:24 pts/2    00:02:23 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node0/../fisco-bcos -c config.ini
```

- 检查节点前置webase-front的进程
```
$ ps -ef | grep webase.front 
```

输出如下
```
root     31805     1  0 17:24 pts/2    00:01:30 /usr/local/jdk/bin/java -Djdk.tls.namedGroups=secp256k1 ... conf/:apps/*:lib/* com.webank.webase.front.Application
```

- 检查节点管理服务webase-node-manager的进程
```
$ ps -ef  | grep webase.node.mgr
```

输出如下
```
root      4696     1  0 17:26 pts/2    00:00:40 /usr/local/jdk/bin/java -Djdk.tls.namedGroups=secp256k1 ... conf/:apps/*:lib/* com.webank.webase.node.mgr.Application
```

- 检查webase-web对应的nginx进程
```
$ ps -ef | grep nginx       
```

输出如下
```
root      5141     1  0 Dec08 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /root/fisco/webase/webase-deploy/comm/nginx.conf
```

- 检查签名服务webase-sign的进程
```
$ ps -ef  | grep webase.sign 
```

输出如下
```
root     30718     1  0 17:24 pts/2    00:00:19 /usr/local/jdk/bin/java ... conf/:apps/*:lib/* com.webank.webase.sign.Application
```

#### 检查进程端口
通过`netstat`命令，检查各子系统进程的端口监听情况。

检查方法如下，若无输出，则代表进程端口监听异常，需要到该子系统的日志中[检查日志错误信息](#checklog)，并根据错误提示或本文档的[常见问题](#q&a)进行排查

- 检查节点channel端口(默认为20200)是否已监听
```
$ netstat -anlp | grep 20200
```
输出如下
```
tcp        0      0 0.0.0.0:20200           0.0.0.0:*               LISTEN      29069/fisco-bcos
```

- 检查webase-front端口(默认为5002)是否已监听
```
$ netstat -anlp | grep 5002
```
输出如下
```
tcp6       0      0 :::5002                 :::*                    LISTEN      2909/java 
```

- 检查webase-node-mgr端口(默认为5001)是否已监听
```
$ netstat -anlp | grep 5001    
```
输出如下
```
tcp6       0      0 :::5001                 :::*                    LISTEN      14049/java 
```

- 检查webase-web端口(默认为5000)在nginx是否已监听
```
$ netstat -anlp | grep 5000
```
输出如下
```
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      3498/nginx: master  
```

- 检查webase-sign端口(默认为5004)是否已监听
```
$ netstat -anlp | grep 5004
```

输出如下
```
tcp6       0      0 :::5004                 :::*                    LISTEN      25271/java 
```

<span id="checklog"></span>
#### 检查服务日志 

<span id="logpath"></span>
##### 各子服务的日志路径如下：

```
|-- webase-deploy # 一键部署目录
|--|-- log # 部署日志目录
|--|-- webase-web # 管理平台目录
|--|--|-- log # 管理平台日志目录
|--|-- webase-node-mgr # 节点管理服务目录
|--|--|-- log # 节点管理服务日志目录
|--|-- webase-sign # 签名服务目录
|--|--|-- log # 签名服务日志目录
|--|-- webase-front # 节点前置服务目录
|--|--|-- log # 节点前置服务日志目录
|--|-- nodes # 一件部署搭链节点目录
|--|--|-- 127.0.0.1
|--|--|--|-- node0 # 具体节点目录
|--|--|--|--|-- log # 节点日志目录
```
*备注：当前节点日志路径为一键部署搭链的路径，使用已有链请在相关路径查看日志*

日志目录中包含`{XXX}.log`全量日志文件和`{XXX}-error.log`错误日志文件
 - *通过日志定位错误问题时，可以结合`.log`全量日志和`-error.log`错误日志两种日志信息进行排查。*，如查询WeBASE-Front日志，则打开`WeBASE-Front-error.log`可以快速找到错误信息，根据错误查看`WeBASE-Front.log`的相关内容，可以看到错误日志前后的普通日志信息

##### 检查服务日志有无错误信息

- 如果各个子服务的进程**已启用**且端口**已监听**，可直接访问下一章节[访问WeBASE](#access)

- 如果上述检查步骤出现异常，如检查不到进程或端口监听，则需要按[日志路径](#logpath)进入**异常子服务**的日志目录，检查该服务的日志

- 如果检查步骤均无异常，但服务仍无法访问，可以分别检查部署日志`deployLog`，节点前置日志`frontLog`, 节点管理服务日志`nodeMgrLog`进行排查：
  - 检查webase-deploy/log中的**部署日志**，是否在部署时出现错误
  - 检查webase-deploy/webase-front/log中的**节点前置日志**，如果最后出现`application run success`字样则代表运行成功
  - 检查webase-deploy/webase-node-mgr/log或webase-deploy/webase-sign/log中的日志
  - 检查webase-deploy/nodes/127.0.0.1/nodeXXX/log中的节点日志


##### 搜索日志

通过查看日志可以检查服务的运行状态，我们可以进入各子服务的日志路径，通过`grep`检查日志文件，以此判断服务是否正常运行

- **查看运行成功日志**：WeBASE子服务运行成功后均会打印日志`main run success`，可以通过搜索此关键字来确认服务正常运行。

如，检查webase-front日志，其他WeBASE服务可进行类似操作
```
$ cd webase-front
$ grep -B 3 "main run success" log/WeBASE-Front.log
```
输出如下：
```
2020-12-09 15:47:25.355 [main] INFO  ScheduledAnnotationBeanPostProcessor() - No TaskScheduler/ScheduledExecutorService bean found for scheduled processing
2020-12-09 15:47:25.378 [main] INFO  TomcatEmbeddedServletContainer() - Tomcat started on port(s): 5002 (http)
2020-12-09 15:47:25.383 [main] INFO  Application() - Started Application in 6.983 seconds (JVM running for 7.768)
2020-12-09 15:47:25.383 [main] INFO  Application() - main run success...
```

- **查看报错日志**：出现异常时，可以搜索关键字`ERROR`进行检查

如，检查webase-front错误日志，其他WeBASE服务可进行类似操作
```
$ cd webase-front
$ grep "ERROR" log/WeBASE-Front.log
```
输出如下
```
2020-12-09 09:10:42.138 [http-nio-5002-exec-1] ERROR ExceptionsHandler() - catch frontException:  no active connection available network exception requset send failed! please check the log file content for reasons.
2020-12-09 09:10:42.145 [http-nio-5002-exec-4] ERROR Web3ApiService() - getBlockNumber fail.
```

如果出现错误日志，根据错误提示或本文档的[常见问题](#q&a)进行排查

启动失败或无法使用时，欢迎到WeBASE[提交Issue](https://github.com/WeBankBlockchain/WeBASE/issues)或到技术社区共同探讨。
- 提交Issue或讨论问题时，可以在issue中配上自己的**环境配置，操作步骤，错误现象，错误日志**等信息，方便社区用户快速定位问题


<span id="access"></span>
## 访问

WeBASE管理平台：

* 一键部署完成后，**打开浏览器（Chrome Safari或Firefox）访问**
```
http://{deployIP}:{webPort}
示例：http://localhost:5000
```

**备注：** 

- 部署服务器IP和管理平台服务端口需对应修改，网络策略需开通
  - 使用云服务厂商的服务器时，需要开通网络安全组的对应端口。如开放webase使用的5000端口
- WeBASE管理平台使用说明请查看[使用手册](../WeBASE-Console-Suit/index.html#id13)（获取WeBASE管理平台默认账号和密码，并初始化系统配置）
  - 默认账号为`admin`，默认密码为`Abcd1234`。首次登陆要求重置密码
  - 添加节点前置WeBASE-Front到WeBASE管理平台；一键部署时，节点前置与节点管理服务默认是同机部署，添加前置则填写IP为`127.0.0.1`，默认端口为`5002`。参考上文中`common.properties`的配置项`front.port={frontPort}`
- 检查节点前置是否启动，可以通过访问`http://{frontIp}:{frontPort}/WeBASE-Front`(默认端口5002)；访问前，确保服务端已对本地机器开放端口，如开放front的5002端口。（不建议节点前置的端口对公网开放访问权限，应对部分机器IP按需开放）


## 附录

<span id="jdk"></span>
### 1. Java环境部署

#### CentOS环境安装Java
<span id="centosjava"></span>

**注意：CentOS下OpenJDK无法正常工作，需要安装OracleJDK[下载链接](https://www.oracle.com/technetwork/java/javase/downloads/index.html)。**

```
# 创建新的文件夹，安装Java 8或以上的版本，推荐JDK8-JDK13版本，将下载的jdk放在software目录
# 从Oracle官网(https://www.oracle.com/java/technologies/downloads/#java8)选择Java 8或以上的版本下载，例如下载jdk-8u301-linux-x64.tar.gz
$ mkdir /software

# 解压jdk
$ tar -zxvf jdk-8u301-linux-x64.tar.gz

#修改解压后文件的文件名
$mv jdk1.8.0_301 jdk-8u301

# 配置Java环境，编辑/etc/profile文件
$ vim /etc/profile

# 打开以后将下面三句输入到文件里面并保存退出
export JAVA_HOME=/software/jdk-8u301  #这是一个文件目录，非文件
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 生效profile
$ source /etc/profile

# 查询Java版本，出现的版本是自己下载的版本，则安装成功。
java -version
```

#### Ubuntu环境安装Java
<span id="ubuntujava"></span>

```
  # 安装默认Java版本(Java 8或以上)
  sudo apt install -y default-jdk
  # 查询Java版本
  java -version
```

### 2. 数据库部署
<span id="mysql-install"></span>

#### ① CentOS安装MariaDB

此处以**CentOS 7(x86_64)**安装**MariaDB 10.2**为例。*MariaDB*数据库是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可。*MariaDB*完全兼容 MySQL，包括API和命令行。MariaDB 10.2版本对应Mysql 5.7。其他安装方式请参考[MySQL官网](https://dev.mysql.com/downloads/mysql/)。
- CentOS 7 默认MariaDB为5.5版本，安装10.2版本需要按下文进行10.2版本的配置。
- 若使用CentOS 8则直接使用`sudo yum install -y mariadb*`即可安装MariaDB 10.3，并跳到下文的 *启停* 章节即可。

使用`vi`或`vim`创建新文件`/etc/yum.repos.d/mariadb.repo`，并写入下文的文件内容（参考[MariaDB中科大镜像源修改](http://mirrors.ustc.edu.cn/help/mariadb.html)进行配置）

- 创建repo文件
```Bash
sudo vi /etc/yum.repos.d/mariadb.repo
```

- 文件内容，此处使用的是中科大镜像源
```Bash
# MariaDB 10.2 CentOS repository list - created 2021-07-12 07:37 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = https://mirrors.ustc.edu.cn/mariadb/yum/10.2/centos7-amd64
gpgkey=https://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

- 更新yum源缓存数据
```
yum clean all
yum makecache all
```

- 安装`MariaDB 10.2`
- 如果已存在使用`sudo yum install -y mariadb*`命令安装的MariaDB，其版本默认为5.5版本，对应Mysql版本为5.5。新版本MariaDB无法兼容升级，需要先**卸载旧版本**的MariaDB，卸载前需要**备份**数据库内容，卸载命令可参考`yum remove mariadb`
```
sudo yum install MariaDB-server MariaDB-client -y
```

若安装时遇到错误`“Failed to connect to 2001:da8:d800:95::110: Network is unreachable”`，将源地址中的 `mirrors.ustc.edu.cn` 替换为 `ipv4.mirrors.ustc.edu.cn` 以强制使用 IPv4：
```
sudo sed -i 's#//mirrors.ustc.edu.cn#//ipv4.mirrors.ustc.edu.cn#g' /etc/yum.repos.d/mariadb
```
详情参考[MariaDB官网安装](https://downloads.mariadb.org/mariadb/repositories/#mirror=digitalocean-nyc)。
- 启停

```shell
启动：sudo systemctl start mariadb.service
停止：sudo systemctl stop  mariadb.service
```

- 设置开机启动

```
sudo systemctl enable mariadb.service
```

- 初始化

```shell
执行以下命令：
sudo mysql_secure_installation
以下根据提示输入：
Enter current password for root (enter for none):<–初次运行直接回车
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
New password: <– 设置root用户的密码
Re-enter new password: <– 再输入一次你设置的密码
Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
Disallow root login remotely? [Y/n] <–是否禁止root远程登录，回车
Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车
```

#### ② 授权访问和添加用户

- 使用root用户登录，密码为初始化设置的密码

```
mysql -uroot -p -h localhost -P 3306
```

- 授权root用户远程访问

```sql
mysql > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql > flush PRIVILEGES;
```

- 创建test用户并授权本地访问

```sql
mysql > GRANT ALL PRIVILEGES ON *.* TO 'test'@localhost IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql > flush PRIVILEGES;
```

**安全温馨提示：**

- 例子中给出的数据库密码（123456）仅为样例，强烈建议设置成复杂密码
- 例子中root用户的远程授权设置会使数据库在所有网络上都可以访问，请按具体的网络拓扑和权限控制情况，设置网络和权限帐号

#### ③ 测试连接和创建数据库

- 登录数据库

```shell
mysql -utest -p123456 -h localhost -P 3306
```

- 创建数据库

```sql
mysql > create database webasenodemanager;
```

#### ④ Ubuntu安装mysql数据库

- 以root用户执行命令

```
apt-get install software-properties-common
sudo add-apt-repository 'deb http://archive.ubuntu.com/ubuntu trusty universe'
sudo apt-get update
sudo apt install mysql-server-5.6
sudo apt install mysql-client-5.6
```
- 执行mysql --version命令，若显示如下则安装成功
```
mysql  Ver 14.14 Distrib 5.6.16, for debian-linux-gnu (x86_64) using  EditLine wrapper
```

<span id="python3"></span>
### 3. Python部署

python版本要求使用python3.x, 推荐使用python3.6及以上版本

- CentOS

  ```
  sudo yum install -y python36
  sudo yum install -y python36-pip
  ```

- Ubuntu

  ```
  // 添加仓库，回车继续
  sudo add-apt-repository ppa:deadsnakes/ppa
  // 安装python 3.6
  sudo apt-get install -y python3.6
  sudo apt-get install -y python3-pip
  ```

<span id="q&a"></span>
## 常见问题

### 1. Python命令出错

- SyntaxError报错
```
  File "deploy.py", line 62
    print helpMsg
                ^
SyntaxError: Missing parentheses in call to "print". Did you mean print(helpMsg)?
```

- 找不到fallback关键字
```
File "/home/ubuntu/webase-deploy/comm/utils.py", line 127, in getCommProperties
    value = cf.get('common', paramsKey,fallback=None)
TypeError: get() got an unexpected keyword argument 'fallback'
```

答：检查[Python版本](#checkpy)，推荐使用python3.6及以上版本


### 2. 使用Python3时找不到pymysql

```
Traceback (most recent call last):
...
ImportError: No module named 'pymysql'
```

答：需要安装PyMySQL，安装请参看 [pymysql](#pymysql-python3-5)

### 3. 部署时某个组件失败，重新部署提示端口被占用问题

答：因为有个别组件是启动成功的，需先执行“python deploy.py stopAll”将其停止，再执行“python deploy.py installAll”部署全部。

### 4. 管理平台启动时Nginx报错

```
...
==============      WeBASE-Web      start...  ==============
Traceback (most recent call last):
...
Exception: execute cmd  error ,cmd : sudo /usr/local/nginx/sbin/nginx -c /data/app/webase-deploy/comm/nginx.conf, status is 256 ,output is nginx: [emerg] open() "/etc/nginx/mime.types" failed (2: No such file or directory) in /data/app/webase-deploy/comm/nginx.conf:13
```

答：缺少/etc/nginx/mime.types文件，建议重装nginx。

### 5. 部署时数据库访问报错

```
...
checking database connection
Traceback (most recent call last):
  File "/data/temp/webase-deploy/comm/mysql.py", line 21, in dbConnect
    conn = mdb.connect(host=mysql_ip, port=mysql_port, user=mysql_user, passwd=mysql_password, charset='utf8')
  File "/usr/lib64/python2.7/site-packages/MySQLdb/__init__.py", line 81, in Connect
    return Connection(*args, **kwargs)
  File "/usr/lib64/python2.7/site-packages/MySQLdb/connections.py", line 193, in __init__
    super(Connection, self).__init__(*args, **kwargs2)
OperationalError: (1045, "Access denied for user 'root'@'localhost' (using password: YES)")
```

答：确认数据库用户名和密码

### 6. 节点sdk目录不存在

```
...
======= FISCO-BCOS sdk dir:/data/app/nodes/127.0.0.1/sdk is not exist. please check! =======
```

答：确认节点安装目录下有没有sdk目录（企业部署工具搭建的链可能没有），如果没有，需手动创建"mkdir sdk"，并将节点sdk证书（ca.crt、sdk.key、sdk.crt）复制到该sdk目录，再重新部署。如果使用国密SSL，则在sdk目录里存放国密sdk证书（sm_ca.crt、sm_sdk.crt、sm_sdk.key、sm_ensdk.crt、sm_ensdk.key）。

### 7. 前置启动报错“nested exception is javax.net.ssl.SSLException”

```
...
nested exception is javax.net.ssl.SSLException: Failed to initialize the client-side SSLContext: Input stream not contain valid certificates.
```

答：CentOS的yum仓库的OpenJDK缺少JCE(Java Cryptography Extension)，导致Web3SDK/Java-SDK无法正常连接区块链节点，因此在使用CentOS操作系统时，推荐使用[OracleJDK](#jdk)。


### 8.前置启动报错“Processing bcos message timeout”

```
...
[main] ERROR SpringApplication() - Application startup failed
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'contractController': Unsatisfied dependency expressed through field 'contractService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'contractService': Unsatisfied dependency expressed through field 'web3jMap'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'web3j' defined in class path resource [com/webank/webase/front/config/Web3Config.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [java.util.HashMap]: Factory method 'web3j' threw exception; nested exception is java.io.IOException: Processing bcos message timeout
...
```

答：一些OpenJDK版本缺少相关包，导致节点连接异常。推荐使用[OracleJDK](#jdk)。

### 9. 服务进程起来了，服务不正常

```
...
======= WeBASE-Node-Manager  starting . Please check through the log file (default path:./webase-node-mgr/log/). =======
```

答：查看日志，确认问题原因。确认后修改重启，如果重启提示服务进程在运行，先执行“python deploy.py stopAll”将其停止，再执行“python deploy.py startAll”重启。

### 10. WeBASE-Web登录页面的验证码加载不出来

答：检查WeBASE-Node-Manager后台服务是否已启动成功。若启动成功，检查后台日志：

* 进入 `webase-node-mgr` 目录下，执行 `bash status.sh` 检查服务是否启动，如果服务没有启动，运行 `bash start.sh` 启动服务；

* 如果服务已经启动，按照如下修改日志级别
    * `webase-node-mgr/conf/application.yml`
    
    ```
    #log config
    logging:
      level:
        com.webank.webase.node.mgr: debug
    ```
    
    * `webase-node-mgr/conf/log/log4j2.xml`

    ```
    <Loggers>
    <Root level="debug">
      <AppenderRef ref="asyncInfo"/>
      <AppenderRef ref="asyncErrorLog"/>
    </Root>
  </Loggers>
  ```

* 修改日志level后，重启服务 `bash stop.sh && bash start.sh`

* 重启服务后，检查日志文件 `log/WeBASE-Node-Manager.log`。
  
    * 检查是否有异常信息。如果有异常信息，根据具体的异常信息检查环境配置，或者通过搜索引擎进行排查。

### 11. WeBASE 国内镜像与CDN加速服务

答：WeBASE CDN 加速服务提供 WeBASE 各子系统安装包的下载服务，可参考[国内镜像和CDN加速攻略](./mirror.html)


*欢迎给WeBASE的文档提交 Pull Request 补充更多的 Q&A*
