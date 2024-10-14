# 国内镜像和CDN加速攻略
本节为访问GitHub较慢的用户提供国内源码镜像与安装包下载地址，以及WeBASE文档加速访问介绍。

## WeBASE及子系统源码及安装包

### 源码同步

WeBASE当前仓库源码位于[https://github.com/WeBankBlockchain/WeBASE](https://github.com/WeBankBlockchain/WeBASE)，每个新的版本发布会将代码合入master分支。

为了方便国内用户，我们同样在gitee上提供了镜像仓库[https://giteee.com/Webank/WeBASE](https://giteee.com/Webank/WeBASE)，每次新版本发布后，镜像仓库会同步GitHub上官方仓库的更新，如果从GitHub下载失败，请尝试使用Gitee镜像仓库。

WeBASE各子系统的Github代码仓库则是`https://github.com/WeBankBlockchain/` + `WeBASE-XXX`，对应的gitee仓库则是`https://gitee.com/WeBank/` + `WeBASE-XXX`

如WeBASE-Front的Github代码仓库为`https://github.com/WeBankBlockchain/WeBASE-Front`，Gitee代码仓库为`https://gitee.com/WeBank/WeBASE-Front`

### 一键部署与安装包
<span id="install_package"></span>
WeBASE每个新版本发布后，会在[WeBASELargefiles](https://github.com/WeBankBlockchain/WeBASELargefiles/releases)GitHub的Releases中提供对应的WeBASE一键部署工具和对应安装包。

其中WeBASELargefiles提供webase-deploy一键部署工具（即WeBASE源码中`/deploy`文件夹），以及webase-front.zip, webase-node-mgr.zip, webase-sign.zip, webase-web.zip子系统的安装包。

同时提供以下国内镜像，**可加速下载安装包**：
```Bash
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/{release_version}/webase-{subsystem}.zip
```
其中`{release_version}`为`v1.x.x`格式，`{subsystem}`则是子系统名字。

支持下载`sign, front, node-mgr, web`子系统的zip安装包（全小写），暂不支持webase-transaction的安装包下载。

*注：可视化部署中通过国内镜像手动下载docker镜像，文件名则是`docker-fisco-webase.tar`*

## WeBASE文档镜像

WeBASE文档使用readthedocs管理，全部开源于[https://webasedoc.readthedocs.io/zh_CN/latest/index.html](https://webasedoc.readthedocs.io/zh_CN/latest/index.html)，同样提供国内镜像文档[https://fintech.webank.com/developer/docs/webase/index.html](https://fintech.webank.com/developer/docs/webase/index.html)，*由于网站资源更新周期安排，国内镜像文档更新会比readthedocs有所延迟*。

每个版本发布会为上个版本的文档打Tag，新版本的文档会合入主干分支，文档由于会持续改进，所以是下个版本发布才打上个版本的tag。readthedocs文档支持下载PDF格式，方便用户使用。

## 举例：使用国内镜像进行一键部署

本节WeBASE 1.5.0为例进行一键部署，一键部署会默认使用国内镜像下载安装包，下面仅演示关键步骤，具体操作可参考[WeBASE一键部署](./install.html)

### 下载WeBASE一键部署工具

```bash
# 使用CDN下载
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-deploy.zip

# 使用github下载
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-deploy.zip
```


### 单独下载WeBASE子系统的安装包

WeBASE一键部署(webase-deploy)会自动下载子系统安装包，用户也可以手动下载安装包或编译源码得到安装包，并复制到webase-deploy目录下。

如需手动下载某一子系统的安装包，可以直接通过`wget`或者`curl -O`命令直接获取安装包。比如：

- 获取WeBASE-Node-Manager v1.4.1的安装包`webase-node-mgr.zip`

```
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-front.zip
// 或
curl -#LO https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-front.zip
```

### 单独下载WeBASE的solc JS文件

WeBASE提供FISCO BCOS中使用的v0.4.25, v0.5.2, v0.6.10三个版本的solc JS编译文件，对应的国密版本则在版本号后加上`-gm`后缀

如需手动下载某一版本的的安装包，可以直接通过`wget`或者`curl -O`命令直接获取安装包。比如：

- 获取v0.4.25的国密版本solc JS编译文件

```
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.4.25-gm.js
// 或
curl -#LO https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.4.25-gm.js
```


若通过源码编译获取安装包并用于一键部署工具，需要进行文件夹的重命名，参考下一章节。

## 举例：使用国内源码镜像编译WeBASE-Front

本节以WeBASE-Front子系统为例，从gitee镜像下载源码并编译，编译后的配置方法，请参考各子系统的安装部署文档。

### 下载源码

```bash
git clone https://gitee.com/WeBank/WeBASE-Front.git
```

### 编译源码

依据WeBASE-Front节点前置安装文档的[环境要求](../WeBASE-Front/install.html)进行配置，如jdk(oracle jdk8及以上)，配置gradle(4.10或以上)或直接使用gradlew脚本进行编译

```bash
// java版本
java -version
// 进入源码目录
cd WeBASE-Front
// 已有gradle时，使用gradle
gradle build -x test
// 不配置gradle，直接使用gradlew
chmod +x ./gradlew && ./gradlew build -x test
```

### 修改配置
编译完成后，将在当前目录得到`dist`文件夹

重命名dist包中的`conf_template`为`conf`后，并将节点的sdk证书复制到`conf`目录后，即可启动。

修改配置的具体方法，可参考[WeBASE-Front部署](../WeBASE-Front/install.html)
```
cd dist
mv conf_template conf
// 此处需要复制ca.crt, node.crt, node.key
cp -rf /fisco/nodes/127.0.0.1/sdk/* ./conf
bash start.sh
```
