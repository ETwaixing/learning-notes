# jenkins 学习笔记

学习jenkins持续集成部署的个人笔记

## jenkins 作用

        自动化编译测试打包部署到服务器上，可集成多种插件 Maven git docker 邮件等。目前实现为：通过向git某个分支push代码
        触发，拉去git远程库中的代码，通过jenkins一系列的配置进行处理，maven打包进行编辑，docker通过其中的dockerFile文件
        进行镜像的构建，push镜像以及进行镜像容器的启动完成部署。

## jenkins的安装启动

        jenkins有多种安装的方式，最快以及最为方便的就是通过java启动。jenkins本身就是通过java编写的，而我之后的项目也需要
        用到java的环境，所以这边只需要通过 java -jar jenkins.war启动即可，十分方便。启动之后为了安全会生成一个初始密码。第
        一次进入jenkins页面时候需要进行设置，然后选择建议的插件下载还是已定义插件下载（初学jenkins,选择的是默认建议的插件）。

## jenkins 基本设置（系统设置）
    
        对jenkins的系统设置进行说明
    
* 系统设置（全局设置&路径）

        maven设置：这边可以进行全局的maven_opts参数设置 jar包的存储地址 可并发进行编译的项目数量设置等
        E-mail设置：设置发邮件人的SMTP Server 邮件的格式内容相关参数等等（注意：SMTP Server的密码为授权密码）
        Git设置：全局设置
        ...
        
* Configure Global Security(安全策略配置)

        启用安全：端点随机生成
        去除浏览器的记住我功能
        jenkins专有用户数据库：设置不允许用户注册
        授权策略：Role-Based Strategy（需要安装插件---Role-based Authorization Strategy）
        
* Configure Credentials（凭证设置）
        
       规定凭证的范围
       
* Global Tool Configuration（全局的工具配置）

        JDK：设置java jdk别名 java_home（最好自己安装jdk）
        GIT：设置git 别名 git path(最好自己安装git)
        MAVEN：设置maven别名 maven_home(最好自己安装maven)
        DOCKER: 设置docker别名 docker root（最后自己安装docker）
        
* 读取设置

        放弃当前内存中所有的设置信息并从配置文件中重新获取 仅用于手动修改配置文件时重新读取配置
        
* 管理插件

        目前我使用较多的插件：
        Email Extension Plugin：邮箱配置的插件
        Git&Github：一系列相关的插件
        JUnit Plugin：java单元测试的插件
        Maven Integration plugin：maven相关插件（新建maven项目时必要的插件）
        Role-based Authorization Strategy：角色权限的插件
        Docker：docker一系列相关的插件
        SSH Slaves plugin：jenkins ssh自身的密钥节点插件
        Workspace Cleanup Plugin：工作空间清空插件
        
* 系统信息

        显示系统环境信息以帮助解决问题
        包括：系统属性、环境变量以及插件的相关信息
        
* System Log

        系统日志从java.util.logging捕获jenkins的相关信息
        
* 负载统计
        
        检测资源的利用情况，是否需要多台计算机进行构建（利用节点进行多台计算机的共同工作）
        
* Jenkins CLI

        使用命令行或者脚本对jenkins进行管理
        
* 脚本命令行

        执行用于管理或故障探测或诊断的任意脚本命令
        
* 管理节点

        添加、删除、控制和监控系统运行任务的节点。（通过这个功能设置多台计算器共同工作，分布式）
        
* Manage and Assign Roles（用户的权限管理设置）

        用户的权限设置以及管理（需要安装插件---Role-based Authorization Strategy）
        
    * Manage Roles（角色设置）
    * Assign Roles（用户角色分配）
    * Role Strategy Macros（角色宏定义的拓展）
    
* 管理用户

        创建/删除/修改jenkins用户
        
* About Jenkins

        jenskin的相关信息
        
* 其他

        ...
        
## 新建jenkins job

        创建maven项目
        
* General

        设置项目名称描述，设置丢弃旧的构建---定时自动清理保持磁盘的空间大小，设置保持构建天数5天，最大构建数3天，发布包
        保留天数1天，发布包最大保留构建数1个 
        
* 源码管理

        利用git进行源码管理
        
    * Repositories
    
            指定git资源库 分两种http以及SSH 需要对应不同的凭证 http对于账号密码的凭证，SSH对于密钥的凭证，凭证需要进行设置
            添加。
            
    * Branches to build
    
            分支代码的拉取，分支的监控 可以根据规则进行不同的设置
            
    * Additional Behaviours
    
            设置监听的目录文件 适用于多模块的项目 只监听这个项目下的某个模块
            
* 构建触发器

        设置触发的条件：Poll SCM 每隔一段时间去检查该项目下监听的分支下监听的目录的代码push情况
        Poll SCM * * * * * 设置为每分钟检查一次
        
* 构建环境

        构造环境的一些相关设置
        
* Pre Steps

        构造前的操作 执行shell脚本之类的
        
* Build

        maven构建的操作设置 
        Root POM：pom.xml所在路径  Goals and options：mvn命令操作等
         
* Post Steps

        构造完成的操作 执行shell脚本之类的 我这边用来进行docker的构建镜像上传之类
        
* 构建设置

* 构建后操作

        job最后的操作，可用于邮件的发送以及其他的相关操作等
        
* 暂无

        ...
        
## 视图的设置

        视图的简单设置：名称、描述、显示的过滤规则、显示的具体job以及显示的信息栏设置等，主视图的设置需要在系统设置里面的
        全局设置里面进行之定义的配置。
        
## Credentials

        凭证设置，ssh、账号密码的凭证以及DockerHost等相关的设置
        
## jenkins其他设置

        如果网络下载插件较慢可以通过上传本地插件来进行安装安装
        
## 暂无

        ...