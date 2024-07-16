##############################################################
WeBASE 技术文档
##############################################################

.. image:: ./images/logo/logo_smaller_new.png

WeBASE（WeBank Blockchain Application Software Extension） 是在区块链应用和FISCO BCOS节点之间搭建的一套通用组件。

- WeBASE lab版本已适配FISCO-BCOS 3.0.0版本，WeBASE 1.x版本与lab版本会持续更新维护。本技术文档适用于WeBASE 1.x版本，lab版技术文档可跳转至 `[WeBASE-lab分支] <https://webasedoc.readthedocs.io/zh_CN/lab>`_ 查看

.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBASE及其他子系统的兼容版本说明！`请查看 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_

.. container:: row 
   
   .. container:: card-holder
      
      .. container:: card rocket

         .. raw:: html

            <br>
            <h style="font-size: 22px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;快速开始</h>
            <br><br>
         
         - `什么是WeBASE <./docs/WeBASE/introduction.html>`_
         - `WeBASE版本信息 <./docs/WeBASE/ChangeLOG.html#fisco-bcos-2-x-x>`_
         - `安装部署 <./docs/WeBASE-Install/index.html>`_
         - `WeBASE管理平台使用手册 <./docs/WeBASE-Console-Suit/index.html>`_


   .. container:: card-holder
      
      .. container:: card ref

         .. raw:: html

            <br>
            <h style="font-size: 22px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WeBASE子系统</h>
            <br><br>
         
         - `节点前置服务 <./docs/WeBASE-Front/index.html>`_
         - `节点管理服务 <./docs/WeBASE-Node-Manager/index.html>`_
         - `WeBASE管理平台 <./docs/WeBASE-Web/index.html>`_
         - `私钥托管与签名服务 <./docs/WeBASE-Sign/index.html>`_
         - `交易服务 <./docs/WeBASE-Transaction/index.html>`_


.. container:: row 

   .. container:: card-holder
      
      .. container:: card ref

         .. raw:: html

            <br>
            <h style="font-size: 22px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决方案</h>
            <br><br>

         - `WeBASE应用管理 <./docs/WeBASE-APP-SDK/index.html>`_
         - `WeBASE合约仓库 <./docs/WeBASE-Contract-Warehouse/index.html>`_
         - `WeBASE实训课程案例 <./docs/WeBASE-Training-Class/index.html>`_


   .. container:: card-holder
      
      .. container:: card manuals

         .. raw:: html

            <br>
            <h style="font-size: 22px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更多资料</h>
            <br><br>
         
         - `WeBASE社区文章 <./docs/WeBASE-Console-Suit/articles/index.html>`_
         - `WeBASE贡献指南 <./docs/WeBASE/CONTRIBUTING.html>`_
         - `WeBASE国内镜像仓库、文档、安装包资源 <./docs/WeBASE/mirror.html>`_
         - `开源社区 <./docs/More/community.html>`_
         
.. container:: row 

   .. container:: card-holder-bigger
      
      .. container:: card-bigger rocket

         .. raw:: html

               <br>
               <h style="font-size: 22px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更多开源工具</h>
               <br><br>

               <img src="_images/overview_blockchain.png" class="card-holder-full">


         .. container:: tools 

            .. raw:: html

            - **FISCO BCOS企业级金融联盟链底层平台**: `[GitHub] <https://github.com/FISCO-BCOS/FISCO-BCOS>`_ `[Gitee] <https://gitee.com/FISCO-BCOS>`_ `[文档] <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/index.html>`_ 
            - **WeBASE 区块链中间件平台**：`[GitHub] <https://github.com/WeBankBlockchain/WeBASE>`_ `[Gitee] <https://gitee.com/WeBank/WeBASE>`_  `[文档] <https://webasedoc.readthedocs.io/>`_ 
            - **WeIdentity 基于区块链的实体身份标识及可信数据交换解决方案**: `[GitHub] <https://github.com/WeBankBlockchain/WeIdentity>`_ `[Gitee] <https://gitee.com/WeBank/WeIdentity>`_ `[文档] <https://weidentity.readthedocs.io/>`_ 
            - **WeDPR 即时可用，场景式隐私保护高效解决方案套件和服务**：`[GitHub] <https://github.com/WeBankBlockchain/WeDPR-Lab-Core>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/WeDPR-Lab-Crypto>`_ `[文档] <https://wedpr-lab.readthedocs.io/>`_ 
            - **WeCross 区块链跨链协作平台**: `[GitHub] <https://github.com/WeBankBlockchain/WeCross>`_ `[Gitee] <https://gitee.com/WeBank/WeCross>`_ `[文档] <https://wecross.readthedocs.io/>`_ 
            - **Truora 可信预言机服务**：`[GitHub] <https://github.com/WeBankBlockchain/Truora>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Truora>`_  `[文档] <https://truora.readthedocs.io/>`_ 
            - **Liquid 智能合约编程语言软件**：`[GitHub] <https://github.com/WeBankBlockchain/liquid>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/liquid>`_  `[文档] <https://liquid-doc.readthedocs.io/>`_
            - **WeBankBlockchain-Data 数据治理通用组件**：
               - Data-Stash 数据仓库组件： `[GitHub] <https://github.com/WeBankBlockchain/Data-Stash>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Data-Stash>`_  `[文档] <https://data-doc.readthedocs.io/zh_CN/stable/docs/WeBankBlockchain-Data-Stash/index.html>`_
               - Data-Export 数据导出组件： `[GitHub] <https://github.com/WeBankBlockchain/Data-Export>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Data-Export>`_  `[文档] <https://data-doc.readthedocs.io/zh_CN/stable/docs/WeBankBlockchain-Data-Export/index.html>`_
               - Data-Reconcile 数据对账组件：  `[GitHub] <https://github.com/WeBankBlockchain/Data-Reconcile>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Data-Reconcile>`_  `[文档] <https://data-doc.readthedocs.io/zh_CN/stable/docs/WeBankBlockchain-Data-Reconcile/index.html>`_
            - **WeBankBlockchain-Governance 多方治理协作组件**：
               - Governance-Account 账户治理组件： `[GitHub] <https://github.com/WeBankBlockchain/Governance-Account>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Governance-Account>`_  `[文档] <https://governance-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-Governance-Acct/index.html>`_
               - Governance-Authority 权限治理组件：`[GitHub] <https://github.com/WeBankBlockchain/Governance-Authority>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Governance-Authority>`_  `[文档] <https://governance-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-Governance-Auth/index.html>`_
               - Governance-Key 私钥管理组件： `[GitHub] <https://github.com/WeBankBlockchain/Governance-Key>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Governance-Key>`_  `[文档] <https://governance-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-Governance-Key/index.html>`_
               - Governance-Cert 证书管理组件：`[GitHub] <https://github.com/WeBankBlockchain/Governance-Cert>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/Governance-Cert>`_  `[文档] <https://governance-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-Governance-Cert/index.html>`_
            - **WeEvent 基于区块链的分布式事件驱动架构**：`[GitHub] <https://github.com/WeBankBlockchain/WeEvent>`_ `[Gitee] <https://gitee.com/WeBank/WeEvent>`_  `[文档] <https://weevent.readthedocs.io/>`_
            - **WeBankBlockchain-SmartDev 区块链应用开发工具**：
               - SmartDev-Contract 智能合约库组件：`[GitHub] <https://github.com/WeBankBlockchain/SmartDev-Contract>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/SmartDev-Contract>`_  `[文档] <https://smartdev-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-SmartDev-Contract/index.html>`_
               - SmartDev-SCGP 合约编译插件：`[GitHub] <https://github.com/WeBankBlockchain/SmartDev-SCGP>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/SmartDev-SCGP>`_  `[文档] <https://smartdev-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-SmartDev-SCGP/index.html>`_
               - SmartDev-Scaffold 应用开发脚手架：`[GitHub] <https://github.com/WeBankBlockchain/SmartDev-Scaffold>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/SmartDev-Scaffold>`_  `[文档] <https://smartdev-doc.readthedocs.io/zh_CN/latest/docs/WeBankBlockchain-SmartDev-Scaffold/index.html>`_
            - **WeBankBlockchain-OpenLedger 分布式账本解决方案**：`[GitHub] <https://github.com/WeBankBlockchain/OpenLedger>`_ `[Gitee] <https://gitee.com/WeBankBlockchain/OpenLedger>`_  `[文档] <https://openledger.readthedocs.io/zh_CN/latest>`_


.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: 平台介绍

   docs/WeBASE/introduction.md
   docs/WeBASE/ChangeLOG.md

.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: 安装部署

   docs/WeBASE-Install/developer.md
   docs/WeBASE/install.md
   docs/WeBASE-Install/enterprise.md
   docs/WeBASE-Install/visual_deploy.md
   docs/WeBASE-Install/docker_install.md   
   docs/WeBASE-Install/upgrade.md
   docs/WeBASE/mirror.md

.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: WeBASE使用指南
   
   docs/WeBASE-Console-Suit/index.md
   docs/WeBASE-APP-SDK/index.md
   docs/WeBASE-Contract-Warehouse/index.md
   docs/WeBASE-Training-Class/index.md
   docs/WeBASE/quick-start.md

.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: WeBASE子系统

   docs/WeBASE-Front/index.md
   docs/WeBASE-Node-Manager/index.md
   docs/WeBASE-Web/index.md
   docs/WeBASE-Transaction/index.md
   docs/WeBASE-Sign/index.md
   docs/WeBASE-Chain-Manager/index.md
   docs/WeBASE-Stat/index.md
   docs/WeBASE-Data/index.md

.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: 更多参考资料
   
   docs/WeBASE-Console-Suit/articles/index.md
   docs/WeBASE/CONTRIBUTING.md
   docs/More/family-bucket.md
   docs/More/community.md

