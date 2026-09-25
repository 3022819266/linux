
# Linux 前后台进程控制、文本三剑客与 SSH 安全配置速查

> 涵盖作业控制（`&` / `Ctrl+Z` / `jobs` / `bg` / `fg`）、文本处理三剑客（`grep` / `awk` / `sed`）以及 SSH 密钥登录、加固策略与 fail2ban 防暴力破解的完整实操流程。

## 目录

- [前后台进程控制（作业控制）](#前后台进程控制作业控制)
- [文本处理三剑客](#文本处理三剑客)
- [SSH 密钥登录配置流程](#ssh-密钥登录配置流程)
- [SSH 加固策略](#ssh-加固策略)
- [部署 fail2ban 防止暴力破解](#部署-fail2ban-防止暴力破解)
- [生产环境操作规范与完整流程](#生产环境操作规范与完整流程)
- [核心原则](#核心原则)
- [速记小结](#速记小结)


---

## 前后台进程控制（作业控制）

Shell 可在前台与后台之间切换正在运行的任务（job）。`&` 通常与 `nohup` 配合，让进程脱离终端在后台持续运行。

| 命令 / 按键 | 作用 |
| :--- | :--- |
| `命令 &` | 启动时即放入**后台**运行（常配合 `nohup` 防挂断） |
| `Ctrl + Z` | 将前台程序**暂停（挂起 / Stopped）**并送回后台 |
| `jobs` | 查看当前终端的后台 / 挂起任务及其编号 |
| `bg %n` | 让编号为 `n` 的挂起任务在**后台继续运行** |
| `fg %n` | 让编号为 `n` 的后台任务回到**前台**运行 |

> [!NOTE]
> `%n` 中的 `n` 是 `jobs` 输出的**作业号**（如 `[1]`），不是进程 PID。不带 `%n` 时，`bg` / `fg` 默认作用于当前活动作业。

```bash
# 典型用法：把挂起的任务放到后台继续跑
sleep 100          # 前台运行中
# 按下 Ctrl + Z    ->  [1]+  Stopped  sleep 100
jobs               # 查看所有作业
bg %1              # 让 1 号作业在后台继续
fg %1              # 再把它拉回前台
```

---

## 文本处理三剑客

### grep 文本搜索

按行匹配并输出符合条件的内容。

| 选项 | 说明 |
| :--- | :--- |
| `-r` | 递归查找整个目录（recursive） |
| `-i` | 忽略大小写（ignore-case） |
| `-v` | 反向匹配，排除指定内容（invert） |
| `-n` | 显示匹配所在行的行号（line-number） |

```bash
# 在 /var/log 下递归查找含 error 的行并显示行号
grep -rn "error" /var/log
```

> [!TIP]
> 当命令通过 `|` 管道传入时，`grep` 接收的是**标准输入文本流**，此时 `-r`（目录递归）无意义，无需加 `-r`。

### awk 列处理与数据提取

按字段（列）处理文本，默认以空白为分隔符。

```bash
awk '{ print $1, $3 }' 文件名       # 提取第 1、3 列
awk -F: '{ print $1 }' /etc/passwd  # 指定冒号为分隔符取第 1 列
```

> [!WARNING]
> `print $N` 打印的是第 **N 列（字段）**，不是第 N 行；`$0` 表示整行。行级过滤应交给条件部分（如 `awk '$3 > 100 { print }'`）。

### sed 流编辑器

无需打开文件即可进行非交互式文本替换 / 删除 / 打印。

**命令格式：** `sed [选项] '地址,命令' 目标文件`

| 选项 | 说明 |
| :--- | :--- |
| `-i.bak` | 就地修改文件，并先备份为 `原文件名.bak` |
| `-n` | 静默模式，仅输出被 `p` 显式打印的行（常与 `-p` 配合） |
| `-e` | 一次执行多个编辑命令 |

| 命令 | 作用 |
| :--- | :--- |
| `s` | 替换（`s/old/new/g`，`g` 表示全局） |
| `d` | 删除匹配行 |
| `p` | 打印匹配行 |
| `a` | 在匹配行**之后**追加文本 |
| `i` | 在匹配行**之前**追加文本 |
| `c` | 将匹配行**整行**替换为新内容 |

```bash
sed -i 's/old/new/g' 1.txt     # 就地全局替换 old -> new
sed -n '10,20p' 1.txt          # 查看第 10 到 20 行
sed '/^#/d' 1.txt              # 删除以 # 开头的行
```

---

## SSH 密钥登录配置流程

SSH 密钥登录相比传统密码登录安全性更高。以下以 Ed25519 算法为例。

```bash
# 1. 生成密钥对（-C 后的邮箱仅作标识，便于识别密钥归属）
ssh-keygen -t ed25519 -C "you@example.com"

# 2. 部署公钥到服务器
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server_ip

# 3. 权限分配（关键，权限过宽会导致密钥被拒用）
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 4. 测试密钥登录
ssh -i ~/.ssh/id_ed25519 user@server_ip
```

> [!IMPORTANT]
> 生成密钥时建议设置 **passphrase**（私钥口令），即使私钥文件泄露也无法被直接使用。

---

## SSH 加固策略

编辑服务端配置文件 `/etc/ssh/sshd_config`：

| 配置项 | 值 | 作用 |
| :--- | :--- | :--- |
| `PermitRootLogin` | `no` | 禁止 root 远程登录。攻击者须先猜中普通用户名再破解密钥，即使普通用户失陷仍需提权才能拿到 root |
| `PasswordAuthentication` | `no` | 彻底关闭密码登录，仅持有合法私钥者可连接 |
| `Port` | `非标准端口` | 避开常见端口，可挡住约 99% 的自动化扫描脚本 |

```text
PermitRootLogin no
PasswordAuthentication no
Port 22222
```

> [!WARNING]
> 关闭密码认证前，**务必确认至少有一个用户的密钥登录已测试可用**，否则将永久失去远程登录权限。修改端口后，下次登录需显式指定 `ssh -p 22222 user@server_ip`，并记得在防火墙 / 云安全组放行新端口。

---

## 部署 fail2ban 防止暴力破解

`fail2ban` 是入侵防御框架，通过监控 `/var/log/auth.log` 识别频繁失败的登录尝试，自动调用防火墙规则封禁恶意 IP，是 SSH 安全的最后一道防线。

> [!TIP]
> 不要直接修改 `jail.conf`，应创建 `jail.local` 覆盖配置，避免升级时被官方配置覆盖。

```bash
vim /etc/fail2ban/jail.local
```

```ini
[sshd]
enabled   = true
port      = 22222      # 监控的 SSH 端口
maxretry  = 3          # findtime 窗口内允许的最大失败次数
bantime   = 3600       # 封禁时长（秒），-1 表示永久封禁
findtime  = 600        # 统计失败次数的时间窗口（秒）
ignoreip  = 127.0.0.1/8 ::1   # 白名单，允许放行的 IP 或地址段
```

**工作原理：** 启动后持续监控 `/var/log/auth.log`，当某 IP 在 `findtime`（如 600 秒）内失败达到 `maxretry`（如 3 次）即触发封禁，防火墙插入 DROP 规则丢弃该 IP 全部流量；`bantime` 到期后自动移除规则。

```bash
# 查看封禁状态
fail2ban-client status sshd
# 手动解封指定 IP
fail2ban-client set sshd unbanip 1.2.3.4
```

> [!IMPORTANT]
> **白名单（`ignoreip`）一定要配置**，优先级最高，防止把自己的 IP 误封导致失联。

---

## 生产环境操作规范与完整流程

> [!WARNING]
> 永远不要在“没有退路”的情况下修改 SSH 配置。改端口要**先放行端口、再改配置**；改完配置**不要关闭当前终端**，另开一个新终端测试密钥认证成功、再关闭旧终端。配置 fail2ban 时白名单优先级最高。

```text
1. 生成 Ed25519 密钥对（或 RSA-4096），设置 passphrase
2. 使用 ssh-copy-id 部署公钥，验证权限 700 / 600
3. 新开终端测试密钥登录成功，确认无需密码
4. 编辑 sshd_config：
     PermitRootLogin no
     PasswordAuthentication no
     Port <非标准端口>
     MaxAuthTries 3
     LoginGraceTime 30
5. 执行 sshd -t 确认无语法错误
6. 云安全组 / 防火墙放行新端口
7. 重启 sshd，保留旧终端，新开终端测试
8. 确认新终端登录后，再关闭旧终端
9. 安装并配置 fail2ban：
     创建 jail.local，启用 [sshd]
     设置 ignoreip 白名单（公司 IP / 堡垒机）
     启动并验证：fail2ban-client status sshd
10. 记录 VNC 登录方式，作为最后应急手段
```

```bash
# 改完配置先校验语法，再重启服务
sshd -t
systemctl restart ssh      # Debian/Ubuntu 服务名为 ssh
# systemctl restart sshd   # RHEL/CentOS 服务名为 sshd
```

---

## 核心原则

1. **密钥优先，密码禁用**：这是安全基线，不可妥协。
2. **操作留退路**：任何 SSH 配置变更，必须保留一个已验证的活跃会话。
3. **纵深防御**：密钥 + 禁 Root + 改端口 + fail2ban，四层防护缺一不可。
4. **白名单先行**：自动化安全工具必须配置白名单，避免自伤。
5. **定期审计**：每月检查 `authorized_keys` 是否有未知公钥，检查 fail2ban 封禁记录。

---

## 速记小结

| 场景 | 命令 | 关键点 |
| :--- | :--- | :--- |
| 挂起当前任务 | `Ctrl + Z` | 进程转为 Stopped |
| 后台继续 | `bg %1` | 配合 `jobs` 取作业号 |
| 回前台 | `fg %1` | 前台恢复交互 |
| 递归搜日志 | `grep -rn "error" /var/log` | 管道输入时免 `-r` |
| 提取列 | `awk '{ print $1 }'` | `$N` 是第 N 列 |
| 就地替换 | `sed -i 's/old/new/g'` | `-i.bak` 先备份 |
| 密钥登录 | `ssh-copy-id` | 权限 700 / 600 |
| 改配置校验 | `sshd -t` | 通过后再 restart |
| 看封禁状态 | `fail2ban-client status sshd` | 白名单必配 |

---

