## putty登录linux输入密码很慢的解决办法
使用putty连接CentOS7.6服务器，可能会等待10秒才有提示输入密码。登录很慢，登录上去后速度正常，这种情况主要有两种可能的原因：

### 1.DNS 反向解析问题
OpenSSH在用户登录的时候会验证 IP，它根据用户的 IP 使用反向 DNS 找到主机名，再使用 DNS 找到 IP 地址，最后匹配一下登录的 IP 是否合法。如果客户机的 IP 没有域名，或者 DNS 服务器很慢或不通，那么登录就会很花时间。
解决办法：

vi /etc/ssh/sshd_config
输入 / ,查找 UseDNS,赋值为 no

### 2. 关闭 ssh 的 gssapi 认证
解决办法：
vi /etc/ssh/sshd_config
输入 / ,查找 GSSAPIAuthentication 赋值为 no ;

重启 sshd生效
service sshd restart

