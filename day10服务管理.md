
# Linux 服务运行级别与端口管理速查手册

> 覆盖运行级别（0–6）、systemctl 开机自启与服务启停、UFW 端口开放与查看。

## 目录

- [服务的运行级别](#服务的运行级别)
- [开机自启管理](#开机自启管理)
- [服务启停与重载](#服务启停与重载)
- [端口管理（UFW）](#端口管理ufw)
- [速记小结](#速记小结)
- [勘误说明](#勘误说明)

---

## 服务的运行级别

Linux 共有 7 个运行级别（Runlevel），用数字 `0–6` 表示，决定系统启动后进入的工作状态。

| 级别 | 名称 | 说明 |
| :--- | :--- | :--- |
| `0` | 停机 | 关机状态。**绝不能设为默认**，否则系统无法开机 |
| `1` | 单用户 | root 权限，用于系统维护，**禁止远程登录** |
| `2` | 多用户（无网络） | 多用户文本模式，不带网络（RHEL/CentOS）；Debian/Ubuntu 默认此级别带网络 |
| `3` | 完全多用户 | 登录后进入控制台命令行模式（服务器常用） |
| `4` | 保留 | 系统未使用，可自定义 |
| `5` | 图形界面 | X11 控制台，登录进入图形 GUI 模式 |
| `6` | 重启 | 正常关闭并重启。同 `0` 一样**不能设为默认**，否则无法正常开机 |

> [!WARNING]
> 运行级别 `0`（停机）与 `6`（重启）一旦被设置为系统默认运行级别，将导致开机即关机或反复重启，无法进入系统。修改默认级别前务必确认目标值为 `3` 或 `5`。

> [!TIP]
> 日常最常使用的是 **`3`（命令行多用户）** 和 **`5`（图形界面）**。服务器一般默认运行级别 `3`。

---

## 开机自启管理

| 命令 | 说明 |
| :--- | :--- |
| `systemctl list-unit-files --type=service` | 查看所有服务的开机自启状态 |
| `systemctl list-unit-files --type=service --state=enabled` | 仅查看已设为开机自启的服务 |
| `systemctl enable 服务名` | 设置服务开机自启动 |
| `systemctl disable 服务名` | 取消服务开机自启动 |

> [!IMPORTANT]
> `enable` / `disable` **只控制开机是否自启，不会立即启动或停止当前运行的服务**。若需"设置自启的同时立即生效"，请加 `--now`，例如 `systemctl enable --now 服务名`。

```bash
# 查看 sshd 是否开机自启
systemctl list-unit-files --type=service | grep sshd

# 设置 sshd 开机自启并立即启动
systemctl enable --now sshd
```

---

## 服务启停与重载

正确语序为 `systemctl 选项 服务名`。

| 选项 | 说明 |
| :--- | :--- |
| `start` | 启动服务 |
| `stop` | 停止服务 |
| `restart` | 重启服务（完全重启进程） |
| `reload` | 重载配置（不中断服务，仅重新读取配置文件） |
| `status` | 查看服务当前运行状态 |

```bash
# 重启 nginx
systemctl restart nginx

# 修改配置后仅重载，不中断服务
systemctl reload nginx

# 查看运行状态
systemctl status nginx
```

---

## 端口管理（UFW）

UFW（Uncomplicated Firewall）用于管理防火墙端口的开放与关闭。

> [!NOTE]
> 使用 UFW 前需先启用防火墙：`sudo ufw enable`。首次启用可能提示会中断 SSH 连接，确认已放行 22 端口后再继续。

| 命令 | 说明 |
| :--- | :--- |
| `ufw allow 端口号/协议` | 开放指定端口（如 `80/tcp`） |
| `ufw delete allow 端口号/协议` | 关闭（删除）已开放的端口规则 |
| `ufw status` | 查看当前端口开放情况 |

```bash
# 开放 80 端口（TCP）
sudo ufw allow 80/tcp

# 关闭 80 端口
sudo ufw delete allow 80/tcp

# 查看所有端口是否开放
sudo ufw status
```

---

## 速记小结

| 场景 | 命令 / 值 | 关键提示 |
| :--- | :--- | :--- |
| 命令行服务器 | 运行级别 `3` | 最常用 |
| 图形界面 | 运行级别 `5` | 桌面环境 |
| 危险级别 | `0` / `6` | 切勿设为默认 |
| 查看自启状态 | `systemctl list-unit-files --type=service` | 配 `--state=enabled` 过滤 |
| 设置开机自启 | `systemctl enable 服务名` | 不立即启动，需 `--now` |
| 重启 / 重载服务 | `systemctl restart / reload 服务名` | reload 不中断 |
| 开放端口 | `ufw allow 端口/协议` | 需先 `ufw enable` |
| 查看端口 | `ufw status` | 显示放行规则 |

---

