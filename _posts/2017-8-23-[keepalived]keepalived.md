---
layout:     post
title:      "【keepalived】keepalived原理及配置"
date:       2017-08-23 15:34:00
author:     "Yuki"
---

# keepalived原理介绍

#### keepalived简介

Keepalived的功能有点像是两个人互相看着一个工作，如果一个人离开岗位另外一个人就会接替，这个keepalived就是他们之间保持这样“替换机制”的工具。keepalived是一个类似于layer3, 4 & 5交换机制的软件，也就是我们平时说的第3层、第4层和第5层交换。Keepalived的作用是检测web服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的web服务器从系统中剔除，当web服务器工作正常后Keepalived自动将web服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的web服务器。

* Keepalived服务主要有两大用途：heartbeat（高可用）&failover（健康检测）
   
Keepalived服务主要依靠vrrp来完成这些工作的，以下我就来介绍下VRRP协议是怎样的工作的，那么基本上keepalived的工作原理就是如此。


**VRRP协议（VRRP Virtual Router Redundancy Protocol，虚拟路由冗余协议）**

VRRP协议过程简述：VRRP 将局域网的一组路由器（包括一个Master 即活动路由器和若干个Backup 即备份路由器）组织成一个虚拟路由器，称之为一个备份组。这个虚拟的路由器拥有自己的IP 地址10.100.10.1（这个IP 地址可以和备份组内的某个路由器的接口地址相同，相同的则称为ip拥有者），备份组内的路由器也有自己的IP 地址（如Master的IP 地址为10.100.10.2，Backup 的IP 地址为10.100.10.3）。局域网内的主机仅仅知道这个虚拟路由器的IP 地址10.100.10.1，而并不知道具体的Master 路由器的IP 地址10.100.10.2 以及Backup 路由器的IP 地址10.100.10.3。[1]它们将自己的缺省路由下一跳地址设置为该虚拟路由器的IP 地址10.100.10.1。于是，网络内的主机就通过这个虚拟的路由器来与其它网络进行通信。如果备份组内的Master 路由器坏掉，Backup 路由器将会通过选举策略选出一个新的Master 路由器，继续向网络内的主机提供路由服务。从而实现网络内的主机不间断地与外部网络进行通信。

**VRRP的工作过程**

VRRP的工作过程如下：

1. 路由器开启VRRP功能后，会根据优先级确定自己在备份组中的角色。优先级高的路由器成为主用路由器，优先级低的成为备用路由器。主用路由器定期发送VRRP通告报文，通知备份组内的其他路由器自己工作正常；备用路由器则启动定时器等待通告报文的到来。

2. VRRP在不同的主用抢占方式下，主用角色的替换方式不同：

a. 在抢占方式下，当主用路由器收到VRRP通告报文后，会将自己的优先级与通告报文中的优先级进行比较。如果大于通告报文中的优先级，则成为主用路由器；否则将保持备用状态。

b. 在非抢占方式下，只要主用路由器没有出现故障，备份组中的路由器始终保持主用或备用状态，备份组中的路由器即使随后被配置了更高的优先级也不会成为主用路由器。

3. 如果备用路由器的定时器超时后仍未收到主用路由器发送来的VRRP通告报文，则认为主用路由器已经无法正常工作，此时备用路由器会认为自己是主用路由器，并对外发送VRRP通告报文。备份组内的路由器根据优先级选举出主用路由器，承担报文的转发功能。

VRRP在提高可靠性的同时，简化了主机的配置。在具有多播或广播能力的局域网中，借助VRRP能在某台路由器出现故障时仍然提供高可靠的缺省链路，有效避免单一链路发生故障后网络中断的问题，而无需修改动态路由协议、路由发现协议等配置信息。

一个VRRP路由器有唯一的标识：VRID，范围为0—255｡该路由器对外表现为唯一的虚拟MAC地址，地址的格式为00-00-5E-00-01-[VRID]｡主控路由器负责对ARP请求用该MAC地址做应答｡这样,无论如何切换，保证给终端设备的是唯一一致的IP和MAC地址，减少了切换对终端设备的影响｡

VRRP控制报文只有一种：VRRP通告(advertisement)｡它使用IP多播数据包进行封装，组地址为224.0.0.18，发布范围只限于同一局域网内｡这保证了VRID在不同网络中可以重复使用｡为了减少网络带宽消耗只有主控路由器才可以周期性的发送VRRP通告报文｡备份路由器在连续三个通告间隔内收不到VRRP或收到优先级为0的通告后启动新的一轮VRRP选举｡

在VRRP路由器组中，按优先级选举主控路由器，VRRP协议中优先级范围是0—255｡若VRRP路由器的IP地址和虚拟路由器的接口IP地址相同，则该VRRP路由器被称为该IP地址的所有者；IP地址所有者自动具有最高优先级：255｡优先级0一般用在IP地址所有者主动放弃主控者角色时使用｡可配置的优先级范围为1—254｡优先级的配置原则可以依据链路的速度和成本､路由器性能和可靠性以及其它管理策略设定｡主控路由器的选举中，高优先级的虚拟路由器获胜，因此，如果在VRRP组中有IP地址所有者，则它总是作为主控路由的角色出现｡对于相同优先级的候选路由器，按照IP地址大小顺序选举｡VRRP还提供了优先级抢占策略，如果配置了该策略，高优先级的备份路由器便会剥夺当前低优先级的主控路由器而成为新的主控路由器｡

为了保证VRRP协议的安全性，提供了两种安全认证措施：明文认证和IP头认证｡明文认证方式要求：在加入一个VRRP路由器组时，必须同时提供相同的VRID和明文密码｡适合于避免在局域网内的配置错误，但不能防止通过网络监听方式获得密码｡IP头认证的方式提供了更高的安全性，能够防止报文重放和修改等攻击｡[3] 

# keepalived 配置

### 配置keepalived为实现haproxy高可用的配置文件/etc/keepalived/keepalived.conf：

	! Configuration File for keepalived  
	  
	global_defs {  
	   notification_email {  
	         PrincessQyy@gmail.com
	   }  
	   notification_email_from kanotify@magedu.com 
	   smtp_connect_timeout 3  
	   smtp_server 127.0.0.1  
	   router_id LVS_DEVEL  
	}  
	
	vrrp_script chk_haproxy {  
	    script "killall -0 haproxy"  
	    interval 1  
	    weight 2  
	}  
	
	vrrp_script chk_mantaince_down {
	   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
	   interval 1
	   weight 2
	}
	
	vrrp_instance VI_1 {  
	    interface eth0  
	    state MASTER  # BACKUP for slave routers
	    priority 101  # 100 for BACKUP
	    virtual_router_id 51 
	    garp_master_delay 1 
	  
	    authentication {  
	        auth_type PASS  
	        auth_pass password  
	    }  
	    track_interface {  
	       eth0    
	    }  
	    virtual_ipaddress {  
	        172.16.100.1/16 dev eth0 label eth0:0 
	    }  
	    track_script {  
	        chk_haproxy  
	        chk_mantaince_down
	    }  
	  
	 
	    notify_master "/etc/keepalived/notify.sh master"  
	    notify_backup "/etc/keepalived/notify.sh backup"  
	    notify_fault "/etc/keepalived/notify.sh fault"  
	}

注意：
1、上面的state为当前节点的起始状态，通常在master/slave的双节点模型中，其一个默认为MASTER，而别一个默认为BACKUP。
2、priority为当关节点在当前虚拟路由器中的优先级，master的优先级应该大于slave的；


**下面是notify.sh脚本**
	#!/bin/bash
	# description: A notify script
	# 
	
	vip=172.16.100.1
	contact='root@localhost'
	
	Notify() {
	    mailsubject="`hostname` to be $1: $vip floating"
	    mailbody="`date '+%F %H:%M:%S'`: vrrp transition, `hostname` changed to be $1"
	    echo $mailbody | mail -s "$mailsubject" $contact
	}
	
	case "$1" in
	    master)
	        notify master
	        /etc/rc.d/init.d/haproxy start
	        exit 0
	    ;;
	    backup)
	        notify backup
	        /etc/rc.d/init.d/haproxy restart
	        exit 0
	    ;;
	    fault)
	        notify fault
	        exit 0
	    ;;
	    *)
	        echo 'Usage: `basename $0` {master|backup|fault}'
	        exit 1
	    ;;
	esac
	
### 配置keepalived为实现haproxy高可用的双主模型配置文件示例：

说明：其基本实现思想为创建两个虚拟路由器，并以两个节点互为主从。

	! Configuration File for keepalived  
	  
	global_defs {  
	   notification_email {  
	         PrincessQyy@gmail.com
	   }  
	   notification_email_from kanotify@magedu.com 
	   smtp_connect_timeout 3  
	   smtp_server 127.0.0.1  
	   router_id LVS_DEVEL  
	}  
	
	vrrp_script chk_haproxy {  
	    script "killall -0 haproxy"  
	    interval 1  
	    weight 2  
	}  
	
	vrrp_script chk_mantaince_down {
	   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
	   interval 1
	   weight 2
	}
	
	vrrp_instance VI_1 {  
	    interface eth0  
	    state MASTER  # BACKUP for slave routers
	    priority 101  # 100 for BACKUP
	    virtual_router_id 51 
	    garp_master_delay 1 
	  
	    authentication {  
	        auth_type PASS  
	        auth_pass password  
	    }  
	    track_interface {  
	       eth0    
	    }  
	    virtual_ipaddress {  
	        172.16.100.1/16 dev eth0 label eth0:0 
	    }  
	    track_script {  
	        chk_haproxy  
	        chk_mantaince_down
	    }  
	  
	 
	    notify_master "/etc/keepalived/notify.sh master"  
	    notify_backup "/etc/keepalived/notify.sh backup"  
	    notify_fault "/etc/keepalived/notify.sh fault"  
	} 
	
	vrrp_instance VI_2 {  
	    interface eth0  
	    state BACKUP  # BACKUP for slave routers
	    priority 100  # 100 for BACKUP
	    virtual_router_id 52
	    garp_master_delay 1 
	  
	    authentication {  
	        auth_type PASS  
	        auth_pass password  
	    }  
	    track_interface {  
	       eth0    
	    }  
	    virtual_ipaddress {  
	        172.16.100.2/16 dev eth0 label eth0:1
	    }  
	    track_script {  
	        chk_haproxy  
	        chk_mantaince_down
	    }    
	}


说明：
1、对于VI_1和VI_2来说，两个节点要互为主从关系；