# 快速入门搭建

在区块链应用开发阶段建议用户使用快速入门搭建。在快速入门搭建模式，开发者只需要搭建节点和节点前置服务(WeBASE-Front)，就可以通过WeBASE-Front的合约编辑器进行合约的编辑，编译，部署，调试。

```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBASE及其他子系统的版本兼容说明！`请查看 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```

### 节点搭建

节点搭建的方法建议使用[build_chain](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)。

##### Liquid支持

如果使用的`liquid`合约的链并在WeBASE-Front的合约IDE中编译Liquid合约，要求**手动**在WeBASE-Front所在主机[配置Liquid环境](https://liquid-doc.readthedocs.io/zh_CN/latest/docs/quickstart/prerequisite.html)

配置好Liquid环境后，需要重启WeBASE-Front

### 节点前置服务搭建

**前提条件** 

| 依赖软件 | 支持版本 |
| :-: | :-: |
| Java | [JDK 8 至JDK 14](../WeBASE-Front/appendix.html#java) |

备注：部署出现问题请查看[问题记录](../WeBASE-Front/appendix.html#q&a)。

1. 下载安装包
    ```shell
    wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.1.1/webase-front.zip

    # 网络访问失败，则可以尝试直接git clone WeBASE-Front的仓库，构建方法可参考节点前置部署文档
    git clone -b master-3.0 https://github.com/WeBankBlockchain/WeBASE-Front.git
    # 若因网络问题导致长时间下载失败，可尝试以下命令
    git clone -b master-3.0 https://gitee.com/WeBank/WeBASE-Front.git

    ```


2. 解压
    ```shell
    unzip webase-front.zip && cd webase-front
    ```

3. 拷贝sdk证书文件（build_chain的时候生成的） 

    将节点所在目录`nodes/${ip}/sdk`下的所有文件拷贝到当前`conf`目录，供SDK与节点建立连接时使用
    - 拷贝命令可使用`cp -r nodes/${ip}/sdk/* ./conf/`

    - 证书为以下两种之一：

      非国密：`ca.crt`、`sdk.crt`、`sdk.key`

      国密：`sm_ca.crt`、`sm_sdk.crt`、`sm_sdk.key`、`sm_ensdk.crt`、`sm_ensdk.key`


4. 修改配置

    ```
    vi conf/application.yml
    ```

    ```
    sdk:
      useSmSsl: false  # sdk连接节点是否使用国密ssl
      peers: ['127.0.0.1:20200','127.0.0.1:20201'] # 节点ip和rpc端口
    ```

5. 服务启停

    返回根目录，服务启停命令：
    ```shell
    启动： bash start.sh
    停止： bash stop.sh
    检查： bash status.sh 
    ```

启动成功将出现如下日志：
```
...
	Application() - main run success...
```

### 状态检查

成功部署后，可以根据以下步骤**确认各个子服务是否启动成功**

##### 1. 检查各子系统进程

通过`ps`命令，检查节点与节点前置的进程是否存在
- 包含：节点进程`nodeXX`，节点前置进程`webase.front`

检查方法如下，若无输出，则代表进程未启动，需要到`webase-front/log`中查看日志的错误信息，并根据错误提示或根据[WeBASE-Front常见问题](../WeBASE-Front/appendix.html#q&a)进行错误排查

检查节点进程
```shell
$ ps -ef | grep node
```
输出如下，此处部署了两个节点node0, node1
```
root     29977     1  1 17:24 pts/2    00:02:20 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node1/../fisco-bcos -c config.ini
root     29979     1  1 17:24 pts/2    00:02:23 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node0/../fisco-bcos -c config.ini
```

检查节点前置webase-front的进程
```
$ ps -ef | grep webase.front 
```
输出如下
```
root     31805     1  0 17:24 pts/2    00:01:30 /usr/local/jdk/bin/java -Djdk.tls.namedGroups=secp256k1 ... conf/:apps/*:lib/* com.webank.webase.front.Application
```

##### 2. 检查进程端口

通过`netstat`命令，检查节点与节点前置的端口监听情况

检查方法如下，若无输出，则代表进程端口监听异常，需要到`webase-front/log`中查看日志的错误信息，并根据错误提示或根据[WeBASE-Front常见问题](../WeBASE-Front/appendix.html)进行错误排查

检查节点channel端口(默认为20200)是否已监听
```shell
$ netstat -anlp | grep 20200
```
输出如下
```
tcp        0      0 0.0.0.0:20200           0.0.0.0:*               LISTEN      29069/fisco-bcos
```

检查webase-front端口(默认为5002)是否已监听
```
$ netstat -anlp | grep 5002
```
输出如下
```
tcp6       0      0 :::5002                 :::*                    LISTEN      2909/java 
```

##### 3. 检查服务日志 

日志中若出现报错信息，可根据信息提示判断服务是否异常，也可以参考并根据错误提示或根据[WeBASE-Front常见问题](../WeBASE-Front/appendix.html)进行错误排查

- 如果节点进程**已启用**且端口**已监听**，可跳过本章节
- 如果节点前置异常，如检查不到进程或端口监听，则需要`webase-front/log`中查看日志的错误信息
- 如果检查步骤出现检查不到进程或端口监听等异常，或者前置服务无法访问，可以按以下顺序逐步检查日志：
  - 检查`webase-front/log`中查看节点前置日志的错误信息，如果无错误，且日志最后出现`application run success`字样则代表运行成功
  - 检查`nodes/127.0.0.1/nodeXXX/log`中的节点日志


**查看运行成功日志**：webase-front运行成功后会打印日志`main run success`，可以通过搜索此关键字来确认服务正常运行。

如，检查webase-front日志，其他webase服务可进行类似操作
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

启动失败或无法使用时，欢迎到WeBASE-Front提交[Issue](https://github.com/WeBankBlockchain/WeBASE-Front/issues)或到技术社区共同探讨
- 提交Issue或讨论问题时，可以在issue中配上自己的**环境配置，操作步骤，错误现象，错误日志**等信息，方便社区用户快速定位问题


### 访问

访问 http://{deployIP}:{frontPort}/WeBASE-Front，示例：  

    ```
    http://localhost:5002/WeBASE-Front 
    ```

**注**：若服务启动后无异常，但仍然无法访问，可以检查服务器的网络安全策略： 
- **开放节点前置端口**：如果希望通过浏览器(Chrome Safari或Firefox)直接访问webase-front节点前置的页面，则需要开放节点前置端口`frontPort`（默认5002）

![](../../images/WeBASE/front-overview.png)

