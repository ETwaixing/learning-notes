# docker学习笔记

学习docker的个人笔记，使用docker命令进行操作

## docker基本操作命令

* 随时随地可以在命令后面 --help 查看帮助

* docker inspect [id|name|repository]   查看容器镜像的详细信息

* docker -v 返回docker的版本信息。备注：执行这个命令不需要docker守护进程启动，但是其他docker命令基本上都需要docker守护进程已经启动

* docker info/version  查看docker的设置信息以及版本信息

* docker system prune  清理docker没有使用的data(所有停止中的容器、没有使用的网关、挂起的镜像以及构建的缓存)

* docker镜像相关操作命令

        docker images 查看镜像列表
        	-a 查看所以镜像列表（包括已隐藏的） 
        	
        docker rmi [镜像id|repository] 删除镜像
        
        docker pull [repository]:[tag] 拉取远程仓库镜像（公共仓库以及登陆的私人仓库）   
        
        docker push [username]/[rep]:[tag] 将本地镜像提交到远程仓库（需登录dockerhub）
        
        docker build [path|url|.] 利用本地文件构建本地镜像（实质利用Dockerfile）
        	-f 指定使用的dockerfile路径	
        	--rm 设置镜像成功后删除中间容器
        	--no-cache 创建镜像的过程不使用缓存
        	-t 创建镜像的repository以及tag
        
        docker tag [镜像1-id|repository]:[tag] [镜像2-id|repository]:[tag] 新生成一个版本镜像
        
        docker search 镜像搜索 
        
        docker save -o [url] [repository]:[tag] 将镜像文件生成tar文件
        
        docker import [url] [repository]:[tag] 将本地tar文件生成镜像文件
        	-m 提交时的说明文字
        
        docker history [repository]:[tag] 查看镜像生成历史
    
* docker容器相关操作命令

        docker start/stop/restart [容器id|name] 启动/停止/重启容器
        
        docker rm [容器id|name] 删除容器
        	-f 强制删除容器
        	-v 删除与容器相关的卷
        	-l 移除容器间的网络连接，而非容器本身
        				
        docker ps 查看容器列表
        	-a 显示所有的容器，包括未运行的
        
        docker logs [容器id|name] 查看容器的日志
        	-f 跟踪日志输出
        	-t 显示时间戳
        
        docker create [repository]:[tag] 创建一个新容器但不启动
        
        docker run [repository]:[tag] 创建一个新容器并运行一个命令
        	-d 后台运行容器
        	-i 以交互模式运行容器，通常与-t同时使用
        	-t 为容器重新分配一个伪输入终端，通常与-i同时使用
        	--name 为容器设置名字
        	-p 进行端口的映射
        
        docker exec 在运行容器中执行命令（退出容器终端后，容器运行不会停止）
        	-i 以交互模式运行容器，通常与-t同时使用
        	-t 为容器重新分配一个伪输入终端，通常与-i同时使用
        
        docker attach 连接到正在运行中的容器（退出容器终端后，容器运行停止）
        	--sig-proxy=false 可以确保退出时候不关闭容器
        
        docker cp [路径1] [路径2] 可以让主机与容器之间文件相互拷贝
        主机容器路径  ID/NAME:容器路径
        
        docker rename [oldname] [newname] 容器重新命名
        
        docker diff [CONTAINER] 查询容器内变换情况
        
* docker数据卷相关操作命令

        -v [主机url]:[容器主机url] 将主机的url挂载到容器上
        
        --volume-from [容器id|name] 将数据容器挂载到其他容器上
        
        docker volume 数据卷操作
        	ls 查看数据卷列表
        	prune 删除没有被使用的数据卷
        	rm 删除指定数据卷

## docker三剑客（）

* docker集群相关命令操作---docker swarm

        docker swarm 集群操作
            init 初始化集群，启用集群模式
            join 加入集群节点或者管理者
            join-token 管理join tokens
            leave 离开集群
            
        docker node 节点操作
            inspect 查看节点的详细信息
            ls 查看集群中的节点列表
            ps [id] 查看节点中运行的任务，不填默认为通用的节点
            rm [id] 删除某个节点
            
        docker stack 堆栈操作
            deploy 部署一个新的堆栈或者更新之前的堆栈
            ls 查看堆栈列表
            ps [id] 查看堆栈运行的任务
            rm [id] 删除某个堆栈
            services [id] 查看堆栈中服务列表

* docker本地环境模拟以及虚拟机的设置---docker-machine

        docker-machine create [name] 利用docker-machine对本地环境进行虚拟机的创建及其环境的虚拟化
            -d [driver] 驱动设置（win10系统为Hyper-V）
            --hyperv-virtual-switch [switch name] 设置要选择的虚拟交换机
        注意事项：在win10下创建虚拟机时，需要以管理员的身份运行cmd再进行命令的执行
        
        docker-machine ssh [machine name] [command] 通过ssh连接到某个vm，并且执行输入的命令（命令需要用引号包含）
        注意事项：不输入命令可以直接连接到虚拟机终端，输入exit将退出终端
        
        docker-machine ls 查看所设置的虚拟机列表以及相关信息
        
        docker-machine env [machine name] 查看某个虚拟机的环境信息
        
        docker-machine ip [machine name] 查看虚拟机对外的ip
        
        docker-machine kill [machine name] 杀掉一个虚拟机进程
        
        docker-machine restart [machine name] 重启一个虚拟机
        
        docker-machine rm [machine name] 删除一个虚拟机
        
        docker-machine scp [machine name]:[path] [machine name]:[path] 在两个虚拟机之间相互拷贝文件数据（也可以从本地
        向虚拟机拷贝文件）
        注意事项：本地环境拷贝数据时候，如果问window环境由于文件类型的不同需要介入其他的工具执行该命令（例如git bash）
        
        docker-machine start [machine name] 启动一个虚拟机
        
        docker-machine status [machine name] 查看一个虚拟机状态
        
        docker-machine stop [machine name] 停止一个虚拟机
        
* docker服务相关设置---docker-compose
        
        docker-compose build [service name] 创建或重新创建服务所使用的镜像
        
        docker-compose kill [service name] 通过向容器发送SIGKILL信号强行停止服务
        
        docker-compose logs [service name] 展示服务的日志
        
        docker-compose pause [service name] 暂停服务
        
        docker-compose unpause [service name] 恢复服务
        
        docker-compose port [service name] [port] 查看服务的端口被映射到了主机的哪个端口
        
        docker-compose ps 用于显示当前项目下的的容器
            -a 显示本机上所有的容器
        注意事项：执行此命令必须cd到项目的根目录，否则会发生错误。
        
        docker-compose pull [service name] 拉去服务依赖的镜像
        
        docker-compose restart [service name] 重启某个服务的所有容器
        
        docker-compose rm [service name] 删除停止的容器
            -f 强制删除
            -v 删除容器相关的数据卷
            
        docker-compose run [service name] 新建一个容器，配置等同于service
        
        docker-compose scale [service name=number]... 指定某一个服务启动的容器个数
        注意事项：当容器有到主机的端口映射时，因为所有容器都指向一个宿主机的端口，所以只能启动一个容器，其他都会失败
        
        docker-compose start [service name] 运行某个服务的所有容器
        
        docker-compose stop [service name] 停止某个服务的所有容器
        
## 其他

        待续...