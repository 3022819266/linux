
# Linux 权限掩码、提权认证与 PAM 速查手册（umask / sudo / PAM）

> 涵盖默认权限掩码 `umask` 的设置与持久化、`sudo` 精细提权与 `/etc/sudoers` 配置、以及可插拔认证模块 `PAM` 的架构、管理组、控制标志与常用模块。

## 目录

- [umask 默认权限掩码](#umask-默认权限掩码)
- [永久修改 umask](#永久修改-umask)
- [提权与认证 sudo](#提权与认证-sudo)
- [sudo 配置语法](#sudo-配置语法)
- [sudo 配置原则](#sudo-配置原则)
- [sudo 别名批量管理](#sudo-别名批量管理)
- [PAM 可插拔认证模块](#pam-可插拔认证模块)
- [PAM 管理组与控制标志](#pam-管理组与控制标志)
- [常用 PAM 模块](#常用-pam-模块)
- [速记小结](#速记小结)


---

## umask 默认权限掩码

`umask` 用于设置**创建文件 / 目录时的默认权限**，可防止数据泄露、SSH 私钥被恶意利用等风险。

> [!IMPORTANT]
> `umask` 只对**新创建**的文件 / 目录生效，对已存在的文件或目录无效；并且临时设置只对**当前会话**生效，关机或重启后失效。

- 创建**文件**时的基准权限为 `666`，创建**目录**时的基准权限为 `777`。
- 实际权限 = 基准权限 减去（按位去除）umask 值。
- 设置 umask 码时必须连用**三个数字**，每一位分别对应所有者 / 所在组 / 其他人要**去除**的权限。

```bash
umask            # 查看当前 umask 码
umask 022        # 组与其他人去除写权限
umask 002        # 其他人去除写权限（2 = w）
```

| 基准 | 数值含义 |
| :--- | :--- |
| 文件默认基准 | `666`（rw-rw-rw-）|
| 目录默认基准 | `777`（rwxrwxrwx）|
| umask 每位 | `4`=去 r，`2`=去 w，`1`=去 x |

---

## 永久修改 umask

| 生效范围 | 配置文件 | 操作 |
| :--- | :--- | :--- |
| 仅当前用户 | `~/.bashrc` | 在文件末尾追加 `umask 数字`，重新登录 / `source` 后生效 |
| 系统所有用户 | `/etc/bash.bashrc` | 在文件末尾追加 `umask 数字`，重启或重新登录会话后生效 |

```bash
# 对当前用户生效
vim ~/.bashrc
# 末行追加（示例）
umask 022

# 对所有用户生效
vim /etc/bash.bashrc
# 末行追加
umask 022
```

> [!TIP]
> 修改后让当前会话立即生效，可执行 `source ~/.bashrc`，无需重新登录。

---

## 提权与认证 sudo

`sudo` 可以**精确设置**某个用户能使用哪些命令。

- 核心配置文件：`/etc/sudoers`
- 该文件权限**必须为 `0440`**（仅 root 可读），**严禁直接用普通编辑器修改**。
- 推荐做法：在 `/etc/sudoers.d/` 目录下为单个用户 / 场景**新建独立配置文件**，避免直接改动主文件。

```bash
visudo                          # 安全打开 /etc/sudoers（带锁与语法检查）
visudo -c                        # 检查 sudoers 语法正确性
visudo -f /etc/sudoers.d/文件名   # 编辑 /etc/sudoers.d 下的独立配置文件
```

---

## sudo 配置语法

```text
用户名 主机名 = (运行身份用户:运行身份组) 标签 命令列表
```

| 配置示例 | 含义 |
| :--- | :--- |
| `root ALL = (ALL:ALL) ALL` | 允许 root 在所有主机上以任意身份使用所有命令 |
| `%sudo ALL = (ALL:ALL) ALL` | 允许 `sudo` 组成员在所有主机上使用所有命令 |
| `alice ALL = (root) /sbin/reboot` | 允许 alice 在所有主机上仅以 root 身份执行 `reboot` |
| `bob ALL = (ALL:ALL) NOPASSWD:ALL` | bob 在所有主机上**免密**使用所有命令 |
| `lzy ALL = (ALL:ALL) NOPASSWD:/usr/bin/ls,/usr/bin/chmod` | lzy 免密执行 `ls`、`chmod` 两个命令 |
| `dev ALL = (ALL:ALL) ALL,!/usr/bin/rm` | dev 可执行所有命令，但**禁止** `rm` |

> [!WARNING]
> 原文「`1 ALL = ...`」以数字 `1` 作为用户名不合规范，此处示例改用 `dev` 演示 `!` 禁止危险命令的写法。命令**必须使用绝对路径**。

---

## sudo 配置原则

- 只授予用户完成工作所需的**最小权限**。
- 优先使用**命令级授权**，而非直接给 `ALL`。
- 命令一律使用**绝对路径**（用 `which 命令名` 查看存放位置）。
- 优先使用 `%用户组` 设置权限，而非单一用户。
- 使用 `!` 禁止危险命令。
- 尽量避免使用 `NOPASSWD`（免密会降低安全性）。

---

## sudo 别名批量管理

配置 sudo 时可用别名进行批量管理，**别名必须全大写**。

```text
# 1. 创建用户别名（多个用逗号分隔）
User_Alias  别名 = 用户1,用户2

# 2. 创建命令别名（命令用绝对路径，逗号分隔）
Cmnd_Alias  别名 = /usr/bin/ls,/usr/bin/chmod

# 3. 创建主机别名
Host_Alias  别名 = 主机名

# 4. 在规则中引用别名
用户别名  主机别名 = (ALL) 命令别名
```

> [!IMPORTANT]
> sudoers 类文件配置完成后默认权限应为 `0440`。改完后用 `sudo visudo -c -f /etc/sudoers.d/文件名` 检查语法是否错误。执行过的操作与被拒绝的操作可在 `/var/log/auth.log` 中查看。

---

## PAM 可插拔认证模块

PAM（Pluggable Authentication Modules，可插拔认证模块）让用户通过**配置模块**实现各种安全策略，而**无需修改程序源代码本身**。

### 三层架构

| 层 | 职责 |
| :--- | :--- |
| 应用层 | 接收用户请求，并将请求转发给 PAM 库层 |
| PAM 库层 | 收到请求后，安排底层模块按一定顺序执行任务 |
| 底层模块层 | 按库层安排的顺序执行任务，并把最终结果返回给应用层 |

> [!WARNING]
> PAM 配置错误会导致用户**无法登录系统**。修改前**必须备份**；修改后建议**保持当前会话不关闭**，以便出错时用现有会话恢复；生产环境改动应在测试环境充分验证。

---

## PAM 管理组与控制标志

### 四大管理组

| 类型 | 作用 |
| :--- | :--- |
| `auth` | 检测身份（认证）|
| `account` | 检查账户状态（有效期、是否允许登录等）|
| `password` | 处理密码更改与策略 |
| `session` | 管理会话生命周期（登录 / 登出时的环境准备）|

### 控制标志

PAM 根据控制标志决定模块执行结果对整体判定的影响。

| 控制标志 | 行为 |
| :--- | :--- |
| `required` | 必须成功；失败时**不立即返回**，继续执行后续模块，全部执行完后整体返回失败 |
| `requisite` | 必须成功；失败时**立即返回**失败，终止后续模块 |
| `sufficient` | 成功则**立即返回成功**（前提：此前没有 `required`/`requisite` 失败，否则忽略继续执行）|
| `optional` | 结果**不影响**整体认证；失败时忽略，继续执行 |

### 配置文件格式

配置文件一般存放在 `/etc/pam.d/` 下，其中 `common-*` 文件可被各服务**共享引用**。

```text
类型  控制标志  模块路径  [模块参数]
```

```text
# 示例：身份验证阶段执行到该模块即立即终止后续验证
auth  requisite  pam_deny.so
```

> [!NOTE]
> 原文示例「`auth requisite pam_dent.so`」中的 `pam_dent` 应为 `pam_deny`（拒绝模块）。`requisite` 的语义正是"失败即终止"，与示例描述一致。

---

## 常用 PAM 模块

| 模块 | 作用 |
| :--- | :--- |
| `pam_unix.so` | 使用 `/etc/passwd`、`/etc/shadow` 进行本地验证 |
| `pam_ldap.so` / `pam_sss.so` | 支持 LDAP / SSS（SSSD），适合企业集中认证环境 |
| `pam_pwquality.so` | 检查密码复杂度，强制密码强度策略 |
| `pam_limits.so` | 设置用户资源限制（读取 `/etc/security/limits.conf`）|
| `pam_tally2.so` / `pam_faillock.so` | 账户锁定机制，防止暴力破解 |
| `pam_google_authenticator.so` | 支持 Google Authenticator 双因素认证（2FA）|
| `pam_mkhomedir.so` | 首次登录时自动创建 `/home` 家目录 |
| `pam_umask.so` | 为用户设置 umask 值 |

### 锁定示例

```text
auth required pam_tally2.so deny=5 unlock_time=300 even_deny_root
```

含义：在身份认证阶段，用户在连续输错密码 **5 次**后将被**锁定账户 300 秒（5 分钟）**，锁定期内即使凭据正确也拒绝登录。（不同发行版对参数名支持略有差异，CentOS 8+ / 新版 Ubuntu 已逐步用 `pam_faillock` 替代 `pam_tally2`。）

---

## 速记小结

| 场景 | 命令 / 配置 | 关键点 |
| :--- | :--- | :--- |
| 查看 / 设置掩码 | `umask 022` | 仅新对象、当前会话生效 |
| 永久生效（用户）| `~/.bashrc` 末行加 `umask` | `source` 立即生效 |
| 永久生效（全系统）| `/etc/bash.bashrc` 末行加 `umask` | 重启 / 重登生效 |
| 编辑 sudo 规则 | `visudo -f /etc/sudoers.d/文件` | 严禁直接 vim 主文件 |
| 检查 sudo 语法 | `visudo -c` | 保存前后都应检查 |
| 禁止危险命令 | `!命令绝对路径` | 配合最小权限原则 |
| PAM 配置目录 | `/etc/pam.d/` | 改前必备份、保留会话 |
| 失败锁定 | `pam_tally2` / `pam_faillock` | `deny` + `unlock_time` |

---
