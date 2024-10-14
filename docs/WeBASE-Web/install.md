## 部署说明

### 1. 依赖环境

| 环境     | 版本              |
| ------ | --------------- |
| nginx   | nginx1.6或以上版本，安装请参考[附录](appendix.html) |
| WeBASE-Node-Manager    |  WeBASE-Node-Manager[对应版本](../WeBASE/ChangeLOG.md) |

### 2. 拉取代码

代码可以放在/data下面，执行命令：

```shell
git clone -b master-3.0 https://github.com/WeBankBlockchain/WeBASE-Web.git

# 若网络问题导致长时间无法下载，可尝试以下命令
git clone -b master-3.0 https://gitee.com/WeBank/WeBASE-Web.git
```

进入目录：

```
cd WeBASE-Web
```

#### 2.1  下载solc-bin

#### solidity v0.8.11支持

WeBASE-Web v3.0.1已支持solidity `v0.8.11`


执行脚本get_solc_js.sh会自动下载solc-bin，即下面v0.4.25.js等文件。
在`WeBASE-Web/`目录中直接执行脚本get_solc_js.sh（（脚本与`dist`文件夹同级））

```
    bash ./get_solc_js.sh
```
等待脚本执行完成


- 如果执行不成功，请使用下面的命令：

`注意：当且仅当get_solc_js.sh脚本执行失败才需要执行下面的命令`  
```
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.4.25.js -o ./dist/static/js/v0.4.25.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.4.25-gm.js -o ./dist/static/js/v0.4.25-gm.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.5.2.js -o ./dist/static/js/v0.5.2.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.5.2-gm.js -o ./dist/static/js/v0.5.2-gm.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.6.10.js -o ./dist/static/js/v0.6.10.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.6.10-gm.js -o ./dist/static/js/v0.6.10-gm.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.8.11.js -o ./dist/static/js/v0.8.11.js
    curl -#L https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v3.0.0/v0.8.11-gm.js -o ./dist/static/js/v0.8.11-gm.js
```

执行完后检查dist/static/js是否下载完这些js文件，文件大小在5m-20m之间。


### 4. 修改配置

在docs目录下有配置文件nginx.conf，修改完后替换安装的nginx的配置文件nginx.conf（这里nginx安装配置文件在/usr/local/nginx/conf下面，如果这里没找到，可以到/etc下寻找，如有权限问题，请加上sudo）。


- 修改配置：

```
# 修改服务器ip，也可以使用域名
sed -i "s%127.0.0.1%${your_ip}%g" docs/nginx.conf

# 修改WeBASE-Web服务端口（端口需要开通策略且不能被占用）
sed -i "s%5000%${your_port}%g" docs/nginx.conf

# 修改静态文件路径（文件需要有权限访问）
sed -i "s%/data/WeBASE-Web/dist%${your_file_dir}%g" docs/nginx.conf

# 节点管理服务ip和端口
sed -i "s%10.0.0.1:5001%${your_node_manager}%g" docs/nginx.conf
```

- 复制配置文件nginx.conf到默认配置目录中
- 也可以直接通过`nginx -c docs/nginx.conf`命令加载docs/nginx.conf配置

```
cp -rf docs/nginx.conf /usr/local/nginx/conf
```

**备注：**  如果服务器已有nginx，可在原配置文件nginx.conf增加一个server：

```
    upstream node_mgr_server{
        server 10.0.0.1:5001; # 节点管理服务ip和端口
    }
    server {
        listen       5000 default_server; # 前端端口（端口需要开通策略且不能被占用）
        server_name  127.0.0.1;           # 服务器ip，也可配置为域名
        location / {
            root   /data/WeBASE-Web/dist;   # 前端文件路径(文件需要有权限访问)
            index  index.html index.htm;
            try_files $uri $uri/ /index.html =404;
        }

        include /etc/nginx/default.d/*.conf;

        location /mgr {
            proxy_pass    http://node_mgr_server/;    		
            proxy_set_header		Host			 $host;
            proxy_set_header		X-Real-IP		 $remote_addr;
            proxy_set_header		X-Forwarded-For	 $proxy_add_x_forwarded_for;
        }
    }
```

### 5. 启动nginx

启动命令：

	/usr/local/nginx/sbin/nginx # nginx在/usr/local目录下

检查nginx是否启动：

```
ps -ef | grep nginx
```

### 6. 访问页面

```
http://{deployIP}:{webPort}
示例：http://127.0.0.1:5000
```

**备注：** 

- 部署服务器IP和管理平台服务端口需对应修改，网络策略需开通
- 默认账号密码：admin/Abcd1234
- WeBASE管理平台使用说明请查看[使用手册](../WeBASE-Console-Suit/index.html#id13)

### 7. 查看日志

```
进程日志：tail -f logs/access.log
错误日志：tail -f logs/eror.log
```

