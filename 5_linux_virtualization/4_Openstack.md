# 使用Kolla部署Pike版本的OpenStack-allinone云平台

1. openstack 概述
5. Kolla 概述和 openstack 所有结点 linux 系统初始配置
6. 安装 kolla-ansible
7. 自定义 kolla-ansible 安装 openstack 的相关配置文件
8. 开始基于 kolla-ansible 安装 openstack 私有云

# 1. openstack概述

OpenStack是一个NASA美国国家航空航天局和Rackspace合作研发的，以Apache许可证授权，并且是一个自由软件和开放源代码项目。

Open是开放，Stack则是堆砌之意，合起来就是将众多的功能服务堆积起来的集合，让人们通过Openstack云计算项目，能够将诸如计算能力、存储、网络和软件等资源抽象成服务，以便让用户可以通过互联网远程来享用，付费的形式也变得因需而定，调整方便，拥有极强的虚拟可扩展性，是公共和私有云的建设与管理软件中的优秀开源项目。

OpenStack是一个云平台管理的项目，它不是一个软件。这个项目由几个主要的组件组合起来完成一些具体的工作。OpenStack是一个旨在为公共及私有云的建设与管理提供软件的开源项目。它的社区拥有超过130家企业及1350位开发者，这些机构与个人都将OpenStack作为基础设施即服务（简称IaaS）资源的通用前端。
- IaaS（Infrastructure as a Service），即基础设施即服务
  - 消费者通过Internet 可以从完善的计算机基础设施获得服务。这类服务称为基础设施即服务。基于 Internet 的服务（如存储和数据库）是 IaaS的一部分。 比如： 在腾讯云上买一台云主机（8个CPU，32G，5T硬盘云主机等）
- PaaS是Platform-as-a-Service的缩写，意思是开发即。 把平台作为一种服务提供的商业模式。例：OpenShift是红帽的云开发平台即服务（PaaS）
  - OpenShift是自由和开放源码的云计算平台，它可以使开发人员能够创建、测试和运行他们的应用程序，并且可以把它们部署到云中。Openshift广泛支持多种编程语言和框架，如Java，Ruby和PHP等。另外它还提供了多种集成开发工具如Eclipse integration，JBoss Developer Studio和 Jenkins等。OpenShift 基于一个开源生态系统为移动应用，数据库服务等，提供支持。
- SaaS是Software-as-a-Service的简称。docs.google.com或 微软的office  365







kolla基于docker的安装方式8G内存就够了，要是按照社区方法安装，至少12G，都不一定够用
[](https://i.loli.net/2019/03/21/5c93b3f3ac779.png)