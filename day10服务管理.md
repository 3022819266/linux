服务的运行级别：
  0 停机状态，默认运行状态为不能设置为0否则无法开机
  1 单用户工作状态，root用户权限，用于系统维护，禁止远程登陆
  2 多用户状态，无网络
  3 完全的多用户状态，登陆后进入控制台命令模式
  4 系统未使用，保留
  5 xll控制台登陆后进入图新gui模式
  6 系统正常关闭并重启，同级别一一致不能设置未默认模式否则无法开机
  一般常用的运行级别未3 5

systemctl list-unit-files --type=service 查看所有服务的开机自启动状态
sysytemctl list-unit-files --type=service -state=enabled 查看开机自启的服务
systemctl enable 服务名 设置服务开机启动
systemctl disable 服务名 设置服务关闭开机自启动

systemctl 选项 服务名 开，关，重启，重载服务
可选选项：start stop restart reload
如果要设置某个服务自启动或者关闭永久生效：
systemctl enable|disable

打开或关闭指定端口
ufw allow 端口号/协议
关闭：
ufw delete allow 端口号/协议
查看端口号是否开放
ufw status

  
