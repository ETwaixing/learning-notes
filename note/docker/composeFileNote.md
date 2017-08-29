# composeFile学习笔记

学习composeFile的个人笔记，如何编写一个composeFile文件，通过该文件进行docker集群以及负载均衡

## 示例

* 1
        
        #设置composeFile的版本为3.0
        version: "3"
        services:
          web:
            #设置镜像
            image: username/repository:tag
            deploy:
              #启动5个实例实现负载均衡
              replicas: 5
              resources:
                limits:
                  #显示每个实例的使用，最多使用10%的CPU（跨所有内核）和50MB RAM内存
                  cpus: "0.1"
                  memory: 50M
              #发生故障重启容器
              restart_policy:
                condition: on-failure
            #将端口80映射到主机web的端口80
            ports:
              - "80:80"
            #指示web容器通过称为负载平衡网络共享端口
            networks:
              - webnet
        #webnet使用默认设置（这是一个负载均衡的重叠网络）来定义网络
        networks:
          webnet:
          
## composeFile命令含义

* build
        
        构建时候应用的相关配置
        
        version: '2'
        services:
          webapp:
            build:
              context: ./dir
              dockerfile: Dockerfile-alternate
              args:
                buildno: 1
                
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。docker stack命令只接受准备好的镜像
        
    * context 

            包含dockerfile目录的路径，或者是git库的url
            当提供的值是相对路径时候，被解释为相对于compose文件的位置。利用docker守护进程构建上下文
            compose将使用生成的名称构建和标记它，然后使用该镜像
            
            build:
              context: ./dir

    * dockerfile
    
            备用的dockerfile
            compose将使用dockerfile文件进行构建。还需要指定构建路径
            
            build:
              context: .
              dockerfile: Dockerfile-alternate
              
    * args
    
            添加构建参数，环境变量只能在构建过程中访问
            首先在dockerfile中指定参数：
            
            ARG buildno
            ARG password
            
            RUN echo "Build number: $buildno"
            RUN script-requiring-password.sh "$password"
            
            然后指定build键下的参数，可以传递映像或者列表
            
            build:
              context: .
              args:
                buildno: 1
                password: secret
            
            build:
              context: .
              args:
                - buildno=1
                - password=secret
                
            也可以在指定构建参数是省略该值，在这种情况下，构建的值是运行compose的环境中的值（即运行命令传递的参数）
            
            args:
              - buildno
              - password
            
            注意事项：YAML布尔值（true，false，yes，no，on，off）必须用引号括起来，这样分析器会将它们解释为字符串
            
    * cache_from
    
            可以引用已经缓存好的镜像列表
            
            build:
              context: .
              cache_from:
                - alpine:latest
                - corp/web_app:3.14
                
            注意事项：此选项是v3.2中的新增功能
            
    * labels
     
            使用docker label将元数据添加到生成的镜像。可以使用数组和字典
            最好使用反向DNS符号来防止标签与其他软件使用的标签相冲突
            
            build:
              context: .
              labels:
                com.example.description: "Accounting webapp"
                com.example.department: "Finance"
                com.example.label-with-empty-value: ""
            
            
            build:
              context: .
              labels:
                - "com.example.description=Accounting webapp"
                - "com.example.department=Finance"
                - "com.example.label-with-empty-value"
            
            注意事项：此选项是v3.3中的新增功能
                
* cap_add，cap_drop
        
        添加或删除容器功能
        
        cap_add:
          - ALL
        
        cap_drop:
          - NET_ADMIN
          - SYS_ADMIN
            
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* command 

        覆盖默认命令
        
        command: bundle exec thin -p 3000
        
        该命令也可以是一个列表，类似于dockerfile
        
        command: ["bundle", "exec", "thin", "-p", "3000"]
        
* configs

        使用每个服务configs配置在每个服务的基础上授予对配置的访问权限。 支持两种不同的语法变体
        注意事项：该配置必须已经存在或在该堆栈文件的顶级配置中定义，否则堆栈部署将失败
        
        短语法：
        短语法变体只指定配置名称。这允许容器访问配置并将其安装在/<config_name> 容器内。源名称和目标装载点都设置为配置名称。
        以下示例使用简短的语法来授予redis对my_config和my_other_config配置的服务访问权限。将值 my_config设置为文件的内容
        ./my_config.txt，并将 my_other_config其定义为外部资源，这意味着它已经在Docker中定义，无论是通过运行docker config create 
        命令还是通过其他堆栈部署。如果外部配置不存在，则堆栈部署失败并出现config not found错误
            
        version: "3.3"
        services:
          redis:
            image: redis:latest
            deploy:
              replicas: 1
            configs:
              - my_config
              - my_other_config
        configs:
          my_config:
            file: ./my_config.txt
          my_other_config:
            external: true
                
        注意事项：config定义仅在版本3.3及更高版本的组合文件格式中受支持
            
        长语法：
        长语法为服务的任务容器中如何创建配置提供了更多的粒度。
        
        source：Docker中存在的配置名称。
        target：将要挂载到服务任务容器中的文件的路径和名称。默认为/<source>未指定。
        uid和gid：数字UID或GID，它将拥有服务的任务容器中的挂载配置文件。0如果没有指定，则默认为Linux。Windows不支持
        mode：将在服务的任务容器中装载的文件的权限，以八进制符号表示。例如，0444 代表世界可读。默认是0444。配置无法写入，
        因为它们被安装在临时文件系统中，因此如果设置可写位，则忽略它。可执行位可以设置。如果您不熟悉UNIX文件权限模式，
        您可能会发现此 权限计算器 很有用。
        下面的示例设置的名称my_config，以redis_config在容器内，将模式设定为0440（组可读），并且将所述用户和组103。
        该redis服务无法访问该my_other_config 配置。
        
        version: "3.3"
        services:
          redis:
            image: redis:latest
            deploy:
              replicas: 1
            configs:
              - source: my_config
                target: /redis_config
                uid: '103'
                gid: '103'
                mode: 0440
        configs:
          my_config:
            file: ./my_config.txt
          my_other_config:
            external: true
                
* container_name 

        指定自定义容器名称，而不是生成的默认名称
        
        container_name: my-web-container
        
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* deploy

        指定与部署和运行相关的配置。只能部署到swarm和docker stack
        
        version: '3'
        services:
          redis:
            image: redis:alpine
            deploy:
              replicas: 6
              update_config:
                parallelism: 2
                delay: 10s
              restart_policy:
                condition: on-failure
                
        注意事项：仅限于版本3
        
    * endpoint_mode 
    
            为连接到群集的外部客户端指定服务发现方法
            endpoint_mode: vip - Docker为服务分配虚拟IP（VIP），作为客户端到达网络服务的“前端”。Docker在服务的客户机和
            可用的工作者节点之间路由请求，而无需客户端知道有多少个节点参与服务或其IP地址或端口。（这是默认值。）
            endpoint_mode: dnsrr - DNS循环（DNSRR）服务发现不使用单个虚拟IP。Docker为服务设置DNS条目，以便服务名称的DNS
            查询返回IP地址列表，客户端直接连接到其中一个。如果要使用自己的负载平衡器，或混合Windows和Linux应用程序，DNS
            循环是有用的。
            
            version: "3.3"
            services:
              wordpress:
                image: wordpress
                ports:
                  - 8080:80
                networks:
                  - overlay
                deploy:
                  mode: replicated
                  replicas: 2
                  endpoint_mode: vip
              mysql:
                image: mysql
                volumes:
                   - db-data:/var/lib/mysql/data
                networks:
                   - overlay
                deploy:
                  mode: replicated
                  replicas: 2
                  endpoint_mode: dnsrr
            volumes:
              db-data:
            networks:
              overlay:
              
            注意事项：仅限于版本3.3
            
    * labels
    
            指定服务的标签。这些标签只能在服务上设置，而不能在服务的任何容器上设置
            
            version: "3"
            services:
              web:
                image: web
                deploy:
                  labels:
                    com.example.description: "This label will appear on the web service"
                    
            要在容器上设置标签labels键deploy
            
            version: "3"
            services:
              web:
                image: web
                labels:
                  com.example.description: "This label will appear on all containers for the web service"
                  
    * mode
    
            global（正好一个每群节点容器）或replicated（一个指定的数量的容器）。默认是replicated
            
            version: '3'
            services:
              worker:
                image: dockersamples/examplevotingapp_worker
                deploy:
                  mode: global
                  
    * placement
    
            指定布局约束
            
            version: '3'
            services:
              db:
                image: postgres
                deploy:
                  placement:
                    constraints:
                      - node.role == manager
                      - engine.labels.operatingsystem == ubuntu 14.04
                      
    * replicated
    
            如果服务是replicated（默认），请指定在任何给定时间应运行的容器数
            
            version: '3'
            services:
              worker:
                image: dockersamples/examplevotingapp_worker
                networks:
                  - frontend
                  - backend
                deploy:
                  mode: replicated
                  replicas: 6
                  
    * resources
    
            配置资源约束。这将替换之前的设置（之前为cpu_shares，cpu_quota， cpuset，mem_limit，memswap_limit，mem_swappiness）
            每个都是一个单一的值
            
            version: '3'
            services:
              redis:
                image: redis:alpine
                deploy:
                  resources:
                    limits:
                      cpus: '0.001'
                      memory: 50M
                    reservations:
                      cpus: '0.0001'
                      memory: 20M
                      
            内存异常（OOME）
            如果服务器或容器尝试使用比系统可用的更多内存，则可能会遇到内存异常（OOME），并且容器或Docker守护程序可能被
            内核OOM杀手杀死。为了防止这种情况发生，确保应用程序在具有足够内存的主机上运行
            
    * restart_policy
    
            配置在退出时是否以及如何重新启动容器。替换restart
            
            condition：其中之一none，on-failure或any（默认：）any。
            delay：在重新启动尝试之间等待多长时间，指定为 持续时间（默认值：0）。
            max_attempts：在放弃之前尝试重新启动一个容器多少次（默认：永不放弃）。
            window：在决定重新启动成功之前等待多长时间，指定为持续时间（默认：立即决定）。
            
            version: "3"
            services:
              redis:
                image: redis:alpine
                deploy:
                  restart_policy:
                    condition: on-failure
                    delay: 5s
                    max_attempts: 3
                    window: 120s
                    
    * update_config
    
            配置服务应该如何更新。用于配置滚动更新
            
            parallelism：一次更新的容器数。
            delay：更新一组容器之间等待的时间。
            failure_action：如果更新失败，该怎么办？其中之一continue，，rollback或pause （默认：）pause。
            monitor：每个任务更新之后的持续时间以监视失败(ns|us|ms|s|m|h)（默认为0）。
            max_failure_ratio：在更新期间容忍的故障率。
            
            version: '3'
            services:
              vote:
                image: dockersamples/examplevotingapp_vote:before
                depends_on:
                  - redis
                deploy:
                  replicas: 2
                  update_config:
                    parallelism: 2
                    delay: 10s
                    
* depends_on

        服务之间的快速依赖，这有两个作用：
        docker-compose up将依依次顺序启动服务。在下面的例子中，db并redis会开始之前web。
        docker-compose up SERVICE将自动包含SERVICE的依赖关系。在下面的例子中，docker-compose up web也将创建和启动db和redis。
        
        version: '3'
        services:
          web:
            build: .
            depends_on:
              - db
              - redis
          redis:
            image: redis
          db:
            image: postgres

        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* dns

        自定义DNS服务器。可以是单个值或列表
        
        dns: 8.8.8.8
        dns:
          - 8.8.8.8
          - 9.9.9.9
          
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* dns_search

        自定义DNS搜索域。可以是单个值或列表
        
        dns_search: example.com
        dns_search:
          - dc1.example.com
          - dc2.example.com
          
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* tmpfs

        在容器内安装临时文件系统。可以是单个值或列表
        
        tmpfs: /run
        tmpfs:
          - /run
          - /tmp
          
        注意事项：在版本3的Compose文件用群组模式部署堆栈时候，将忽略此选项。
        
* entrypoint

        覆盖默认的入口点
        
        entrypoint: /code/entrypoint.sh
        
        entrypoint也可以是一个列表，类似于 dockerfile：
        
        entrypoint:
            - php
            - -d
            - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
            - -d
            - memory_limit=-1
            - vendor/bin/phpunit
            
        注意事项：设置entrypoint将使用ENTRYPOINTDockerfile指令覆盖服务映像上的任何默认入口点集，并 清除图像上的任何默认
        命令 - 这意味着如果CMD Dockerfile中有指令，则将忽略它
        
* env_file

        从文件添加环境变量。可以是单个值或列表。
        如果您指定了一个Compose文件docker-compose -f FILE，则路径 env_file相对于文件所在的目录。
        在环境部分中 声明的环境变量覆盖这些值 - 即使这些值为空或未定义也是如此
        
        env_file: .env
        
        env_file:
          - ./common.env
          - ./apps/web.env
          - /opt/secrets.env
          
        Compose期望env文件中的每一行都是VAR=VAL格式。以#（即注释）开始的行将被忽略，空白行也将被忽略
        
        # Set Rails/Rack environment
        RACK_ENV=development
        
        注意事项：如果您的服务指定构建选项，则在构建期间将不会自动显示在环境文件中定义的变量。使用args子选项build来定义
        构建时环境变量
        
* environment

        添加环境变量。您可以使用数组或字典。任何布尔值; true，false，yes no，需要用引号括起来，以确保它们不被YML解析器转
        换为True或False。
        仅具有密钥的环境变量将被解析为其运行的机器上的值，这对于秘密或主机特定值有帮助
        
        environment:
          RACK_ENV: development
          SHOW: 'true'
          SESSION_SECRET:
        
        environment:
          - RACK_ENV=development
          - SHOW=true
          - SESSION_SECRET
          
        注意事项：如果您的服务指定构建选项，environment则在构建期间不会自动显示定义的变量。使用args子选项build来定义构建
        时环境变量
        
* expose

        暴露端口而不将它们发布到主机 - 它们只能被链接服务访问。只能指定内部端口
        
        expose:
         - "3000"
         - "8000"
        
* external_links

        连接到这个docker-compose.yml或甚至外面的Compose之外的容器，特别是对于提供共享或公共服务的容器。 在指定容器名称和
        链接别名（）时，external_links遵循类似于legacy选项的语义。linksCONTAINER:ALIAS
        
        external_links:
         - redis_1
         - project_db_1:mysql
         - project_db_1:postgresql
         
         注意事项：使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
         
* extra_hosts

        添加主机名映射。使用与docker客户端--add-host参数相同的值
        
        extra_hosts:
         - "somehost:162.242.195.82"
         - "otherhost:50.31.209.229"
         
         将在/etc/hosts此服务的内部容器中创建具有ip地址和主机名的条目，例如：
         
         162.242.195.82  somehost
         50.31.209.229   otherhost
         
* image

        指定要启动容器所使用的镜像。可以是存储库/标签或部分映像ID
        
        image: redis
        image: ubuntu:14.04
        image: tutum/influxdb
        image: example-registry.com:4000/postgresql
        image: a4bc65fd
        
        如果图像不存在，Compose尝试拉它，除非您还指定了构建，在这种情况下，它使用指定的选项构建它，并使用指定的标签进行标记
        
* labels

        使用Docker标签将元数据添加到容器。您可以使用数组或字典。
        建议您使用反向DNS符号来防止标签与其他软件使用的标签相冲突。
        
        labels:
          com.example.description: "Accounting webapp"
          com.example.department: "Finance"
          com.example.label-with-empty-value: ""
        
        labels:
          - "com.example.description=Accounting webapp"
          - "com.example.department=Finance"
          - "com.example.label-with-empty-value"
        
* links

        链接到另一个服务中的容器。既可以指定服务名称，也可以指定链接别名（SERVICE:ALIAS）或服务名称
        
        web:
          links:
           - db
           - db:database
           - redis
           
        链接服务的容器将以与别名相同的主机名或如果未指定别名的服务名称可访问。
        链接还以与depends_on相同的方式表示服务之间的依赖关系 ，因此它们确定服务启动的顺序。
        注意事项：使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
        
* logging

        记录该服务的配置
        
        logging:
          driver: syslog
          options:
            syslog-address: "tcp://192.168.0.42:123"
            
* network_mode

        网络模式。使用与docker客户端--net参数相同的值，加上特殊格式service:[service name]
        
        network_mode: "bridge"
        network_mode: "host"
        network_mode: "none"
        network_mode: "service:[service name]"
        network_mode: "container:[container name/id]"
        
        注意事项：使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
        
* networks

        别名
        网络上此服务的别名（替代主机名）。同一网络上的其他容器可以使用服务名称或此别名来连接到其中一个服务的容器。
        既然aliases是网络范围，同一个服务可以在不同的网络上有不同的别名。
        在下面的例子中，提供了三种服务（web，worker，和db），其中两个网络（沿new和legacy）。该db服务是在到达的主机名db
        或database上new网络，并db或mysql将上legacy网络。
        
        version: '2'
        services:
          web:
            build: ./web
            networks:
              - new
          worker:
            build: ./worker
            networks:
              - legacy
          db:
            image: mysql
            networks:
              new:
                aliases:
                  - database
              legacy:
                aliases:
                  - mysql
        networks:
          new:
          legacy:
                
        IPV4_ADDRESS，IPV6_ADDRESS
        在加入网络时为该服务指定容器的静态IP地址。
        顶级网络部分中的相应网络配置必须具有ipam覆盖每个静态地址的子网配置的块。如果需要IPv6寻址，则enable_ipv6必须设置该选项。
        一个例子：
        
        version: '2.1'
        services:
          app:
            image: busybox
            command: ifconfig
            networks:
              app_net:
                ipv4_address: 172.16.238.10
                ipv6_address: 2001:3984:3989::10
        networks:
          app_net:
            driver: bridge
            enable_ipv6: true
            ipam:
              driver: default
              config:
              - subnet: 172.16.238.0/24
              - subnet: 2001:3984:3989::/64
        
        注意事项：全网域别名可由多个容器共享，甚至由多个服务共享。如果是，则不能保证名称解析到的哪个容器。

* pid
        
        pid: "host"
                
        将PID模式设置为主机PID模式。这将打开容器与主机操作系统之间的共享PID地址空间。使用此标志启动的容器将能够访问和
        操纵裸机机器名称空间中的其他容器，反之亦然。
        
* ports

        暴露端口
        
        短语法：
        既可以指定ports（HOST:CONTAINER），也可以指定容器端口（将选择一个随机的主机端口）。
        
        ports:
         - "3000"
         - "3000-3005"
         - "8000:8000"
         - "9090-9091:8080-8081"
         - "49100:22"
         - "127.0.0.1:8001:8001"
         - "127.0.0.1:5000-5010:5000-5010"
         - "6060:6060/udp"
        
        注意事项：以HOST:CONTAINER格式映射端口时，使用低于60的容器端口时，可能会遇到错误的结果，因为YAML会将格式的数字
        解析xx:yy为六进制（基数为60）。因此，我们建议您始终将端口映射明确指定为字符串。
        
        长语法：
        长格式语法允许配置不能以简短形式表达的其他字段。
        target：容器内的端口
        published：公开的港口
        protocol：端口协议（tcp或udp）
        mode：host用于在每个节点上发布主机端口，或者ingress为将负载均衡的群模式端口。
        
        ports:
          - target: 80
            published: 8080
            protocol: tcp
            mode: host
            
        注意事项：v3.2中的长语法是新的
        
* secrets

        使用每个服务secrets 配置在每个服务的基础上授予访问权限。支持两种不同的语法变体。
        
        短语法：
        短语法变体仅指定秘密名称。这允许容器访问秘密，并将其安装在/run/secrets/<secret_name> 容器内。源名称和目标挂载点
        都设置为密码名称
        以下示例使用简短的语法来授予redis对my_secret和my_other_secret秘密的服务访问权限。将值 my_secret设置为文件的内容
        ./my_secret.txt，并将 my_other_secret其定义为外部资源，这意味着它已经在Docker中定义，无论是通过运行docker secret create 
        命令还是通过其他堆栈部署。如果外部机密不存在，则堆栈部署失败，并出现secret not found错误。
        
        version: "3.1"
        services:
          redis:
            image: redis:latest
            deploy:
              replicas: 1
            secrets:
              - my_secret
              - my_other_secret
        secrets:
          my_secret:
            file: ./my_secret.txt
          my_other_secret:
            external: true
            
        长语法：
        长语法在服务的任务容器中如何创建秘密提供了更多的粒度。
        source：Docker中存在的秘密的名称。
        target：将要安装在/run/secrets/服务的任务容器中的文件的名称。默认为source未指定。
        uid和gid：/run/secrets/在服务的任务容器中拥有文件的数字UID或GID 。两者默认为0未指定。
        mode：将以/run/secrets/ 八进制符号方式挂载在服务的任务容器中的文件的权限。例如，0444 代表世界可读。Docker 1.13.1
        中的默认值是0000，但将来会0444在。秘密不能被写入，因为它们被安装在临时文件系统中，所以如果设置了可写位，它将被忽
        略。可执行位可以设置。如果您不熟悉UNIX文件权限模式，您可能会发现此 权限计算器 很有用。
        以下示例设置容器内的my_secretto的名称redis_secret，将模式设置为0440（组可读），并将用户和组设置为103。该redis服务
        无法访问该my_other_secret 秘密。
        
        version: "3.1"
        services:
          redis:
            image: redis:latest
            deploy:
              replicas: 1
            secrets:
              - source: my_secret
                target: redis_secret
                uid: '103'
                gid: '103'
                mode: 0440
        secrets:
          my_secret:
            file: ./my_secret.txt
          my_other_secret:
            external: true
            
        您可以授予对多个秘密的服务访问权限，您可以混合使用长和短的语法。定义秘密并不意味着授予服务访问权限。
        注意事项：该秘密必须已经存在或 在secrets 该堆栈文件的顶级配置中定义，否则堆栈部署将失败。
        
* security_opt

        覆盖每个容器的默认标签方案。
        
        security_opt:
          - label:user:USER
          - label:role:ROLE
          
        注意事项：在 使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项 。
        
* stop_grace_period

        指定stop_signal在发送SIGKILL之前尝试停止集装箱等待多长时间，如果它不处理SIGTERM（或任何停止信号已指定 ））。指定
        为持续时间
        
        stop_grace_period: 1s
        stop_grace_period: 1m30s
        
        默认情况下，stop在发送SIGKILL之前，等待10秒钟才能使该容器退出。
        
* stop_signal

        设置一个替代信号来停止容器。默认情况下stop使用SIGTERM。设置替代信号stop_signal将导致 stop发送该信号。
        
        stop_signal: SIGUSR1
        
        注意事项：在 使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
        
* sysctls

        在容器中设置的内核参数。您可以使用数组或字典
        
        sysctls:
          net.core.somaxconn: 1024
          net.ipv4.tcp_syncookies: 0
        
        sysctls:
          - net.core.somaxconn=1024
          - net.ipv4.tcp_syncookies=0
          
        注意事项：在 使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
        
* ulimits

        覆盖容器的默认ulimits。您可以将单个限制指定为整数或软/硬限制作为映射。
        
        ulimits:
          nproc: 65535
          nofile:
            soft: 20000
            hard: 40000
            
* userns_mode

        userns_mode: "host"
        
        如果Docker守护程序配置了用户名空间，则禁用此服务的用户命名空间
        注意事项：在 使用（版本3）Compose文件以群组模式部署堆栈时，将忽略此选项。
        
* volumes

        挂载主机路径或命名卷，指定为服务的子选项。
        您可以将主机路径作为单个服务的定义的一部分进行安装，并且不需要在顶级volumes密钥中定义它。
        但是，如果要跨多个服务重用卷，请在顶级volumes密钥中定义一个命名卷。使用具有服务，群集和堆栈文件的命名卷
        该实施例显示了一个名为体积（mydata）正在使用的web服务，和一个绑定安装为一个单一的服务（下第一路径定义db的服务 
        volumes）。该db服务还使用一个名为dbdata（称为db服务的第二个路径volumes）的命名卷，但是使用旧的字符串格式来定义它，
        用于安装一个命名卷。命名卷必须列在顶级 volumes密钥下，如图所示。
        
        version: "3.2"
        services:
          web:
            image: nginx:alpine
            volumes:
              - type: volume
                source: mydata
                target: /data
                volume:
                  nocopy: true
              - type: bind
                source: ./static
                target: /opt/app/static
        
          db:
            image: postgres:latest
            volumes:
              - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
              - "dbdata:/var/lib/postgresql/data"
        
        volumes:
          mydata:
          dbdata:
          
        注意事项：顶级 卷密钥定义了一个命名卷，并从每个服务的volumes列表中引用它。这将替代volumes_from早期版本的Compose
        文件格式。有关卷的一般信息，请参阅使用卷和卷插件。
        
        短语法：
        可选地指定主机（HOST:CONTAINER）或访问模式（HOST:CONTAINER:ro）的路径。
        您可以在主机上挂载相对路径，该路径将相对于正在使用的Compose配置文件的目录进行扩展。相对路径应该始于.或..。
        
        volumes:
          # Just specify a path and let the Engine create a volume
          - /var/lib/mysql
          # Specify an absolute path mapping
          - /opt/data:/var/lib/mysql
          # Path on the host, relative to the Compose file
          - ./cache:/tmp/cache
          # User-relative path
          - ~/configs:/etc/configs/:ro
          # Named volume
          - datavolume:/var/lib/mysql
        
        长语法：
        长格式语法允许配置不能以简短形式表达的其他字段。
        type：安装类型volume或bind
        source：安装的源，主机上的绑定安装路径，或顶级密钥中定义的卷的名称 volumes
        target：容器中将要安装卷的路径
        read_only：将卷设置为只读的标志
        bind：配置其他绑定选项
        propagation：用于绑定的传播模式
        volume：配置其他卷选项
        nocopy：在创建卷时禁止从容器复制数据的标志
        
        version: "3.2"
        services:
          web:
            image: nginx:alpine
            ports:
              - "80:80"
        networks:
          webnet:
        volumes:
          - type: volume
            source: mydata
            target: /data
            volume:
              nocopy: true
          - type: bind
            source: ./static
            target: /opt/app/static
        
        注意事项：v3.2中的长语法是新的
        
* restart

        no是默认的重新启动策略，它不会在任何情况下重新启动容器。当always指定时，容器总是重新启动。on-failure如果退出代码
        指示出错故障，则该 策略将重新启动容器。
        
        restart: "no"
        restart: always
        restart: on-failure
        restart: unless-stopped
        
        
        
        
        















