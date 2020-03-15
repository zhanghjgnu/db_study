## Chrony设置服务器同步时间

CentOS7.7环境
Chrony有两个核心组件，分别是：chronyd：是守护进程，主要用于调整内核中运行的系统时间和时间服务器同步。它确定计算机增减时间的比率，并对此进行调整补偿。chronyc：提供一个用户界面，用于监控性能并进行多样化的配置。它可以在chronyd实例控制的计算机上工作，也可以在一台不同的远程计算机上工作。
要使程序运行应该打开防火墙的udp123端口.
### 1 安装
```
 # yum install chrony -y
``` 
### 2 启动和开机启动
```
 # systemctl enable chronyd
 # systemctl restart chronyd
 # systemctl status chronyd
``` 
### 3.设置
```
 # nano /etc/chrony.conf
``` 
 一般只需要增加server为本地时间服务器的IP即可
 增加一行
 server 172.20.20.10 iburst
 其余的设置基本都可以采用默认值
### 4.检查一下时区 timedatectl
### 5.重启服务
```
  # systemctl restart chronyd
```  
### 6.检查工作状态 
查看时间同步源：
chronyc sources -v

查看时间同步源状态：
chronyc sourcestats -v

校准时间服务器：
chronyc tracking
