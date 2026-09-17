用户管理：
useradd + 用户名 添加一个用户
password 用户名 更改用户密码
userdel 用户名 删除用户
userdel -r 删除用户以及目录，一般不建议删完

用户组管理：
groupadd 组名 新增用户组
groupdel 组名 删除用户组
useradd -g 组名 用户名 创建用户并且直接将用户加入到该组

指定运行级别：
0 关机
1 单用户
2 多用户无网络服务
3 多用户有网络服务
4 系统未使用保留给用户
5 图形界面
6 系统重启
使用init 来指定运行级别

找回root密码大致流程（ubuntu）：
在启动时按e进入gbk界面，随后选在进入恢复模式，找到root获取root shell,随后挂在文件系统mount -o remount.rw /,接着输入passwd root.然后在输入密码，reboot重启即可

