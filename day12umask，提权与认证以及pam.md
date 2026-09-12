umask
用于设置创建文件/目录时的权限，可以防止数据泄露,ssh私钥被恶意利用等，设置umask时只对创建的文件/目录生效，对已经存在的文件或目录无用
在创建文件时默认的umsk码是666，目录是777
umask 查看当前umask码

在设置umask码时必须连用三个数字不同的数字意味着去除不同的权限
比如:
umask 002 对其他人去除可读权限
注意 设置umask码只会对当前会话生效，关机或者重启后失效

永久修改umask码
对当前用户生效
vim ~/.bashrc
在最后一行输入
umask 对应的数字

 对系统的所有用户生效：
 vim /etc/bash.bashrc
 在最后一行输入
 umask 对应的数字
 重启后即可生效

提权与认证
sudo可以精确的设置用户可以使用哪些命令其核心配置文件为/etc/sudoers，文件权限必须为0440，仅仅root可读，严禁编辑，要设置对用户的命令使用权限时，一般在/etc/sudoers下面创建一个文件

使用visudo 来配置文件
visudo 打开/etc/sudoers文件
visudo -c 检查sudoers语法正确性
visudo -f /etc/sudoers.d/文件名 编辑配置文件

sudo的配置语法
用户名 主机名 = （用户：用户组） 标签 命令列表
比如：
root ALL = (ALL:ALL) ALL
允许root用户在所有主机上使用所有命令
%sudo ALL = (ALL:ALL) ALL
允许suodo组的成员在所有主机上使用所有命令
alice ALL = (root) /sbin/reboot
允许alice在所有主机上只能以root身份能使用reboot命令
bob ALL = (ALL:ALL) NOPASSWD:ALL
bob用户可以在所有主机上免密使用所有命令
lzy ALL = (ALL:ALL) NOPASSWD:/usr/bin/ls,/usr/bin/chmod
lzy用户可以在所有主机上免密执行ls，chmod命令
1 ALL = (ALL:ALL) ALL,!/usr/bin/rm -rf *
1用户可以在所有主机上执行所有命令但不能删除任何文件

在配置sudo时采用一下原则:
只授予用户完成工作所需的最小权限
优先使用命令级授权而非全部命令（all）
命令必须使用绝对路径（使用which可以查看命令存放位置）
优先使用%用户组来设置权限而非单一用户
使用！来禁止危险命令
避免使用NOPASSWD

在配置sudo时，可以使用别名来进行批量管理：
1.创建用户别名
User_Alias 别名 = 要添加的用户 使用，分隔
2.创建命令别名
Cmnd_Alias 别名 = 命令的绝对路径 ,分隔
3.创建主机别名
Host_Alias 别名 = 主机名
4.进行配置
用户别名 主机别名 = （ALL） 命令别名

在配置sudo完成后其默认的权限为0440当完成配置后，使用sudo visudo -c -f /etc/sudoers/文件名 来检查语法是否错误，在设置别名时，一定要大写
可在/var/log/auth.log中查看执行过的操作以及被禁止的操作

PAM 可插拔认证模块
PAM可以使用户通过一定的模块配置各种安全策略而不是修改程序代码本身

PAM三层架构
应用层 用于接收请求并将请求转发给pam库层
PAM库层 收到请求后安排底层模块层按照一定顺序开始执行任务
底层模块曾 按照pam库层安排的顺序开始执行任务，并将最后的结果转发给应用层

PAM四大管理组
auth 检测身份
account 检查账户状态
paasword 处理密码更改
session 管理会话生命周期

控制标志
pam根据控制标志决定模块执行结果的整体判定
required 必须成功，失败时不立即返回，继续执行后续模块，当执行完所有模块后返回失败
requisite 必须成功，失败时立即返回
sufficient 成功立即返回，前提是前面没有required/requisite失时忽略继续执行模块
optional 结果不影响整体认证结果，失败时忽略

pam配置文件一般存放在/etc/pam.d下面，其中common-文件可以共享配置

配置文件格式
类型 控制标志 模块路径 参数
例如：
auth requisite pam_dent.so 在身份验证阶段，只要执行到这个模块立即终止后续的验证流程

常用的pam模块
pam_unis.so 使用/etc/passwd /etc/shadow 进行验证
pam_ldap.so/pam_ssss.so 支持ldap或者sss，适合企业环境
pam_pwqulity.so 检查密码复杂度，强制密码强度策略
pam_limits.so 设置用户资源限制
pam_tally2.so/pam_faillock.so 账户锁定机制，防止暴力破解
pam_geogle_authenticator.so 支持geogle authenticator 双因素认证
pam_mdhomedir.so 首次登陆时自动创建/home目录
pam_umask.so 为用户设置umask值
举一个例子：
auth required pam_tally2.so deny=5 unlock_time = 300 other = fail
在身份认证阶段时，限制用户在五次内输错密码将会锁定账户五分钟，期间账户如果发生任何异常系统也会拒接登陆

pam使用灵活，无需重新编辑程序，统一策略管理，支持多种服务，配置错时将会导致用户无法登陆系统，修改前必须备份，修改后建议保持当前会话开启，出错时便于恢复，生产环境修改前应在测试环境充分验证

























 
