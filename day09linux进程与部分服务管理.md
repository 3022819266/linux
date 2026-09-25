
# Linux 进程与服务管理速查手册

> 覆盖进程查看（`ps`）、进程状态解读、终止进程（`kill` / `killall`）、进程树（`pstree`）与服务管理（`systemctl`）。

## 目录

- [查看进程（ps aux）](#查看进程ps-aux)
- [ps aux 各列含义](#ps-aux-各列含义)
- [STAT 进程状态详解](#stat-进程状态详解)
- [全格式查看进程（ps -ef）](#全格式查看进程ps--ef)
- [终止进程](#终止进程)
- [进程树（pstree）](#进程树pstree)
- [服务管理（systemctl）](#服务管理systemctl)
- [速记小结](#速记小结)
- [勘误说明](#勘误说明)

---

## 查看进程（ps aux）

`ps` 用于查看当前系统正在运行的进程。

```bash
# 查看所有进程的详细信息（BSD 风格，aux 不加前置 -）
ps aux
```

> [!TIP]
> `ps aux` 是 BSD 风格写法（`a` 所有终端、`u` 含用户、`x` 无终端进程），通常不加前置 `-`；
> 而 `ps -ef` 是 System V 风格写法，需要带 `-`。

---

## ps aux 各列含义

执行 `ps aux` 后会显示如下列：

| 列名 | 含义 |
| :--- | :--- |
| `USER` | 进程所属用户名 |
| `PID` | 进程号 |
| `%CPU` | 进程占用的 CPU 百分比 |
| `%MEM` | 进程占用物理内存的百分比 |
| `VSZ` | 进程占用虚拟内存的大小（KB）|
| `RSS` | 进程占用的物理内存大小（KB）|
| `TTY` | 终端名称 |
| `STAT` | 进程状态（详见下一节）|
| `START` | 进程启动时间 |
| `TIME` | CPU 时间，进程累计使用 CPU 的总时长 |
| `COMMAND` | 启动进程所用的命令与参数，过长会被截断显示 |

---

## STAT 进程状态详解

`STAT` 列的字符含义：

| 字符 | 含义 |
| :--- | :--- |
| `S` | 睡眠（可中断休眠）|
| `s` | 该进程是会话的先导进程 |
| `N` | 低优先级进程 |
| `R` | 正在运行 |
| `D` | 短期等待（不可中断睡眠，通常在等 I/O）|
| `Z` | 僵死进程 |
| `T` | 被跟踪或被停止的进程 |
| `<` | 高优先级 |
| `L` | 有页面锁定在内存中 |
| `+` | 处于前台进程组 |

---

## 全格式查看进程（ps -ef）

`ps -ef` 以全格式（System V 风格）显示当前所有进程，包含父进程关系，常用来看进程树状依赖。

```bash
ps -ef
```

| 列名 | 含义 |
| :--- | :--- |
| `UID` | 用户 ID（用户名）|
| `PID` | 进程 ID |
| `PPID` | 父进程 ID |
| `C` | CPU 用于计算优先级的估算因子 |
| `STIME` | 进程启动时间 |
| `TTY` | 完整的终端名称 |
| `TIME` | 累计 CPU 时间 |
| `CMD` | 启动进程所用的命令和参数 |

> [!NOTE]
> `C` 是 CPU 调度估算因子，并不直接等于"优先级"；要直观看优先级应使用 `ps aux` 配合排序，或查看 `NI`（nice）列。

---

## 终止进程

```bash
# 按进程号终止（默认发送 SIGTERM，可被进程捕获做清理）
kill 进程号

# 按进程名称终止，会终止同名进程及其所有子进程
killall 进程名称

# 加 -9 强制终止（发送 SIGKILL，进程无法捕获）
kill -9 进程号
killall -9 进程名称
```

> [!WARNING]
> 优先使用默认的 `kill`（SIGTERM），让进程有机会保存数据、清理资源；只有当进程无响应时再用 `-9` 强制杀死，强制终止可能造成数据丢失。

---

## 进程树（pstree）

`pstree` 以树状结构直观展示进程之间的父子关系。

```bash
# 显示进程树
pstree

# 显示每个进程的 PID
pstree -p

# 显示进程的所属用户
pstree -u
```

---

## 服务管理（systemctl）

`systemctl` 用于管理系统服务，注意语序是 **`systemctl 选项 服务名`**。

```bash
# 启动 / 停止 / 重启 / 重载某个服务
systemctl start   服务名
systemctl stop    服务名
systemctl restart 服务名
systemctl reload  服务名

# 查看服务运行状态
systemctl status  服务名

# 设置开机自启 / 取消自启
systemctl enable  服务名
systemctl disable 服务名

# 查看全部系统服务
systemctl list-unit-files --type=service --all
```

---

## 速记小结

| 场景 | 命令 | 关键提示 |
| :--- | :--- | :--- |
| 详细查看进程 | `ps aux` | BSD 风格，含 CPU/内存占用 |
| 全格式含父进程 | `ps -ef` | System V 风格，看 PPID |
| 按 PID 终止 | `kill 进程号` | 默认 SIGTERM，`-9` 强制 |
| 按名称终止 | `killall 进程名` | 含其所有子进程 |
| 看进程树 | `pstree -pu` | `-p` 显示 PID，`-u` 显示用户 |
| 管理服务 | `systemctl 选项 服务名` | start/stop/restart/status/enable |

---


