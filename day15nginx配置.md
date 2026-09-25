
# Linux Nginx 配置速查

> 涵盖 Nginx 核心定位、配置文件分层结构、`server_name` 域名匹配、`location` URI 规则、反向代理与负载均衡、静态文件服务，以及一份完整的站点配置示例。完全符合 GitHub Flavored Markdown 标准，可直接复制粘贴或拖入仓库渲染。

## 目录

- [Nginx 是做什么的](#nginx-是做什么的)
- [配置文件分层结构](#配置文件分层结构)
- [目录结构速览](#目录结构速览)
- [server_name：域名匹配](#server_name域名匹配)
- [location：URI 匹配规则](#locationuri-匹配规则)
- [反向代理：proxy_pass 与 proxy_set_header](#反向代理proxy_pass-与-proxy_set_header)
- [负载均衡：upstream](#负载均衡upstream)
- [静态文件路径：root 与 alias](#静态文件路径root-与-alias)
- [try_files：SPA 防刷新 404](#try_filesspa-防刷新-404)
- [完整配置示例](#完整配置示例)
- [配置生效与重载](#配置生效与重载)
- [速记小结](#速记小结)


---

## Nginx 是做什么的

Nginx 是一个高性能的 **Web 服务器 / 反向代理 / 负载均衡器**，常见三类职责：

| 职责 | 说明 |
| :--- | :--- |
| **静态文件服务** | 直接返回图片、CSS、JS、HTML 等静态资源，无需后端程序参与 |
| **反向代理** | 当用户请求动态数据（如登录、查库）时，把请求转发给后端程序，再把结果带回给用户 |
| **负载均衡** | 后端有多台服务器时，把用户请求均匀分配过去，防止单台被打爆 |

---

## 配置文件分层结构

Nginx 配置采用层层嵌套的块（block）结构：

```text
main (全局块)
└── events (事件块)
    └── http (HTTP 块)
        ├── server (虚拟主机块)
        │   └── location (URI 匹配块)
        └── upstream (负载均衡块)
```

- 主配置文件为 **`/etc/nginx/nginx.conf`**，其中包含上述各模块，并通过一条 `include` 语句引入所有虚拟主机配置。
- **虚拟主机配置**（vhost）相当于一个"标识"，告诉 Nginx 访问到某个域名/端口时去哪里执行任务。
- 在 `/etc/nginx/` 目录下，除 `nginx.conf` 外绝大部分都是虚拟主机配置文件，它们都由主配置中的 `include /etc/nginx/conf.d/*.conf;` 引入。

> [!IMPORTANT]
> 原文"虚拟机配置"应为"**虚拟主机配置**（virtual host）"，与虚拟化"虚拟机"无关。

---

## 目录结构速览

```text
/etc/nginx/
├── nginx.conf              # 主配置文件
├── conf.d/                 # 虚拟主机配置目录
│   ├── default.conf
│   └── example.com.conf
├── sites-available/        # 可用站点配置（Debian 系）
├── sites-enabled/          # 已启用站点配置（符号链接）
├── mime.types              # MIME 类型定义
└── ssl/                    # SSL 证书目录

/var/log/nginx/
├── access.log              # 访问日志
└── error.log               # 错误日志

/usr/share/nginx/html/      # 默认静态文件根目录
```

> [!NOTE]
> `sites-available` / `sites-enabled` 是 Debian/Ubuntu 的组织习惯，靠"软链接启用站点"；RHEL/CentOS 默认只用 `conf.d/*.conf`。

---

## server_name：域名匹配

让 Nginx 知道访问到哪个域名时把任务交给哪个 `server` 块处理：

```nginx
server {
    listen      80;
    server_name example.com www.example.com;
}
```

`server_name` 支持多种匹配方式：

| 方式 | 写法 | 说明 |
| :--- | :--- | :--- |
| **精确匹配** | `www.example.com` | 域名完全相等 |
| **通配符在前** | `*.example.com` | 匹配以 `example.com` 结尾的域名（如 `a.example.com`） |
| **多域名** | `example.com www.example.com` | 用空格分隔多个域名 |
| **正则匹配** | `~^www\d+\.example\.com$` | 以 `www`+数字+`.example.com` 结尾 |
| **默认（兜底）** | `server_name _;` | 约定俗成，处理所有未匹配到的请求 |

> [!WARNING]
> `server_name` 只写**域名**，**不要带 `http://` 协议前缀**（原文示例误写为 `http://www.example.com`）。

**匹配优先级**：精确匹配 > 通配符在前（`*.example.com`） > 通配符在后（`www.example.*`） > 正则匹配 > 默认匹配 `_`。

---

## location：URI 匹配规则

`location` 分为"外部（匹配语法）"与"内部（执行动作）"两部分。

### 外部：匹配语法

| 语法 | 含义 |
| :--- | :--- |
| `location = /path {}` | **精确匹配**某一个路径 |
| `location ^~ /path {}` | 前缀匹配，命中后**不再**看正则 |
| `location ~ /suffix$ {}` | **正则匹配**，区分大小写 |
| `location ~* (jpg\|png\|gif)$ {}` | 正则匹配，**不区分大小写**（如匹配图片） |
| `location /api {}` | 普通前缀匹配，以 `/api` 开头 |

**匹配优先级**：精确匹配 `=` > `^~` 前缀匹配 > 正则匹配 `~`/`~*` > 普通前缀匹配。

> [!TIP]
> `^~` 与 `~`/`~*` 的先后关系是常见考点：一旦命中 `^~`，Nginx 立即停止正则匹配。

---

## 反向代理：proxy_pass 与 proxy_set_header

### proxy_pass：打包并转发请求

```nginx
proxy_pass 协议://地址或域名:端口/可选路径;
```

- **结尾带 `/`**：会**替换** `location` 匹配到的前缀为该 URI。
- **结尾不带 `/`**：**原样转发**完整 URI。

```nginx
# 转发给本机 8080 端口的后端程序
location /api/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

### proxy_set_header：添加/修改请求头

通常与 `proxy_pass` 连用，把客户端真实信息传递给后端：

```nginx
proxy_set_header 头部名 头部值;
```

| 常用头部 | 值 | 作用 |
| :--- | :--- | :--- |
| `Host` | `$host` | 传递用户访问的原始域名 |
| `X-Real-IP` | `$remote_addr` | 传递客户端真实 IP |
| `X-Forwarded-For` | `$proxy_add_x_forwarded_for` | 记录完整的代理链路 IP |
| `X-Forwarded-Proto` | `$scheme` | 告知原始请求协议（http/https） |

> [!WARNING]
> 变量必须紧跟 `$`，**中间不能有空格**（原文误写为 `$ host`、`$ remote_addr`，正确是 `$host`、`$remote_addr`）。

---

## 负载均衡：upstream

```nginx
upstream backend {
    # 调度算法写在 server 之前
    server 10.0.0.1:8080 weight=3;   # 权重：接收请求约为其它服务器的 3 倍
    server 10.0.0.2:8080;            # 正常接收请求
    server 10.0.0.3:8080 backup;     # 备机：以上都不可用时才接管
}
```

### 调度算法（写在 server 行之前）

| 算法 | 写法 | 说明 |
| :--- | :--- | :--- |
| **轮询** | 默认（什么都不写） | 依次轮流分配 |
| **最少连接** | `least_conn;` | 优先分给当前连接数最少的后端 |
| **IP 哈希** | `ip_hash;` | 按客户端 IP 哈希，同一 IP 固定落到同一后端（会话保持） |

### server 行常用参数

| 参数 | 说明 |
| :--- | :--- |
| `weight=N` | 权重，性能强的机器配更高值 |
| `backup` | 热备，仅主池全挂时启用 |
| `down` | 临时标记该节点下线 |
| `max_fails` / `fail_timeout` | 失败次数与冷却时间，用于被动健康检查 |

> [!NOTE]
> `ip_hash` 与 `least_conn` 二者只能选其一，且注意是**下划线**拼写。

---

## 静态文件路径：root 与 alias

两者都能定位静态文件，但**拼接规则不同**：

```nginx
# root：把 location 路径追加到 root 之后
location /images {
    root /var/www/html;
}
# 访问 /images/a.jpg  → 实际读取 /var/www/html/images/a.jpg

# alias：用 alias 路径直接替换 location 路径
location /images/ {
    alias /var/www/html/;
}
# 访问 /images/a.jpg  → 实际读取 /var/www/html/a.jpg
```

| 指令 | 路径拼接方式 | 典型用途 |
| :--- | :--- | :--- |
| `root` | `root` + 完整 URI | 目录结构与对外路径一致 |
| `alias` | 用 `alias` 替换掉 `location` 前缀 | 对外路径与磁盘路径不一致（如虚拟子目录） |

---

## try_files：SPA 防刷新 404

```nginx
try_files $uri $uri/ /index.html;
```

含义：先按 `$uri` 找真实文件 → 找不到则当作目录找 `$uri/` → 仍找不到就返回 `/index.html`。

> [!TIP]
> 这是 **SPA（单页应用，如 Vue/React）路由防刷新 404 的核心配置**，通常与 `root` 连用。前端路由的深层地址（如 `/user/1`）在磁盘上并不存在真实文件，必须兜底回 `index.html` 交给前端路由处理。

---

## 完整配置示例

```nginx
server {
    listen      80;
    server_name example.com www.example.com;

    # 静态文件服务
    location / {
        root      /var/www/example;
        index     index.html;
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理
    location /api/ {
        proxy_pass         http://127.0.0.1:3000/;
        proxy_set_header   Host       $host;
        proxy_set_header   X-Real-IP  $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 静态资源缓存
    location ~* \.(css|js|jpg|jpeg|png|gif|ico)$ {
        root  /var/www/example;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # 自定义错误页
    error_page 404 /404.html;
    location = /404.html {
        root     /var/www/example;
        internal;
    }

    # 访问 / 错误日志
    access_log /var/log/nginx/example_access.log;
    error_log  /var/log/nginx/example_error.log;
}
```

---

## 配置生效与重载

```bash
# 1. 检查配置语法是否正确
nginx -t

# 2. 平滑重载（不中断现有连接）
nginx -s reload
```

> [!WARNING]
> 修改配置后务必先 `nginx -t` 校验语法，再用 `nginx -s reload` **平滑重载**；**严禁**直接 `systemctl restart nginx` 重启服务，那会中断所有现有连接。

---

## 速记小结

| 场景 | 关键指令 / 写法 | 要点 |
| :--- | :--- | :--- |
| 改完配置验证 | `nginx -t` | 先测试再重载 |
| 不中断生效 | `nginx -s reload` | 替代 restart |
| 域名分流 | `server_name` | 只写域名不带 `http://` |
| URI 路由 | `location` | `=` > `^~` > `~` > 普通前缀 |
| 转发后端 | `proxy_pass` | 结尾带 `/` 会替换前缀 |
| 传真实 IP | `proxy_set_header X-Real-IP $remote_addr;` | 变量 `$` 后无空格 |
| 负载均衡 | `upstream` + `least_conn`/`ip_hash` | 算法写在 server 前 |
| 静态路径 | `root` 追加 / `alias` 替换 | 二者拼接规则不同 |
| SPA 防 404 | `try_files $uri $uri/ /index.html;` | 配合 root 使用 |

---

