# dockerFile学习笔记

学习dockerFile的个人笔记，如何编写一个dockerFile文件，通过该文件构建docker镜像

## 注意事项

* 命令大写  命令后的内容小写

##  事例

* 1

        #  设置运行时父镜像为python
        FROM python:2.7-slim
        #  设置工作目录为/app
        WORKDIR /app
        #  将当前目录的内容添加到容器的/app中
        ADD . /app
        #  安装在requirement.txt中所需要的包
        RUN pip install -r requirements.txt
        #  设置容器对外的端口为80
        EXPOSE 80
        #  定义环境变量
        ENV NAME World
        #  当容器启动的时候运行app.py
        CMD ["python","app.py"]
        
* 2
        
        #  设置运行时的父镜像为java8,alpine为精简版
        FROM java:8-jre-alpine
        #  将当前目录下的target目录下的*.jar添加到app.jar
        ADD target/*.jar app.jar
        #  运行容器的sh，让它执行-c touch /app.jar命令
        RUN sh -c 'touch /app.jar'
        #  运行java -jar app.jar 设置参数-Djava.security.egd=file:/dev/./urandom
        ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
        
## dockerfile指令含义

* ENV

        环境变量
        ENV abc=hello 或者ENV abc hello
        下文可以通过$abc或者${abc}引用
        
* FROM

        设置基本镜像（有效的dockerfile必须从from开始）
        FROM <image> [AS <name>]   或者  FROM <image>[:<tag>] [AS <name>]  或者 FROM <image>[@<digest>] [AS <name>]
        ARG 指令可以在FROM前面设置FROM镜像的版本号
        ARG VERSION=latest
        FROM busybox:$VERSION
        
* RUN

        执行任何命令并提交结果
        RUN <command>（shell形式，命令在shell中运行，默认情况下/bin/sh -c在Linux或cmd /S /CWindows上）
        RUN ["executable", "param1", "param2"]（执行表单）
        
* CMD

        不要混淆RUN使用CMD。RUN实际上运行命令并提交结果; CMD在构建时不执行任何操作，而是指定图像的预期命令
        CMD ["executable","param1","param2"]（exec表单，这是首选表单）
        CMD ["param1","param2"]（作为ENTRYPOINT的默认参数）
        CMD command param1 param2（shell形式）
        
* LABEL

        定义镜像的标签
        LABEL <key>=<value> <key>=<value> <key>=<value> ...
        
* EXPOSE

        设置容器内部监听的端口号
        EXPOSE <port> [<port>...]
        
* ADD

        将复制新文件，目录或者远程文件url到镜像指定路径中
        ADD <src>... <dest>
        ADD ["<src>",... "<dest>"] （此窗体是包含空格的路径所必需的）
        <src>可以指定多个资源，但如果它们是文件或目录，则它们必须相对于正在构建的源目录（构建的上下文）
        
* COPY

        该COPY指令将复制新文件或目录，并将其添加到该路径上容器的文件系统。
        COPY <src>... <dest>
        COPY ["<src>",... "<dest>"] （此窗体是包含空格的路径所必需的）
        
* ENTRYPOINT

        允许您配置将作为可执行文件运行的容器
        ENTRYPOINT ["executable", "param1", "param2"] （exec表格，首选）
        ENTRYPOINT command param1 param2 （shell形式）
        
* VOLUME

        该VOLUME指令将创建具有指定名称的安装点，并将其标记为从本机主机或其他容器保存外部安装的卷
        VOLUME ["/data"]
        
* USER

        该USER指令设置用户名（或UID）和可选的用户组（或GID）在运行图像时使用
        USER <user>[:<group>]
        USER <UID>[:<GID>]
        
* WORKDIR

        工作目录
        WORKDIR /path/to/workdir
        
* ARG

        该ARG指令定义了用户可以docker build使用该--build-arg <varname>=<value> 标志使用命令在构建时传递给构建器的变量
        ARG <name>[=<default value>]
        如果用户指定了在Dockerfile中未定义的构建参数，则构建会输出警告。
        
* ONBUILD

        该ONBUILD指令在将图像用作另一构建的基础时，将更多时间执行的触发指令添加到图像中
        ONBUILD [INSTRUCTION]

* STOPSIGNAL

        该STOPSIGNAL指令设置将发送到容器的系统调用信号退出
        STOPSIGNAL signal
        
* HEALTHCHECK

        该HEALTHCHECK指令告诉Docker如何测试一个容器来检查它是否仍在工作。即使服务器进程仍在运行，这样可以检测出一系列无限循环的Web服务器，无法处理新连接的情况
        HEALTHCHECK [OPTIONS] CMD command （通过在容器内运行命令检查容器的健康状况）
        HEALTHCHECK NONE （禁用从基本图像继承的任何healthcheck）
        
* SHELL

        该SHELL指令允许覆盖命令shell形式的默认shell被覆盖
        SHELL ["executable", "parameters"]
        
* 其他

        待续...