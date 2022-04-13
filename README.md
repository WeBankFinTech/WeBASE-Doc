# 区块链教程 | 交易处理接口（结合WeBASE-Sign）的流程分析
来自WeBASE-Front官网：

WeBASE-Front是和FISCO BCOS节点配合使用的一个子系统，需要和节点统计部署，目前支持FISCO BCOS 2.0以上版本，可通过HTTP请求和节点进行通信，集成了web3sdk(java-sdk)，对接口进行了封装和抽象，具备可视化控制台，可以在控制台上查看交易和区块详情，开发智能合约，管理私钥，并对节点健康度进行监控和统计。

## 前期准备
- 搭建一条 FISCO BCOS 区块链
- 搭建 WeBASE-Front 前置服务
    - https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Front/index.html 
- 搭建 WeBASE-Sign 签名服务
    - https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Sign/index.html 
- 搭建 WeBASE-Manager 管理服务
    - https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Node-Manager/index.html  
- 搭建 WeBASE-Web 管理平台
  - https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Web/index.html 

#### 最终效果
浏览器访问：http://127.0.0.1:5000/#/login

![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCEfb48fc6ab7f0905c7638aece691cb4a0)

## 使用 WeBASE-Web 搭建所需要的环境：
**1. 使用 WeBASE-Web 创建一个用户**

![image.png](https://note.youdao.com/yws/res/5/WEBRESOURCE5d82f83535f1628a14c0e311689ec2a5)

**2. 编写 HelloWorld.sol**
```
contract HelloWorld {
    string my_name;
    function Insert(string name) public {
       my_name = name;
    }
}
```
 WeBASE-Web 管理平台 上部署的效果
 
![image.png](https://note.youdao.com/yws/res/8/WEBRESOURCE13eec6bf2d3553335adab8ee9ee274c8)

==然后 保存 -> 编译 -> 部署==

## 访问 WeBASE-Front Swagger-ui 
- 浏览器访问: http://127.0.0.1:5002/WeBASE-Front/swagger-ui.html
- 找到 /trans/handleWithSign 的 API 接口
![image.png](https://note.youdao.com/yws/res/1/WEBRESOURCEf25a215989691aae3f8f654a5cf7fbb1)

里面有相关的测试用例对其进行测试操作

**可以查看 WeBASE-Front 接口文档**
- 文档地址：https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE-Front/interface.html?highlight=handleWithSign#id2

![image.png](https://note.youdao.com/yws/res/f/WEBRESOURCE9aa46968a4382f7f2d31a4899da9f07f)

## 使用 Swagger-ui 或者 Postman 发送请求
这里是使用 postman 来对 /trans/handleWithSign 发送请求，postman 看起来比较清晰

**1) 请求所需要的参数**
```
{
    "groupId" :1,
    "signUserId": "2a6559877ea341e087f6822459679aa4", 
    "contractAbi":[{"constant":false,"inputs":[{"name":"name","type":"string"}],"name":"Insert","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}],
    "contractAddress":"0xeeae8c36c4df8965fca297bed205af722be817f7",
    "funcName":"Insert",
    "funcParam":["coms"],
    "useCns":false
}

```
- signUserId : 根据部署合约的用户 signUserId 

![image.png](https://note.youdao.com/yws/res/9/WEBRESOURCEe84211af3f8150ed93ecec4974c89c79)

- contractAddress: 合约地址
- contractAbi: 合约相应的 abi

![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCE834a7920d970232178848e566acf7130)


**2) 使用 postman 请求**

![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCE7a6b241b81d6a0d6b3d623c2c4309e80)

- **通过调用WeBASE-Sign服务的签名接口让相关用户对数据进行签名，拿回签名数据再发送上链** "status": "0x0" 则为成功

## 源码流程分析

- 要从 WeBASE-Front 前置节点源码深入分析
- 分析 WeBASE-Sign 是如果对交易进行加密

### 1）WeBASE-Front 源码分析
WeBASE-Front 源码：
git clone https://gitee.com/WeBank/WeBASE-Front.git
![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCE45e5ffbb9ae6ea2ccb85ded55a085b30)

**public Object transHandle**
- 处理 postman request 数据
![image.png](https://note.youdao.com/yws/res/3/WEBRESOURCE44f0509f2ca4dce805c65148fc8b5f53)

**public Object transHandleWithSign**
- 解析过后的数据，交给该函数进行处理
![image.png](https://note.youdao.com/yws/res/9/WEBRESOURCE2d68751874d8347fcd785f0e45877b09)

**public Object transHandleWithSign**
- 解析后的 groupId 数据，获取 web3ApiService client
- 解析后的  funcName (执行函数名：insert), funcParam（传过去数据：coms）对其进行编码成字符串数据流
![image.png](https://note.youdao.com/yws/res/e/WEBRESOURCE29e9b9ae2dab08cb077f462217d09c0e)

**public TransactionReceipt handleTransaction**
- this.requestSignForSign 是把编码后的交易和签名用户ID 发送给 WeBASE-Front-sign 对其进行交易签名
- this.sendMessage 则是发送交易
![image.png](https://note.youdao.com/yws/res/a/WEBRESOURCE44e7b2ff203c8cc374fb471ad9270b2a)



### 2）WeBASE-Sign 源码分析
2. WeBASE-Sign 源码：git clone https://github.com/WeBankBlockchain/WeBASE-Sign.git

重点关注：
![image.png](https://note.youdao.com/yws/res/8/WEBRESOURCE347999d9e8025ba6fd306b0040f7f438)

![image.png](https://note.youdao.com/yws/res/4/WEBRESOURCEb56cf9f54dea28675c94213795da8c44)

**sign 方法**
- check signUserId
- 根据 signUserId 获取 cryptoKeyPair（加密密钥对），往下的代码就是对交易加密相关操作
- 对其交易内容进行消息签名
- 往下最后把执行结果交给 WeBASE-Front

![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCE24ef70abe624bf0d5c602808cb741c70)

### 3）总结
  WeBASE-Front API: /trans/handleWithSign 接受数据求就交给 WeBASE-Sign API: sign 对数据进行签名 ， 然后返回
  给  WeBASE-Front 在进行上链操作。重点了解上面标注的函数，都是核心方法。
