# 升级说明

WeBASE-Web升级说明，请结合[WeBASE-Web Changelog](https://github.com/WeBankBlockchain/WeBASE-Web)和[WeBASE管理平台使用手册](../WeBASE-Console-Suit/index.html)进行阅读。

WeBASE-Web升级的必须步骤：
0. 备份已有文件或数据，下载新的安装包（可参考[安装包下载](../WeBASE/mirror.html#install_package)）
1. 采用新的安装包，替换旧的`webase-web`目录，无需重启nginx

各个版本的具体修改可参考下文

#### v1.5.4 
- 增加管理台右边栏操作指引
- 优化合约IDE交易体验
- 优化登录页“忘记密码“与”验证码加载失败“提示


#### v1.5.3
- 新增WeBASE-Web docker镜像，`webasepro/webase-web:v1.5.3`
- 新增数据监控大屏
- 新增导出私钥功能，导出格式为p12
- 新增公钥用户绑定私钥功能
- 新增系统管理中的共识超时时间配置
- 合约仓库新增代理合约模板、溯源合约模板
- 优化合约IDE回退问题，仅回退到后台记录版本
- 修复合约IDE页面事件bus重复问题
- 修复全量合约/全量用户搜索问题
- 修复合约IDE创建文件夹未自动刷新问题
- 修复交易回执解析中int类型显示为string的显示问题

#### v1.5.2
- 优化合约Java项目导出功能，支持批量编译合约，支持多用户与channel端口检查
- 合约仓库新增Asset资产合约
- 优化交易弹窗，支持无私钥用户时直接创建私钥
- 支持开发者导出私钥、支持合约IDE绑定合约地址

#### v1.5.1
- 新增导出合约Java工程脚手架
- 新增合约仓库
- 新增全量用户/合约通过地址搜索功能
- 合约IDE编译器js切换至WASM版本，并采用Worker加载方式，修复部分chrome浏览器加载失败问题
- 修复手机版登陆态过期未跳转到登录页
- 修复无法删除合约目录问题
- 修复合约IDE中合约调用参数为字符串时不能输入空格

#### v1.5.0
- 新增应用管理，支持WeIdentity模板和自定义应用接入
- 新增节点监控的链上TPS、出块周期、块大小的统计
- 新增合约列表中的已登记合约与链上全量合约视图、新增私钥用户列表中的已登记私钥与链上全量私钥视图
- 支持导出Txt/Pem/P12/WeID私钥文件、支持导出前置的SDK证书
- 新增适配移动端的WeBASE管理台

其中移动端管理台需要启用新的nginx.conf，新增了移动端自动重路由、移除auto-index、增加gzip压缩
- 需要将已有的webase-node-mgr的ip port及webase-web的port配置到新的nginx.conf文件中，使用nginx重载配置文件

新增内容如下
- 需要启用移动端时，则需要下载移动端的webase-web-mobile.zip安装包并解压，即`wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-web-mobile.zip`，并解压`unzip webase-web-mobile.zip`
- 在nginx.conf中的`location /`中的`phone_page_url`替换为`webase-web-mobile`解压后的路径(已有的`root web_page_url`无需修改)
- 最后使用`nginx -s reload`重载新的nginx配置文件
```
location / {    
    # default pc page url
    root   web_page_url;
    # if using phone
    if ( $http_user_agent ~ "(MIDP)|(WAP)|(UP.Browser)|(Smartphone)|(Obigo)|(Mobile)|(AU.Browser)|(wxd.Mms)|(WxdB.Browser)|(CLDC)|(UP.Link)|(KM.Browser)|(UCWEB)|(SEMC-Browser)|(Mini)|(Symbian)|(Palm)|(Nokia)|(Panasonic)|(MOT-)|(SonyEricsson)|(NEC-)|(Alcatel)|(Ericsson)|(BENQ)|(BenQ)|(Amoisonic)|(Amoi-)|(Capitel)|(PHILIPS)|(SAMSUNG)|(Lenovo)|(Mitsu)|(Motorola)|(SHARP)|(WAPPER)|(LG-)|(LG/)|(EG900)|(CECT)|(Compal)|(kejian)|(Bird)|(BIRD)|(G900/V1.0)|(Arima)|(CTL)|(TDG)|(Daxian)|(DAXIAN)|(DBTEL)|(Eastcom)|(EASTCOM)|(PANTECH)|(Dopod)|(Haier)|(HAIER)|(KONKA)|(KEJIAN)|(LENOVO)|(Soutec)|(SOUTEC)|(SAGEM)|(SEC-)|(SED-)|(EMOL-)|(INNO55)|(ZTE)|(iPhone)|(Android)|(Windows CE)|(Wget)|(Java)|(curl)|(Opera)" )
    {
            root   phone_page_url;
    }
    index  index.html index.htm;
    try_files $uri $uri/ /index.html =404;
}
```
- 需要启用gzip时，在nginx.conf中的`server`中添加以下内容
```
server {
    ...

    # zip solidity js file
    gzip  on;
    gzip_min_length     10k;
    gzip_buffers        32 4k;
    gzip_http_version   1.0;
    gzip_comp_level     1;
    gzip_proxied        any;
    gzip_types          text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
    gzip_vary           on;
    gzip_disable        "MSIE [1-6]\.";
}
```


#### v1.4.1
新增FISCO BCOS v2.5.0及以上版本的基于角色的权限管理功能，新增了开发者模式
- 新的权限管理基于角色，可参考FISCO BCOS[权限控制文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/permission_control.html)
- 开发者模式：新增了用户角色developer，可进行查询交易，合约部署调用等功能，无法使用管理员的系统管理与监控等功能。

#### v1.4.0
v1.4.0 主要在兼容原有手动部署底层服务，手动添加 WeBASE-Front 前置服务的基础上，新增了可视化部署底层服务，以及节点的动态管理功能。

- 增加左下展示版本号，包括**链版本**和**兼容版本**。如果是国密版本，链版本号会**带有** `gm` 后缀，兼容版本仅代表兼容的节点版本，**不带有** `gm` 后缀。

**提示**
- 如果要体验可视化部署，请参考[可视化部署](../WeBASE-Install/visual_deploy.html)部署新环境然后部署新链；


#### v1.3.1

v1.3.1主要新增了动态群组管理、合约ABI导入、合约ABI编码器、支持导入私钥等功能，详情升级说明如下：

- 新增动态群组管理，包含生成群组、启动/停止群组、删除/恢复群组、查询节点群组状态等功能，操作说明可参考[动态群组管理使用指南](../WeBASE-Console-Suit/index.html#dynamic_group_use)
- 新增导入已部署合约ABI功能，支持导入已部署合约，进行合约调用
- 新增合约Abi编码器，可通过ABI构建交易参数
- 新增导入.p12/.pem/.txt私钥功能；其中.txt与节点前置导出私钥格式一致，.p12/.pem与控制台导出私钥格式一致；
