# 一键Docker部署

​	一键部署（Docker模式）可以在 **同机** 快速搭建WeBASE管理台环境，方便用户快速体验WeBASE管理平台。

​	一键部署（Docker模式）会搭建：节点（FISCO-BCOS 2.x，暂未支持FISCO-BCOS 3.x）、管理平台（WeBASE-Web）、节点管理子系统（WeBASE-Node-Manager）、节点前置子系统（WeBASE-Front）、签名服务（WeBASE-Sign）。其中，节点的搭建是可选的，可以通过配置来选择使用已有链或者搭建新链。一键部署架构如下：


```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBASE及其他子系统的兼容版本说明！`请查看 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```

<img src="../../_images/one_click_structure.png" width="700">


## 前提条件

| 环境   | 版本                   |
| ------ | ---------------------- |
| Docker | 20.10.0及以上 |
| Docker-Compose | 1.29.2及以上 |
| Python | Python3.6及以上 |
| PyMySQL | |
| MySQL | MySQL-5.6及以上 （可选） |

### 检查环境

#### 平台要求

推荐使用CentOS 7.2+, Ubuntu 16.04, MacOS 10.2.x 及以上版本, 一键部署（Docker模式）脚本依赖Docker与Docker-Compose进行容器的编排，将自动安装`openssl, curl, wget, git, dos2unix`等相关依赖项。

其余系统可能导致安装依赖失败，可自行安装`openssl, curl, wget, git, dos2unix`依赖项后重试

*由于WeBASE Docker镜像中自带Java环境，无需在宿主机中配置Java环境；同时支持使用Docker启动一个新的mysql服务*

*如果在MacOS中进行配置*，需要按照[Mac配置sed指令](#mac_sed)完成配置后，才能使用一键部署

#### 检查Docker

Docker 20.10.0及以上版本，如需安装，参考[Docker安装](#install_docker)
```
$ docker --version
Docker version 20.10.0, build 7287ab3
```

**注意**：确保Docker免sudo执行，参考[Docker用户组配置](#docker_sudo)

#### 配置Docker国内镜像源

由于部分网络直接访问DockerHub官方镜像源拉取镜像的速度较慢，为提高部署的成功率，需要配置Docker的镜像源为国内的镜像源。

##### 查看镜像源配置
```Bash
cat /etc/docker/daemon.json

{}
```
若提示“目录不存在”、“该文件不存在”或“文件内容为空”属于正常现象，则说明未配置过Docker镜像源

##### 新建/修改Docker镜像源配置
以中科大的镜像源为例（若提示权限不足(Permission Denied)，则在命令前加上sudo）
```Bash
# 若目录不存在
mkdir -p /etc/docker
# 创建/修改daemon.json配置文件
vi /etc/docker/daemon.json

# 配置内容如下：
{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

##### 重新加载配置文件，重启docker服务
```Bash
systemctl daemon-reload
systemctl restart docker.service
```

#### 检查Docker-Compose

Docker-Compose 1.29.2 及以上版本，如需安装，参考[Docker-Compose安装](#docker-compose-install)

```
$ docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

**注意**：确保Docker免sudo执行，参考[Docker用户组配置](#docker_sudo)


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
- MacOS

  ```
  sudo brew install python3
  sudo pip3 install PyMySQL
  ```


 若CentOS/Ubuntu/MacOS不支持pip命令的话，可以使用以下方式：

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
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-deploy.zip

# 网络访问失败，则可以尝试直接git clone WeBASE的仓库
# 其中deploy目录即webase-deploy的目录
git clone -b master-3.0 https://github.com/WeBankBlockchain/WeBASE.git
# 若因网络问题导致长时间下载失败，可尝试以下命令
git clone -b master-3.0 https://gitee.com/WeBank/WeBASE.git

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

① 若未安装mysql，可在配置文件中启用`docker.mysql=1`，并配置Docker中Mysql端口与密码，**使用Docker启动Mysql**。若使用已安装的Mysql则在配置文件中webase-node-mgr和webase-sign填入对应配置；

② 修改配置文件（`vi common.properties`）；

③ 一键部署支持使用已有链或者搭建新链。通过参数"if.exist.fisco"配置是否使用已有链，以下配置二选一即可：

- 当配置"yes"时，需配置已有链的路径`fisco.dir`。路径下要存在sdk目录，sdk目录中包含ca.crt, sdk.crt, sdk.key及gm目录，gm目录中包含国密SSL所需证书，包含gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt和gmensdk.key
- 当配置"no"时，需配置节点fisco版本和节点安装个数，搭建的新链默认两个群组

​    如果不使用一键部署搭建新链，可以参考FISCO BCOS官方文档搭建 [FISCO BCOS部署流程](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html#fisco-bcos)；

​    **注：使用国密版需要修改设置配置项`encrypt.type=1`。前置SDK与节点默认使用非国密SSL，如果需要使用国密SSL，需要修改设置配置项`encrypt.sslType=1`。**

④ 服务端口不能小于1024

⑤ 部署时，修改 `common.properties` 配置文件

```shell
# WeBASE子系统的最新版本(v1.1.0或以上版本)
webase.web.version=v1.5.5
webase.mgr.version=v1.5.5
webase.sign.version=v1.5.5
webase.front.version=v1.5.5

#####################################################################
## 使用Docker启用Mysql服务，则需要配置以下值

# 1: enable mysql in docker
# 0: mysql run in host, required fill in the configuration of webase-node-mgr and webase-sign
docker.mysql=1

# if [docker.mysql=1], mysql run in host (only works in [installDockerAll])
# run mysql 5.6 by docker
docker.mysql.port=23306
# default user [root]
docker.mysql.password=123456

#####################################################################
## 不使用Docker启动Mysql，则需要配置以下值

# 节点管理子系统mysql数据库配置
mysql.ip=127.0.0.1
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


# 节点前置子系统h2数据库名和所属机构
front.h2.name=webasefront
front.org=fisco

# WeBASE管理平台服务端口
web.port=5000

# 节点管理子系统服务端口
mgr.port=5001
# 节点前置子系统端口
front.port=5002
# 签名服务子系统端口
sign.port=5004


# 节点监听Ip
node.listenIp=127.0.0.1
# 节点p2p端口
node.p2pPort=30300
# 节点链上链下端口
node.channelPort=20200
# 节点rpc端口
node.rpcPort=8545

# 加密类型 (0: ECDSA算法, 1: 国密算法)
encrypt.type=0
# SSL连接加密类型 (0: ECDSA SSL, 1: 国密SSL)
# 只有国密链才能使用国密SSL
encrypt.sslType=0

# 是否使用已有的链（yes/no）
if.exist.fisco=no

# 使用已有链时需配置
# 已有链的路径，start_all.sh脚本所在路径
# 路径下要存在sdk目录（sdk目录中包含了SSL所需的证书，即ca.crt、sdk.crt、sdk.key和gm目录（包含国密SSL证书，gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt和gmensdk.key）
fisco.dir=/data/app/nodes/127.0.0.1
# 前置所连接节点，在127.0.0.1目录中的节点中的一个
# 节点路径下要存在conf文件夹，conf里存放节点证书（ca.crt、node.crt和node.key）
node.dir=node0

# 搭建新链时需配置
# FISCO-BCOS版本
fisco.version=v2.11.0
# 搭建节点个数（默认两个）
node.counts=nodeCounts
```


## 拉取镜像

* 在上文已配置Docker镜像源为国内镜像源后，我们执行`pullDockerAll`命令，部署服务将拉取所需的Docker镜像，包括 `fiscoorg/fiscobcos, webasepro/webase-front, webasepro/webase-node-mgr, webasepro/webase-sign, webasepro/webase-web`，并根据配置确认是否拉取 mysql:5.6 的数据库镜像

**备注：**
- 请确认已配置Docker镜像源为国内镜像源，以提高拉取镜像的速度。可通过`cat /etc/docker/daemon.json`进行查看
- 拉取镜像开始前需要输入一个**拉取超时时间**，如60，即60s拉取未完成则提示超时
- 超时拉取的镜像，可通过`docker pull`进行手动拉取，如手动拉取webase-front v1.5.3的镜像为`docker pull webasepro/webase-front:v1.5.3`

```Bash
# 拉取时，可输入拉取超时时间，默认为60s
$ python3 deploy.py pullDockerAll
```

## 部署

* 执行`installDockerAll`命令，部署服务将**使用Docker**自动部署并启动 FISCO BCOS节点 与 WeBASE 中间件服务，包括签名服务（sign）、节点前置（front）、节点管理服务（node-mgr）、节点管理前端（web）

**备注：** 
- 部署脚本会拉取相关Docker镜像进行部署，需保持网络畅通
- 首次部署需要初始化数据库，重复部署时可以根据提示不重复操作
- 部署过程中出现报错时，可根据错误提示进行操作，或根据本文档中的[常见问题](#q&a)进行排查
- **不要用sudo执行脚本**，例如`sudo python3 deploy.py installDockerAll`（sudo会导致无法获取当前用户的环境变量如JAVA_HOME）
- 确保**已安装Docker与Docker-Compose、配置Docker国内镜像源并配置Docker用户组**

```shell
# 部署并启动所有服务（重新安装时需要先停止服务再重新安装，避免端口占用）
$ python3 deploy.py installDockerAll
```

部署完成后可以看到`deploy has completed`的日志：

```shell
$ python3 deploy.py installDockerAll
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
==============    webase-web version  v1.5.3        ========
==============    webase-node-mgr version  v1.5.3   ========
==============    webase-sign version  v1.5.3       ========
==============    webase-front version  v1.5.3      ========
============================================================
```

* 服务启动后，通过`docker-compose -f docker/docker-compose.yaml logs -f`命令查看Docker-Compose运行日志（不同容器会以不同颜色的日志打印）
```Bash
# 可通过Ctrl + C 取消日志打印
$ docker-compose -f docker/docker-compose.yaml logs -f

...
webase-front-5002  | INFO: Starting ProtocolHandler ["http-nio-5002"]
webase-front-5002  | Sep 06, 2021 3:18:23 PM org.apache.tomcat.util.net.NioSelectorPool getSharedSelector
webase-front-5002  | INFO: Using a shared selector for servlet write/read
mysql-webase-23306 exited with code 0
webase-web-5000    | wait-for-it.sh: waiting 30 seconds for 127.0.0.1:5001
webase-web-5000    | wait-for-it.sh: timeout occurred after waiting 30 seconds for 127.0.0.1:5001
webase-web-5000    | start webase-web now...
```


* 服务部署后，需要对各服务进行启停操作，可以使用以下命令：

```shell
# 一键部署
部署并启动所有服务        python3 deploy.py installDockerAll
停止一键部署的所有服务    python3 deploy.py stopDockerAll
启动一键部署的所有服务    python3 deploy.py startDockerAll
# 节点的启停
启动所有FISCO-BCOS节点:      python3 deploy.py startNode
停止所有FISCO-BCOS节点:      python3 deploy.py stopNode
# WeBASE服务的启停
启动所有WeBASE服务:      python3 deploy.py dockerStart
停止所有WeBASE服务:      python3 deploy.py dockerStop
```


## 状态检查

成功部署后，可以根据以下步骤**确认各个子服务是否启动成功**

#### 检查各子系统容器

通过`docker ps`命令，检查各子系统的容器是否存在
- 包含：节点容器`fiscobcos`，节点前置容器`webase-front`，节点管理服务容器`webase-node-mgr`，节点管理平台容器`webase-web`，以及签名服务容器`webase-sign`

检查方法如下，若无输出，则代表容器未启动，需要到该子系统的日志中[检查日志错误信息](#checklog)，并根据错误提示或本文档的[常见问题](#q&a)进行排查

- 检查节点容器，此处部署了两个节点node0, node1
```
$ docker ps | grep fiscobcos
```

输出如下
```
8fc863019565   fiscoorg/fiscobcos:v2.7.2          "/usr/local/bin/fisc…"   About a minute ago   Up About a minute             datahomedeploynodes127.0.0.1node1
87978ae7050c   fiscoorg/fiscobcos:v2.7.2          "/usr/local/bin/fisc…"   About a minute ago   Up About a minute             datahomedeploynodes127.0.0.1node0
```

- 检查节点前置webase-front的容器
```
$ docker ps | grep webase-front 
```

输出如下
```
26a131040e58   webasepro/webase-front:v1.5.3      "/wait-for-it.sh 127…"   37 seconds ago   Up 36 seconds             webase-front-5002
```

- 检查节点管理服务webase-node-manager的容器
```
$ docker ps  | grep webase-node.mgr
```

输出如下
```
cc6bbce73a85   webasepro/webase-node-mgr:v1.5.3   "/wait-for-it.sh 127…"   37 seconds ago   Up 36 seconds             webase-node-mgr-5001
```

- 检查webase-web对应的nginx容器
```
$ docker ps | grep webase-web       
```

输出如下
```
e056eca7ffa5   webasepro/webase-web:v1.5.3        "/wait-for-it.sh 127…"   2 minutes ago   Up 2 minutes             webase-web-5000
```

- 检查签名服务webase-sign的容器
```
$ docker ps  | grep webase-sign 
```

输出如下
```
49d0650ae904   webasepro/webase-sign:v1.5.3       "/wait-for-it.sh 127…"   37 seconds ago   Up 37 seconds             webase-sign-5004
```

#### 检查容器端口
通过`netstat`命令，检查各子系统容器的端口监听情况。

检查方法如下，若无输出，则代表容器端口监听异常，需要到该子系统的日志中[检查日志错误信息](#checklog)，并根据错误提示或本文档的[常见问题](#q&a)进行排查

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

- 如果各个子服务的容器**已启用**且端口**已监听**，可直接访问下一章节[访问WeBASE](#access)

- 如果上述检查步骤出现异常，如检查不到容器或端口监听，则需要按[日志路径](#logpath)进入**异常子服务**的日志目录，检查该服务的日志

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
  - 使用云服务厂商的服务器时，需要开通网络安全组的对应端口。如开放WeBASE管理台使用的5000端口
- WeBASE管理平台使用说明请查看[使用手册](../WeBASE-Console-Suit/index.html#id13)（获取WeBASE管理平台默认账号和密码，并初始化系统配置）
  - 默认账号为`admin`，默认密码为`Abcd1234`。首次登陆要求重置密码
  - 添加节点前置WeBASE-Front到WeBASE管理平台；一键部署时，节点前置与节点管理服务默认是同机部署，添加前置则填写IP为`127.0.0.1`，默认端口为`5002`。参考上文中`common.properties`的配置项`front.port={frontPort}`
- 检查节点前置是否启动，可以通过访问`http://{frontIp}:{frontPort}/WeBASE-Front`(默认端口5002)；访问前，确保服务端已对本地机器开放端口，如开放front的5002端口。（不建议节点前置的端口对公网开放访问权限，应对部分机器IP按需开放）


## 附录


<span id="install_docker" />

### 安装 Docker
- Debian/Ubuntu/CentOS/RHEL
  ```Bash
  # 该脚本是 Docker 官方提供的 Linux 自动安装脚本
  bash <(curl -s -L get.docker.com)
  ```
  
  在 CentOS/RHEL 8.x 中，使用上面的自动脚本安装时，会出现下面的错误：
  
  ```Bash
  Last metadata expiration check: 0:37:43 ago on Sat 22 Feb 2020 07:40:15 PM CST.
  Error: 
   Problem: package docker-ce-3:19.03.6-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
    - cannot install the best candidate for the job
    - package containerd.io-1.2.10-3.2.el7.x86_64 is excluded
    - package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
    - package containerd.io-1.2.2-3.el7.x86_64 is excluded
    - package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
    - package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
    - package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
  (try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages) 
  ```
  
  要解决这个问题，需要手动安装 `containerd.io`后，在执行自动安装脚本
  
  ```Bash
  # 下载最新的 containerd.io 安装包
  wget https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm 
  
  # 手动安装 containerd.io 
  yum localinstall containerd.io-1.2.13-3.2.el7.x86_64.rpm 
  
  ```

- MacOS

  ```brew install --cask --appdir=/Applications docker```

<span id="docker-compose-install"></span>

### Docker-Compose安装

获取Docker-Compose的github仓库提供的二进制文件，其中版本号`1.29.2`可切换到更新版本，`-o`则输出到指定位置

*依赖 curl 进行下载*
```Bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

修改执行权限
```
sudo chmod +x /usr/local/bin/docker-compose
```

检测安装是否成功
```
$ docker-compose --version

docker-compose version 1.29.2, build 5becea4c
```


<span id="python3"></span>
### Python部署

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
- MacOS

  ```
  // brew 安装python3时，会自动安装pip3
  sudo brew install python3
  ```

### 数据库部署
<span id="mysql-install"></span>

#### ① CentOS安装MariaDB

此处以**CentOS 7(x86_64)**安装**MariaDB 10.3**为例。*MariaDB*数据库是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可。*MariaDB*完全兼容 MySQL，包括API和命令行。MariaDB 10.3版本对应Mysql 5.7。其他安装方式请参考[MySQL官网](https://dev.mysql.com/downloads/mysql/)。
- CentOS 7 默认MariaDB为5.5版本，安装10.3版本需要按下文进行10.3版本的配置。
- 若使用CentOS 8则直接使用`sudo yum install -y mariadb*`即可安装MariaDB 10.3，并跳到下文的 *启停* 章节即可。

使用`vi`或`vim`创建新文件`/etc/yum.repos.d/mariadb.repo`，并写入下文的文件内容（参考[MariaDB中科大镜像源修改](http://mirrors.ustc.edu.cn/help/mariadb.html)进行配置）

- 创建repo文件
```Bash
sudo vi /etc/yum.repos.d/mariadb.repo
```

- 文件内容，此处使用的是中科大镜像源
```Bash
# MariaDB 10.3 CentOS repository list - created 2021-07-12 07:37 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = https://mirrors.ustc.edu.cn/mariadb/yum/10.3/centos7-amd64
gpgkey=https://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

- 更新yum源缓存数据
```
yum clean all
yum makecache all
```

- 安装`MariaDB 10.3
- 如果已存在使用`sudo yum install -y mariadb*`命令安装的MariaDB，其版本默认为5.5版本，对应Mysql版本为5.5。新版本MariaDB无法兼容升级，需要先**卸载旧版本**的MariaDB，卸载前需要**备份**数据库内容，卸载命令可参考`yum remove mariadb`
```
sudo yum install MariaDB-server MariaDB-client -y
```

若安装时遇到错误`“Failed to connect to 2001:da8:d800:95::110: Network is unreachable”`，将源地址中的 `mirrors.ustc.edu.cn` 替换为 `ipv4.mirrors.ustc.edu.cn` 以强制使用 IPv4：
```
sudo sed -i 's#//mirrors.ustc.edu.cn#//ipv4.mirrors.ustc.edu.cn#g' /etc/yum.repos.d/mariadb
```

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

答：因为有个别组件是启动成功的，需先执行“python deploy.py stopDockerAll”将其停止，再执行“python deploy.py installDockerAll”部署全部。

### 4. 部署时数据库访问报错

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

### 5. 节点sdk目录不存在

```
...
======= FISCO-BCOS sdk dir:/data/app/nodes/127.0.0.1/sdk is not exist. please check! =======
```

答：确认节点安装目录下有没有sdk目录（企业部署工具搭建的链可能没有），如果没有，需手动创建"mkdir sdk"，并将节点证书（ca.crt、sdk.key、sdk.crt、node.crt、node.key）复制到该sdk目录，再重新部署。如果是国密链，并且sdk和节点使用国密ssl连接时，需在sdk目录里创建gm目录，gm目录存放国密sdk证书（gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt和gmensdk.key）。


### 6. 服务进程起来了，服务不正常

```
...
======= WeBASE-Node-Manager  starting . Please check through the log file (default path:./webase-node-mgr/log/). =======
```

答：查看日志，确认问题原因。确认后修改重启，如果重启提示服务进程在运行，先执行“python deploy.py stopDockerAll”将其停止，再执行“python deploy.py startDockerAll”重启。


<span id="docker_sudo"></span>

### 7. docker必须使用sudo才能运行，但是sudo下系统环境变量失效

答：可以在root用户下配置环境变量如JAVA_HOME等，或者通过下面操作，尝试创建docker用户组

```
# 创建docker用户组
sudo groupadd docker
# 将当前用户添加到docker用户组
sudo usermod -aG docker $USER
# 重启docker服务
sudo systemctl restart docker
# 切换或者退出当前账户，重新ssh登入
exit
```

### 8. 如何使用启动Docker镜像

答：使用`docker run`命令启动的配置方法，可以参考WeBASE一键部署工具`WeBASE/deploy/docker`目录中的docker-compose配置文件`docker-compose-template.yaml`。

以`docker-compose-template.yaml`中的WeBASE-Front为例，启动容器时会用到docker启动时专用的`application-docker.yml`配置文件，可以通过配置docker容器的环境变量以替换`application-docker.yml`中的默认变量：

```yaml
 webase-front:
    image: webasepro/webase-front:v1.5.3
    container_name: webase-front-5002
    network_mode: "host"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SERVER_PORT: 5002
      SDK_IP: 127.0.0.1
      SDK_CHANNEL_PORT: 20200
      KEY_SERVER: "127.0.0.1:5004"
    volumes:
      - /webase-deploy/nodes/127.0.0.1/sdk:/dist/sdk 
      - /webase-deploy/webase-front/log:/dist/log
      - /webase-deploy/webase-front/h2:/h2
```

如，在`application-docker.yml`中替换`sdk`连接节点的配置`SDK_CHANNEL_IP`:
- 在容器启动时通过`-e SPRING_PROFILES_ACTIVE=docker`指定spring的配置文件为docker的yml，并通过`-e SDK_CHANNEL_PORT=20201`指定channel连接的端口，
```yml
sdk:
  corePoolSize: 50
  maxPoolSize: 100
  queueCapacity: 100
  ip: ${SDK_IP:127.0.0.1}
  channelPort: ${SDK_CHANNEL_PORT:20200}
  certPath: conf
```

综上所述，以下为启动WeBASE-Front容器的完整`docker run`命令示例
- 将sdk证书挂载到容器的`/dist/sdk`中
- 将容器中的h2数据库目录和日志目录挂载到宿主机中
- 将一个`application-docker.yml`挂载到容器中，覆盖容器中默认的yml配置，并配置SPRING环境变量以启用`-docker`结尾的yml配置文件
```shell
docker run -d --rm --name=webase-front  --network=host -v /data/home/webase/webase/compose/WeBASE-Front/sdk:/dist/sdk -v /data/home/webase/webase/compose/WeBASE-Front/h2:/h2  -v /data/home/webase/webase/compose/WeBASE-Front/log:/dist/log -v /data/home/webase/webase/compose/WeBASE-Front/application-docker.yml:/dist/conf/application-docker.yml -e SPRING_PROFILES_ACTIVE=docker webasepro/webase-front:v1.5.3
```

使用WeBASE-Web镜像时同理，参考`docker-compose.yaml`对WeBASE-Web的配置项，挂载一个nginx.conf到容器中以覆盖容器中默认的nginx配置，并将log目录挂载出来
```yaml
  webase-web:
    image: webasepro/webase-web:v1.5.3
    container_name: webase-web-5000
    network_mode: "host"
    volumes:
      - /webase-deploy/webase-web/nginx-docker.conf:/data/webase-web/nginx/nginx.conf
      - /webase-deploy/webase-web/log:/dist/log
```

### 9. MacOS中如何使用`sed`命令替换文件中的内容
<span id="mac_sed"></span>
答：因为MacOS与`Linux`系统中对于`sed`的处理处理方式不同。如果想在MacOS中拥有Linux中`sed`体验，可以通过以下方式。
- 安装 gnu-sed
  ```brew install gnu-sed```
- 设置环境变量
  ```vi ~/.zshrc```

  ```export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"```

  ```source ~/.zshrc```
- 替换文件中的内容

  ```sed -i "" "s/target/value/g"```
  
  * Tips
  ```sed -i ""``` 等同于Linux中的```sed -i```
  
    如果没有添加`""`，则会替换文本的时候，生成副本.


##### demo
* 现在有文件`webase.txt`, 内容如下
```text
Hello,
12345
aaabbbaaa
cccaba
```
* 将`webase.txt`中的`12345`替换成`webase`
```
sed -i "" "s/12345/webase/" webase.txt
```

>`-i`表示直接操作文件并不需要备份文件，如果需要备分则使用 `-i "备份名称"`
> 
>`s` 代表 substitue 即替换
> 
>`12345` 表示被代替的文字
>
>`webase` 表示替换的文字
>
>`webase.txt` 表示对应修改的文件

* 此时`webase.txt`内容如下：
```text
Hello,
webase
aaabbbaaa
cccaba
```



*欢迎给WeBASE的文档提交 Pull Request 补充更多的 Q&A*
