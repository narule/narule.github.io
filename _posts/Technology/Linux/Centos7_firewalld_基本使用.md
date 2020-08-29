Centos7 的防火墙 firewalld比较常见

　　简单介绍使用

　　详细介绍链接推荐：  https://blog.csdn.net/buster_zr/article/details/80604933

　　　　　　　　　　　　http://blog.51cto.com/13503302/2095633

　　　　　　　　　　　　https://www.jianshu.com/p/c06890b7739e

## firewalld 使用：

　　iptables firewalld 都是防火墙程序，并且firewalld内部也是通过iptables实现防火墙配置，但是两个程序不能同时运行 

####  　安装

　　如果iptables在运行，先关闭

　　systemctl stop iptables　　　　#关闭iptables

　　systemctl disable iptables　　#禁止iptables 开机启动

　　systemctl status firewalld　　#查看防火墙firewalld状态

　　如果运行，会显示Active: active (running)　　

　　firewall-cmd --state　　　　　　#查看防火墙firewalld状态的另一种方式

　　no runing 表示没有运行

　　yum -y install firewalld　　　　#安装 firewalld

　　systemctl enable firewalld　　#设置开启启动

　　systemctl start firewalld　　　#启动

　　firewall-cmd --list-all 　　#查看默认区域已有规则 

　　firewall-cmd --list-all-zones #查看所有区域规则　

 

#### 　　添加端口

　　firewall-cmd --zone=public --add-port=端口/类型 --permanent # 加上--permanent 表示永久生效

　　如果修改了ssh端口，ssh端口修改为5511 那么可以开启5511端口

　　firewall-cmd --zone=public --add-port=5511/tcp --permanent #开启5511端口 tcp类型 

　　firewall-cmd --reload #重新加载规则，使设置马上生效

　　　

#### 　　配置文件

　　两个位置   /usr/lib/firewalld/　　/etc/firewalld/

　　默认的配置在第一个位置，如果添加自己的规则，建议在第二个文件位置出修改

 

#### 　　添加服务　　

　　firewall-cmd --zone=public --add-service=服务名

　　服务很多已经有了配置文件 就是在/usr/lib/firewalld/目录下面 如 http服务 就是 /usr/lib/firewalld/services/http.xml

　　所以如若要开启80端口 可以有两种方式

　　方式一： firewall-cmd --zone=public --add-service=http --permanent　　#添加http 服务 配置文件中默认是80端口

　　方式二： firewall-cmd --zone=public --add-port=80/tcp --permanent　　#添加 80/tcp 端口　

　　firewall-cmd --reload #重载配置

　　添加 443 端口https 服务

　　firewall-cmd --zone=public --add-service=https --permanent

　　firewall-cmd --reload　　　　　　　　

　　