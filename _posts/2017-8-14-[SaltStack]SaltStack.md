---
layout:     post
title:      "【SaltStack】SaltStack基本操作"
date:       2017-08-14 18:40:00
author:     "Yuki"
---

# 基础介绍

### 简介
SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，一般可以理解为简化版的puppet和加强版的func。SaltStack基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。
通过部署SaltStack环境，我们可以在成千上万台服务器上做到批量执行命令，根据不同业务特性进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。
 
* 特性

(1)、部署简单、方便；

(2)、支持大部分UNIX/Linux及Windows环境；

(3)、主从集中化管理；

(4)、配置简单、功能强大、扩展性强；

(5)、主控端（master）和被控端（minion）基于证书认证，安全可靠；

(6)、支持API及自定义模块，可通过Python轻松扩展。

# Salt安装

	 yum install python-crypto ##安装
	 yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm  ##安装一个salt的源
	 yum clean all
	 yum install salt-master  ##安装master（仅安装在master端）
	 yum install salt-minion  ##安装minion（仅安装在minion端）

# salt配置

[详细配置参考](https://wenku.baidu.com/view/9be3fe195022aaea988f0f1d.html)

在安装好salt后，在minion端，编辑配置文件，

	`vim /etc/salt/minion`

在配置文件中，`/id` 搜索到id这一栏，冒号后面空一个可以给你的minion起个名儿，也是为了标识，起不起名儿无所谓，`\master`搜索到master一栏，冒号后面空一格配置上master端的ip,然后就可以使用啦。

# salt基本操作

* salt 的使用语法为 `salt 目标 模块.方法`

目标是你要操作的minion ，模块和方法就是具体操作啦。

### 目标的筛选

* 基于Grains System ：静态数据，当Minion启动的时候收集的MInion本地的相关信息。（包含操作系统版本、内核版本、CPU、内存、硬盘、设备型号等）
`salt -G 'os:Ubuntu' test.ping`

* 使用扩展正则表达式
`salt -E 'minion[0-9]' test.ping`

* 使用列表指定
`salt -L 'minion1,minion2' test.ping`

* 在一个命令中组合多个目标类型
`salt -C 'G@os:Ubuntu and minion* or S@192.168.50.*' test.ping`

### 方法使用

* `salt-key -L` ：安装和配置好salt minion后，可以在master端执行命令 `salt-key -L` 列出所有key，key前面的标识就是状态啦，很容易理解。
* `salt-key -A`:通过 `salt-key -a id` 或者 `salt-key -A` 来添加指定的/全部的未接受的key.
* `salt '*' test.ping`：在执行其他操作前，通常需要执行这条命令来测试minion的存活性，返回TRUE表示存活
* `salt-run manage.up`： 查看活着的minion
* `salt-run manage.down` ：查看查看down的minion
* `salt-run manage.status`：查看所有的状态
* `salt-run manage.down removekeys=True`：查看并删除down的key

* `salt '*' disk.usage`：查看磁盘使用情况
* `salt '*' pkg.install cowsay`：批量安装软件
* `salt '*' network.interfaces`：列出网络接口
* `salt '*' cron.list_tab user` ：列出定时任务，最后一项填用户

### SLS

SLS（代表SaLt State文件）是Salt State系统的核心。SLS描述了系统的目标状态，由格式简单的数据构成。这经常被称作配置管理 首先，在master上面定义salt的主目录，默认是在/srv/salt/下面，打开 /etc/salt/master，找到以下行取消注释：

	file_roots:
	   base:
	     - /srv/salt

top.sls 是配置管理的入口文件，一切都是从这里开始，在master 主机上，默认存放在/srv/salt/目录. 
top.sls 默认从 base 标签开始解析执行,下一级是操作的目标，可以通过正则，grain模块,或分组名,来进行匹配,再下一级是要执行的state文件，不包换扩展名。

#### 创建 /srv/salt/top.sls

* 通过正则进行匹配的示例，
		
	base:
		 '*':
		   - webserver

* 通过分组名进行匹配的示例，必须要有 - match: nodegroup
		
	base:
		 group1:
		   - match: nodegroup    
		   - webserver
		    
* 通过grain模块匹配的示例，必须要有- match: grain
		
	base:
		'os:Fedora':
		   - match: grain
		   - webserver

* 准备好top.sls文件后，编写一个state文件
/srv/salt/webserver.sls
	
	apache:                 # 标签定义
		pkg:                  # state declaration
			- installed         # function declaration
		    
第一行被称为（ID declaration） 标签定义，在这里被定义为安装包的名。注意：在不同发行版软件包命名不同,比如 fedora 中叫httpd的包 Debian/Ubuntu中叫apache2

第二行被称为（state declaration）状态定义， 在这里定义使用（pkg state module）

第三行被称为（function declaration）函数定义， 在这里定义使用（pkg state module）调用 installed 函数

最后可以在终端中执行命令来查看结果：

`salt '*' state.highstate`

或附件 test=True参数 测试执行

`salt '*' state.highstate -v test=True `

主控端对目标主机（targeted minions）发出指令运行state.highstatem模块，目标主机首先会对top.sls下载，解析，然后按照top.sls内匹配规则内的定义的模块将被下载,解析,执行，然后结果反馈给 master.

#### SLS文件命名空间

注意在以上的例子中,SLS文件 webserver.sls 被简称为webserver. SLS文件命名空间有如下几条基本的规则：

* SLS文件的扩展名 .sls 被省略。 (例如. webserver.sls 变成 webserver)
* 子目录可以更好的组织,每个子目录都由一个点来表示.(例如 webserver/dev.sls 可以简称为 webserver.dev）
* 如果子目录创建一个init.sls的文件，引用的时候仅指定该目录即可. (例如 webserver/init.sls 可以简称为 webserver）
* 如果一个目录下同时存在webserver.sls 和 webserver/init.sls，那么 webserver/init.sls 将被忽略，SLS文件引用的webserver将只引用webserver.sls

#### state多文件示例
	    
编写完top.sls后，编写state.sls文件；

<img src="../../../../../img/blogs/SaltStack/01.png">

在当前目录下面（salt的主目录）创建files/nginx/nginx.conf、files/nginx/fastcgi.conf文件，里面肯定是你自己项配置的nginx配置文件的内容啦；使用salt做自动化，一般nginx都是挺熟悉的，这里不做详细解释了

#### state的逻辑关系列表：

* include： 包含某个文件 比如我新建的一个my_webserver.sls文件内，就可以继承nginx和PHP相关模块配置，而不必重新编写

<img src="../../../../../img/blogs/SaltStack/02.png">
	  
* match: 配模某个模块，比如 之前定义top.sls时候的 

`match: grain match: `

* nodegroup require： 依赖某个state，在运行此state前，先运行依赖的state，依赖可以有多个 比如文中的nginx模块内，相关的配置必须要先依赖nginx的安装

<img src="../../../../../img/blogs/SaltStack/03.png">

* watch： 在某个state变化时运行此模块，文中的配置，相关文件变化后，立即执行相应操作

<img src="../../../../../img/blogs/SaltStack/04.png">

* order： 优先级比require和watch低，有order指定的state比没有order指定的优先级高，假如一个state模块内安装多个服务，或者其他依赖关系，可以使用
 
<img src="../../../../../img/blogs/SaltStack/05.png">
	
想让某个state最后一个运行，可以用last

#### 进阶主题：模板
使用模板来精简SLS，使SLS可以使用python的 循环，分支，判断 等逻辑

<img src="../../../../../img/blogs/SaltStack/06.png">

通过加载jinja模板引擎，可以模板配置文件按照预订条件来生成最终的配置文件

<img src="../../../../../img/blogs/SaltStack/07.png">


### 实例

#### 批量部署 Nginx

**环境准备**

建议关闭防火墙 `systemctl stop firewalld.service`

关闭selinux，直接在/etc/selinux/config 文件中将selinux选项改为disabled

**方法一：**

这个方法是不写top文件的，先看下目录结构吧

<img src="../../../../../img/blogs/SaltStack/08.png">

再看看 sls文件

<img src="../../../../../img/blogs/SaltStack/09.png">

一般 name 选项后面写minion端的路径（绝对路径）

source 选项后面写 master端的路径，这里是相对路径，相对于/srv/salt/

其他字段的含义就看链接里的博客吧，写得很清楚：[文件管理](http://blog.csdn.net/hnhuangyiyang/article/details/50412738)

在master端直接执行 `salt '*' state.sls nginx` 命令，会自动去Nginx目录下找到init.sls文件完成安装，。成功图和方法二一样，就不截图啦。

file和 yum.repos.d目录下就是配置和yum源，就不截图了。。。

**方法二：**

同理，先看下目录结构


<img src="../../../../../img/blogs/SaltStack/10.png">

其他文件都一样，就是多了个top.sls文件，那么，看看这个文件写了些啥吧

<img src="../../../../../img/blogs/SaltStack/11.png">

这里需要注意格式，缩进分别是两个空格、四个空格，对格式要求很严格，缩进不符合规则，文件就不起作用。内容其实就一句...执行命令用 `salt '*'  state.highstate` ，它会自动去找nginx目录下的init.sls执行,可以看到执行成功，图太长，我就截一点点吧，反正就是一片绿色，如下图：

<img src="../../../../../img/blogs/SaltStack/12.png">

# salt 常用模块总结

### cmd模块

* name：要执行的命令，记住该命令将会在salt-minion的路径和权限下执行

* onlyif：用于检查的命令，仅当``onlyif``选项指向的命令返回true时才执行name定义的命令

* unless：用于检查的命令，仅当``unless``选项指向的命令返回false时才执行name指向的命令

* cwd：执行命令时的当前工作目录，默认是/root

* user：以指定用户身份运行命令

* group：以指定用户组身份运行命令

* shell：用于执行命令的shell，默认shell grain

* run：运行name后的命令    

举例：

<img src="../../../../../img/blogs/SaltStack/13.png">

### cron模块（设置定时任务）

* minute：分
* hour：时
* daymonth：日
* month：月
* dayweek：周
* user：用户名
* present：创建计划任务
* name：计划任务内容

举例：


<img src="../../../../../img/blogs/SaltStack/14.png">

### file模块

	#校验所有被控主机/etc/fstab文件的md5值是否为xxxxxxxxxxxxx,一致则返回True值
	salt '*' file.check_hash /etc/fstab md5=xxxxxxxxxxxxxxxxxxxxx
	
	#校验所有被控主机文件的加密信息，支持md5、sha1、sha224、shs256、sha384、sha512加密算法
	salt '*' file.get_sum /etc/passwd md5
	
	#修改所有被控主机/etc/passwd文件的属组、用户权限、等价于chown root:root /etc/passwd
	salt '*' file.chown /etc/passwd root root
	
	#复制所有被控主机/path/to/src文件到本地的/path/to/dst文件
	salt '*' file.copy /path/to/src /path/to/dst
	
	#检查所有被控主机/etc目录是否存在，存在则返回True,检查文件是否存在使用file.file_exists方法
	salt '*' file.directory_exists /etc
	
	#获取所有被控主机/etc/passwd的stats信息
	salt '*' file.stats /etc/passwd
	
	#获取所有被控主机/etc/passwd的权限mode，如755，644
	salt '*' file.get_mode /etc/passwd
	
	#修改所有被控主机/etc/passwd的权限mode为0644
	salt '*' file.set_mode /etc/passwd 0644
	
	#在所有被控主机创建/opt/test目录
	salt '*' file.mkdir /opt/test
	
	#将所有被控主机/etc/httpd/httpd.conf文件的LogLevel参数的warn值修改为info
	salt '*' file.sed /etc/httpd/httpd.conf 'LogLevel warn' 'LogLevel info'
	
	#给所有被控主机的/tmp/test/test.conf文件追加内容‘maxclient 100’
	salt '*' file.append /tmp/test/test.conf 'maxclient 100'
	
	#删除所有被控主机的/tmp/foo文件
	salt '*' file.remove /tmp/foo

### pkg模块

<img src="../../../../../img/blogs/SaltStack/15.png">

### service模块

<img src="../../../../../img/blogs/SaltStack/16.png">

### user模块：

<img src="../../../../../img/blogs/SaltStack/17.png">

举例：公司来新的员工，需要在所有的服务器上添加一个普通账号：
 

<img src="../../../../../img/blogs/SaltStack/18.png">

<img src="../../../../../img/blogs/SaltStack/19.png">

<img src="../../../../../img/blogs/SaltStack/20.png">

# salt-ssh

Salt 在版本 0.17.0 当中，引入了新的传输系统，它支持通过 SSH 通道来实现 Salt 的通信。通过这种方式，我们可以直接通过 SSH 通道在远程主机上执行使用 SaltStack，而不需要在远程主机上运行 Salt Minion ，同时又能支持 SaltStack 的大部分功能，而且 Salt Master 也不需要运行了。这样，也就实现了免客户端方式的部署和实施。


但是由于无客户端本身的局限性， Salt SSH 并不能完全取代标准的 Salt 通信方式，只是简单地提供了一个基于 SSH 通道的可选方式，这种方式不需要 ZeroMQ 和远程 Agent 的支持；整体的工作流程和基于客户端架构大致相同。但必须意识到，通过 Salt SSH 的执行速度会远远低于 ZeroMQ 支持的标准的 Salt 通信方式。

通过ssh连接被管理端，被管理端不用安装minion，管理端也不用安装master，salt-ssh是一个独立的包。一般没有客户端没有安装minion时候才考虑先用salt-ssh批量安装minion。

### salt-ssh安装

在master端执行 `yum -y install salt-ssh`  安装salt-ssh

### salt rosters

Roster 系统为可插拔设计，可以非常方便地加入到已有的系统中，用于 Salt SSH 获取需要连接的服务器信息。默认情况下 Roster 文件本地路径为：/etc/salt/roster。

Roster 系统编译了一个内部数据结构，称为 Targets。Targets 是一个目标系统和关于如何连接到系统属性的列表。对于一个在 Salt 中的 Roster 模块来说，唯一要求是返回 Targets 数据结构：

	<SaltID>： 	# 目标 ID
	  host： 	# 远程主机的 IP 地址或者主机名
	  user： 	# 可以登录的用户
	  passwd： 	# 可以登录用户的密码
	# 可选参数
	  port： 	# SSH 端口
	  sudo： 	# 是否运行 sudo，设置 True 或者 False
	  priv： 	# SSH 私钥的路径，默认是 salt-ssh.rsa
	  timeout： # 连接 SSH 时的超时时间
	  thin_dir：# 目标系统 Salt 的存储路径，默认是 /tmp/salt-<hash>

注意：这里定义的user一定要是远程主机上已有的user

### 初始化

第一次运行 Salt SSH 会提示进行 salt-ssh key 的部署，需要在 Rosters 中配置用户的密码，即可进行 Key 的部署，初始化代码如下：

` salt-ssh '*' test.ping -i` 一定要加上 -i 参数禁用StrictHostKeyCheck来放松接受新的和未知的主机密钥，然后根据提示操作就行了，当部署过 Key 之后，后期再执行 Salt SSH 就会通过密钥认证，不需要用户再输入密码了。

### 基本使用

salt-ssh 命令的语法和 Salt 命令类似，下面看几个典型的使用场景。

* Salt SSH Target

相对于 C/S 模式，salt-ssh 在 Target 中还有一定的局限性，目前支持如下匹配模式：

-E pcre 正则匹配
-L list 列表匹配
-G grain grains 匹配
-grain-pcre grains 加正则匹配
-N nodegroup 组匹配
-R range 范围匹配

* 命令执行

默认情况下 salt-ssh 可以直接在远程系统上运行 Salt 执行模块。通过 salt-ssh 也可以直接执行原始 shell 命令：

[root@linux-node1 ~]# salt-ssh ‘*’ -r ‘w’

该命令基本等同于 salt ‘’ cmd.run ‘w’ 
执行其他模块时也类似，如 salt-ssh ‘’ disk.usage

* 状态管理

salt-ssh 命令可以直接使用 Salt 状态管理系统。使用时将 salt 命令直接替换为 salt-ssh 即可：

salt-ssh ‘*’ state.sls init.dns

上面的命令基本等同于 salt ‘*’ state.sls init.dns