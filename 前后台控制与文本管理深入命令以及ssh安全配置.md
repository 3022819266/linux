前后台进程控制
& 通常与nohup相互配合使用
ctrl+z 将正在执行的程序暂停并进入后台挂起状态
jobs 查看正在后台或者挂起的程序
bg %n(n为用jobs查看的进程号)将挂起的任务放在后台运行
fg %n 将挂起的后台或者任务放在前台运行

grep,awk,sed
grep 文本搜索命令
-r 查找整个目录
-i 忽略大小写
-v 排除指定内容
-n 显示匹配的行号
当有|符号时，grep接收的时文本数据流因此不用加-r
举一个例子：
grep -rn  "error" /var/log 查找/var/log下含有error的行并且显示行号

awk 列处理与数据提取
awk '条件 {print #要打印的行}' 文件名 可以提取文件列的数据

sed 擅长在不打开文件的情况下进行非交互式文本替换
sed 选项 ‘地址，命令’ 目标文件
选项：
-i.bak 先备份原文件在修改
-n 静默模式，通常与-p配合
-e 执行多个编辑命令

命令：
s 替换
d 删除
p 打印
a 在行后追加文本
i 在行前追加文本
c 直接替换整行内容

例如：
sed -i 's/old/new/g' 1.txt 在1.txt文件上直接修改，将文档中old全局替换为new
sed -n '10,20p' 1.txt 查看文档的第十到二十行
sed '/^#d' 1.txt 删除1.txt中以#开头的行

ssh安全配置
ssh 密钥登陆是一种相较于传统密码登陆的一种安全性更高的方式

1.生成密钥对
ssh-keygen -t ed25519 -C "邮箱名" 此处的邮箱名仅作为标识作用，让你知道这是你的密钥对
2.部署公钥到服务器
ssh-copy-id -i ~/.ssh/id_ed25519.pub 服务器名@服务器ip
3.进行权限分配
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
4.测试密钥登陆
ssh -i ~/.ssh/id_ed25519 服务器名@服务器ip

ssh加固策略
1.禁止root远程登陆
PermitRootLogin no 禁止登陆后，攻击者必须才对普通用户名在才对密钥，即使普通用户被入侵后，攻击者还要提升权限才能获取root权限
2.禁用密码认证
PasswordAuthentication no 彻底关闭密码登录，只有持有合法的私钥用户才能链接，在执行此操作前，必须确保至少有一个密钥登陆可用的用户，否则将永远失去远程登陆权限
3.修改默认端口
port 端口号 建议选择10000-65535之间的端口号，避免使用系统端口和常见服务端口，此操作可以挡住99%的自动化扫描脚本
在设置端口号，下次登陆就要用 ssh -p 端口号 服务器名@服务器ip 进行登陆

部署fail2ban防止暴力破解
fail2ban是一个入侵防御框架，通过/var/log/auth.log识别频繁失败的登陆尝试，并自动调用防火墙规则封禁恶意ip，他是ssh安全的最后一道防线
在配配置时，不要直接修改jail.conf,=而是创建fail.local覆盖配置
vim /etc/fail2ban/jail.local
例如：
[sshd]
enabled = true 启用此jail
port = 端口号 监控的ssh端口
maxretry = 3  在findtime里允许的最大失败次数
bantime = 3600 封禁时间，若设置为-1则永久封禁
findtime = 600 统计失败次数的额时间窗口
ignoreip = 127.0.0.1/8 ：：1 白名单配哦之，允许放行的ip地址或地址段
工作原理：
当fail2ban启动后，持续监控/var/log/auth.log文件，当某一ip在十分钟内进行失败登录三次后，则触发封禁动作，随后防火墙插入drop规则，丢弃该ip所有流量，封禁到期后，移除该规则
注意，白名单配置一定要写

fail2ban-client status sshd 查看封禁状态
fail2ban-client set sshd unbanip ip地址 手动解封被封禁的ip地址

在生产环境中的操作规范：
永远不要在没有退路的情况下修改ssh配置，若修改了端口，先放行端口，在改配置，修改配置后，不要关闭当前终端，测试时，要新开一个终端来测试，确认执行权限认证成功后，再关闭，在配置fail2ban时，白名单配置优先级最高
在完成配置后，使用sshd -t 来检查语法错误，随后ststemctl restart ssh重启ssh服务查看服务状态

ssh的配置流程大致如下：
生成 Ed25519 密钥对（或 RSA-4096），设置 passphrase
使用 ssh-copy-id 部署公钥，验证权限 700/600
新开终端测试密钥登录成功，确认无需密码
编辑 sshd_config：
PermitRootLogin no
PasswordAuthentication no
Port <非标准端口>
MaxAuthTries 3
LoginGraceTime 30
执行 sshd -t 确认无语法错误
云安全组/防火墙放行新端口
重启 sshd，保留旧终端，新开终端测试
确认新终端登录后，再关闭旧终端
安装并配置 fail2ban：
创建 jail.local，启用 [sshd]
设置 ignoreip 白名单（公司IP/堡垒机）
启动并验证：fail2ban-client status sshd
记录 VNC 登录方式，作为最后应急手段

核心原则
密钥优先，密码禁用：这是安全基线，不可妥协。
操作留退路：任何 SSH 配置变更，必须保留已验证的活跃会话。
纵深防御：密钥 + 禁Root + 改端口 + fail2ban，四层防护缺一不可。
白名单先行：自动化安全工具必须配置白名单，避免自伤。
定期审计：每月检查 authorized_keys 是否有未知公钥，检查 fail2ban 封禁记录。

