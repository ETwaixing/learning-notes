# Linux学习笔记

学习linux的个人笔记，使用linux命令进行操作

## Linux约定

* linux系统约定不同类型文件默认的颜色
    * 白色 表示普通文件
    * 蓝色 表示目录
    * 红色 表示压缩文件
    * 绿色 表示可执行文件
    * 浅蓝色 表示链接文件
    * 红色闪烁 表示链接文件有问题
    * 黄色 表示设备文件
    * 灰色 表示其他文件
    
* 向linux传输文件
    * 共享文件
    * 挂载（包括虚拟机的挂载，win10子系统的自动挂载以及docker容器的挂载）
    * WMware Toll
    * 使用winSCP工具等

## Linux命令

* 切换到root用户 su

* 随时随地可以在命令后面 --help 查看帮助

* linux基本命令
        
    * 常用命令
        
            ls 查看目录中的文件
                -l 显示文件和目录的详细信息
                -a 显示所有文件（包括隐藏文件）
            mkdir 创建新目录
            cd 切换目录
            touch 创建空文件
            echo 打印指定的变量值
            cat 查看文件的内容
            cp 拷贝
            mv 移动或者重命名
            rm 删除文件
                -r 递归删除，可删除目录及文件
                -f 强制删除
            find 在文件系统中搜索某文件
            wc [textfile] 统计文本中行数、字数、字符数
            grep [textfile] 在文本文件中查找某个字符串
            rmdir 删除空目录
            pwd 显示当前目录
            ln 创建链接文件
            clear 清除控制台命令信息
            df 显示已经挂载的分区列表
            du 查看文件和目录磁盘使用的空间情况
            which [command] 查看命令路径
            diff [path1|filename1] [path2|filename2] 比较两个文件的差异性
                        
    * 系统信息
    
            arch 显示机器的处理器架构
            uname -m 显示机器的处理器架构
            uname -r 显示正在使用的内核版本
            cat /proc/cpuinfo 显示CPU info信息
            cat /proc/meminfo 显示内存 info信息
            cat /proc/version 显示内核的版本信息
            cat /proc/net/dev 显示网络适配器及统计信息
            cat /proc/mounts 显示已加载的文件系统
            date 显示系统日期
            cal 显示当前日历 
                [year] 显示年份的日历信息
            stat [filename] 显示指定文件的详细信息，比ls更详细
            who 显示在线登陆用户
            whoami 显示当前操作用户
            hostname 显示主机名
            top 动态显示当前耗费资源最多进程信息
            ps 显示瞬间进程状态
            ifconfig 查看网络情况
            ping [url] 测试网络连通性
            netstat 显示网络状态信息
            kill [id] 杀死进程
            
    * 关机
    
            shutdown 关闭系统
                -r 关机重启
                -h 关机不重启
                now 立刻关机
            halt 关机
            init 0 关闭系统
            telinit 0 关闭系统
            shutdown -h hours:minutes 按预定时间关闭系统
            shutdown -c 取消按预定时间关闭系统
            reboot 重启
            logout 注销
            
    * 文件和目录
    
            pwd 显示工作路径
            cd 进入个人的主目录
                [path] 进入某个目录
                .. 返回上一级目录
                ../.. 返回上两级目录
                ~[path] 模糊搜索进入某个路径（局限于用户目录）
            ls 查看目录中的文件
                -F 查看目录中的文件
                -l 显示文件和目录的详细资料
                -a 显示所有文件（包括隐藏文件）
            mkdir [directory]... 创建新目录，可以创建多个目录
                -p [path] 创建一个目录树
            rmdir [directory]... 删除目录，可以删除多个目录
            rm -f [filename] 删除某个文件
                -rf [directory]... 删除目录及该目录下的所有内容，可以删除多个
            mv [param1] [param2] 重命名文件或者将文件从一个目录转移到另一个目录
            cp [param1] [param2] 复制文件或者将目录下的所有文件复制到另一个目录
            ln [filename] [link] 创建一个指向文件或目录的物理链接
                -s [filename] [link] 创建一个指向文件或目录的软链接
            touch [filename]  创建一个空文件
                -t [ms] [filename|directory] 修改一个文件或目录的时间戳
            find [path] [rules]根据规则在制定目录下查找文件
    
    * 磁盘空间
    
            df 显示已经挂载的分区列表
                -h 以可读性更高的方式来显示列表
            du [filename|directory]查看文件和目录磁盘使用的空间情况，可以查看单个文件或目录
                -s 只显示总和大小
                -h 以可读性更高的方式来显示列表

    * 用户和群组
            
            groupadd [name] 创建一个新用户组
            groupdel [name] 删除一个用户组
            groupmod -n [new_name] [old_name] 重命名一个用户组
            useradd [name] 创建一个新用户
            userdel -r [name] 删除一个用户（-r 排除主目录）
            password 修改密码
                [username] 修改某个用户的密码（只允许root用户）    
            

* 安装软件
    
        不能连接互联网的情况（通过传递的软件包）
        1.安装软件：sudo dpkg -i
        2.卸载软件：sudo dpkg -r
        
        连接互联网的情况
        1.安装软件：sudo apt-get install 
        2.卸载软件：sudo apt-get remove
        3.更新软件：sudo apt-get update
        4.软件更新对比：sodu apt-get upgrade
 
* 将.rpm文件转换为.deb文件

        sudo alien：.rpm为RedHat使用的软件格式。在Ubuntu下不能直接使用，所以需要转换一下。
        
* -man手册操作守则  
      
        心得
        1.当一个命令没有规定大小写的时候，那么他的大小写都代表同一个意思。
        2.当一个命令规定区分大小写的时候，那么可以用shift+小写代替大写
        	        
    * enter return键：向后翻一行
    * k：向前翻一行
    * ctrl+d：向文件尾部翻半屏
    * ctrl+u：向文件首部翻半屏
    * G：跳至最后一行
    * \#G：跳转至指定行，#表示行号
    * /keyword：以keyword指定的字符串为关键字，从当前界面所在位置向文件尾部搜索；不区分字符大小写
    * ?keyword：以keyword指定的字符串为关键字，从当前界面所在位置向文件首部搜索；不区分字符大小写
    * n：下一个
    * N：上一个
    * q 退出
    
* vim使用操作详解

        vim 选择文本，删除，复制，粘贴：
        v选择 再按一下v结束  ctrl+v选中矩形
        ggVG 选中全部的文本，其中gg为调到行首，V选中整行，G末尾
        d删除  y复制   p粘贴   yyp复制光标所在行，并粘贴
        
    * 删除字符 将光标移到该字符上按下x
    * 删除一行 删除一行内容使用dd
    * 删除换行符 删除两行之间的换行符用J
    * 撤销 撤销命令用u
    * 重做 反转撤销 crtl+R ，U命令他一次撤销对一行的全部操作，第二次使用该命令则会撤销前一个U的操作。
    * 追加 i光标前插入文本   a当前光标插入文本  o在当前行另起一行    O当前行上面另起一行
    * 使用命令计数 很多命令都可以接受一个数字作为重复执行同一命令的次数
    * 退出 ZZ该命令保持当前文件并推出vim  也可以:wq
    * 放弃编辑 q! 丢弃所有的修改并推出      e! 放弃所有修改并且重新载入该文件的原始内容

* 文件权限管理

        三种基本权限     R读数值为4     W写数值为2    X可执行数值为1（可执行权限表示运行以及进入该目录）
        
        文件分为四段：
        1.第一个字符   表示文件类型    -普通文件  l链接   d目录
        2.第二三四个字符 表示当前所属用户的权限（文件所有者的权限-创建该文件的人）
        3.第五六七个字符 表示当前所属组的权限（与文件所有者同一组的用户）
        4.第八九十个字符 表示其他用户的权限（不与文件所有者同组的其他用户）
        
        改变权限的命令：
        chmod 755 adb：赋予abc权限rwxr-xr-x
        chmod u=rwx,g=rx,o=rx abd：同上
        chmod u-x,g+w：给abc去除用户执行的权限，增加组写的权限
        chmod a+r abc：给所有用户添加读的权限
        
* 压缩解压

        gzip:
        bzip2:
        tar:    打包压缩
            -c  归档文件
            -x  压缩文件
            -z  gzip压缩文件
            -j  bzip2压缩文件
            -v  显示压缩或解压缩过程v(view 视图)
            -f  使用档名
        例：
        tar -cvf /home/abc.tar /home/abc              只打包，不压缩
        tar -zcvf /home/abc.tar.gz /home/abc        打包，并用gzip压缩
        tar -jcvf /home/abc.tar.bz2 /home/abc      打包，并用bzip2压缩
        当然，如果想解压缩，就直接替换上面的命令  tar -cvf  / tar -zcvf  / tar -jcvf 中的“c” 换成“x” 就可以了。