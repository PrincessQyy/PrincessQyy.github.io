---
layout:     post
title:      "【shell-scripts】常用shell小脚本"
date:       2017-08-25 15:01:00
author:     "Yuki"
---

# 移动指定修改时间的文件

### 需求：某些目录下的文件可能过一段时间就需要清理，这个脚本的功能就是将指定年份的文件移动到某个指定目录下。

    #!/bin/bash
    
    #author:
    #	Qiyuyue
    #program:
    #   This script is used to back up files that specify the modification time
    
    if [ ! -d /tmp/etc ];then
    mkdir /tmp/etc
    fi
    
    ls -l /etc | grep 2016 | awk '{print $9}' >> file_dir
    
    for each in `cat file_dir`;do
    cp -r /etc/${each}  /tmp/etc/
    done

# 测试指定IP是否能ping通

### 需求：在服务器批量装系统前，需要测试是否具备装系统条件，即其lo口要能ping通，而其eth0 ip应该是ping不通的，此脚本就是检测ip列表的ip是否能ping通

	#!/bin/bash

	#author:
    #	Qiyuyue
    #program:
    #   This script is used to ping a machine

	for i in `cat ip_list`;do
	        ping  $i -c2 -w2  > result;
	        cat result|grep "64 bytes">&/dev/null  && echo "True" || echo "False"
	
	done

# 统计网络数据包

### 需求：某些场景下，可能会需要统计某个端口的进出数据包数，以便于分析流量等，监听时间可以指定
	
	#!/bin/bash

	#author:
    #	Qiyuyue
    #program:
    #   This script is used to count the number of packages
	#usage:
	#	bash $0 port time

	tcpdump -i eth0 "port $1" &>tmp.txt &
	
	sleep $2
	#kill tcpdump process ,remenber to filter command "grep"
	kill `ps aux | grep "tcpdump" |grep -v "grep" | awk '{print $2}'`

	a=`cat tmp.txt | grep "received" | awk '{print $1}'`
	
	echo "Packages: ${a}"

# 提取文本中的链接信息

### 需求：文本中可能包含有用的链接信息，这里把链接和链接的文字描述提取出来（md文本格式的，所以链接形式是 ![链接描述](链接)）

	#!/bin/bash
	#author:
    #	Qiyuyue
    #program:
    #   This script is used to get links
	
	#创建临时文件，仅做缓冲
	if [ -f tmp ];then
	        rm -f tmp &> /dev/null
	fi
	#将含有链接的行输出到文件
	for line in `cat $1`;do
	        echo "$line" | grep "https://" | sed -n '/[.+]/,$p' >> tmp
	done
	#有的一行里含有多个链接，所以先按逗号把每个包含链接的语句单独成行，然后只显示我们想要的[]()部分，再进行操作
	cat tmp | sed 's/,/\n/g' |grep -o "\[.*\](.*)" |sed -r 's/\[(.*)\]\((https:\/\/.*)\)/\1 \2/g' | grep -v "jpg" | grep -v "png"


# 避免误删


### 需求： Linux中，`rm -f` 是个相当危险的命令，如果在自己的 Linux 服务器上不小心执行了这个操作那必须十分慎重。为了避免误删除一些重要的文件，需要通过一些设置来实现回收站的功能~

脚本如下:

	#!/bin/bash
	#author:
    #	Qiyuyue
    #program:
    #   This script is used to define actions when using "rm -f"
	
	#创建垃圾箱目录
	if [ ! -e "/tmp/trash" ];then
	        mkdir /tmp/trash
	fi

	#对要删除的文件（文件名或者路径）进行操作
	for each in $@;do
		#仅取文件名
        file_name=`echo $each | awk -F/ '{print $NF}'`
        mv $each /tmp/trash/$file_name &> /dev/null
	done

然后在 /home/username/.bashrc 文件中新增一行：其实就是给rm -f起个别名

`alias rm -f='bash /root/my_scripts/avoid_rm.sh'`

然后执行 `source /home/username/.bashrc` 使配置立即生效

如果担心 trash文件越来越大，可以设置定时任务定期清理（例如每天0点清理一下）

`0 0 * * * rm -rf /tmp/trash/*`

# Nginx访问日志分析

	#!/bin/bash
	
	#author:
	    #   Qiyuyue
	#program:
	    #   This script is used to back up files that specify the modification time
	echo "2015年4月10日访问次数最多的五个ip:"
	cat /root/my_scripts/access.log |grep "10/Apr/2015" | awk '{print $1}' |sort | uniq -c | sed -r 's/^[[:space:]]+//g' | sort -nr | awk '{print $2}' | head -n 5
	
	echo "访问次数最多的10个请求内容"
	cat /root/my_scripts/access.log | grep -v " /robots.txt|*.js|*.css|*.png"|sort -k 1 | uniq -cw 14 | head | awk '{print $8}'
	
	echo "2015年4月11日期间访问大于等于 10 次的所有 IP 地址:"
	cat /root/my_scripts/access.log |grep "11/Apr/2015" | awk '{print $1}' |sort | uniq -c | sed -r 's/^[[:space:]]+//g' | sort -nr | awk '{if($1>=10){print $2}}'
	
	echo "日志文件中访问状态为 404 的所有访问请求地址:"
	
	#cat /root/my_scripts/access.log | grep  "\<404\>" | awk '{print $7}' 
	
