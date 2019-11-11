# 搭建Hadoop的准备工作

## 安装Linux系统

* 要稍后选择要安装的系统

* 一般划定三个分区

  * /boot 200M：系统挂载地区
  * /swap：相当于RAM
  * /：剩余地区

## Linux-start-config

### 配置网络地址

* 找到配置文件`[root@node01 ~]# cd /etc/sysconfig/network-scripts/`

* 修改文件`[root@node01 network-scripts]# vim ifcfg-eth0`

* 注释HWADDR，将BOOTPROTO设置为static即固定IP地址

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Hadoop/%E5%9B%BA%E5%AE%9Aip%E5%9C%B0%E5%9D%80.PNG )

* 启用service network restart重启网络

* ping一下baidu验证网络是否配置成功

* 使用ifconfig查看网络配置情况

* tplink使用192.168.1.1当网关

### 关闭防火墙

* `[root@node01 ~]# service iptables status`查看防火墙状态

* `[root@node01 ~]# service iptables stop`关闭防火墙

* 再输入命令`chkconfig iptables off`

* 再切换到selinux目录下

  ```
  [root@node01 etc]# cd selinux/
  [root@node01 selinux]# pwd
  /etc/selinux
  [root@node01 selinux]# ll
  ```

* `[root@node01 selinux]# vim config `

* 将selinux更改为disabled

* 删除一个文件`[root@node01 selinux]# rm -f /etc/udev/rules.d/70-persistent-net.rules`，之所以这样做是为了以后克隆虚拟机的MAC地址不要相同，不然会导致虚拟机不可用

* 立即poweroff， poweroff之后再进行快照，克隆四台虚拟机

### Linux-clone-config

* 为每个克隆的虚拟机设置一个独立的ip地址

* 需要为每个主机设置一个独立的名字`[root@node01 ~]# vim /etc/sysconfig/network`

  ```
  NETWORKING=yes
  HOSTNAME=node01
  ```

* 为每个主机设置映射`[root@node01 ~]# vim /etc/hosts`，相当于windows中的host表

  ```
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  192.168.68.31 node01
  192.168.68.32 node02
  192.168.68.33 node03
  192.168.68.34 node04
  ```

* 配置完成之后重启，并拍摄快照
* 可通过互相ping ip来判断有无配置成功

  




