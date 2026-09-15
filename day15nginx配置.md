nginx
它可以处理静态文件，比如用户图片等，当用户要查找数据，登陆时他就把请求转发给后面的程序，再把结果带回来，它还可以实现负载均衡，当后面有好几台服务器，他把用户均匀地分配过去，防止某一台服务器被挤爆
它采用以下分层结构
main (全局块)
└── events (事件块)
└── http (HTTP块)
├── server (虚拟主机块)
│ └── location (URI匹配块)
└── upstream (负载均衡块)
它的主要配置文件为/etc/nginx/nginx.conf
它包含了如上所示的模块配置，并且其中有一条include语句来引入虚拟机配置
虚拟机配置
它相当于一个标识，让nginx知道去哪里执行任务，在/etc/nginx/目录下，除了nginx.conf，其余绝大部分都为虚拟机配置文件，它们都被主配置文件中一条include /etc/nginx/conf.d/*.conf引入
/etc/nginx的大致目录结构如下
/etc/nginx/
├── nginx.conf # 主配置文件
├── conf.d/ # 虚拟主机配置目录
│ ├── default.conf
│ └── example.com.conf
├── sites-available/ # 可用站点配置（Debian系）
├── sites-enabled/ # 已启用站点配置（符号链接）
├── mime.types # MIME类型定义
└── ssl/ # SSL证书目录
/var/log/nginx/
├── access.log # 访问日志
└── error.log # 错误日志
/usr/share/nginx/html/ # 默认静态文件根目录
下面逐一讲解配置文件中每个模块的配置：
域名匹配，这一步让nginx知道访问到那个域名时，把任务交给谁来处理,格式如下：
server {
listen 监听端口
server_name 域名;
}
其中匹配域名有多种匹配方式
1，精确匹配
http://www.example.com
2.通配符匹配
*.example.com
匹配以example.com结尾的域名
3.匹配多个域名，使用空格分隔
example.com http://www.example.com
4.正则匹配
~^www\d+examplecom $ ;
匹配以www开头+数字+.example.com的域名
5.默认匹配
server_name _;
约定俗成的写法，用于兜底处理未知域名的请求
优先级：
精确匹配 > 通配符在前 > 通配符在后 > 正则匹配 > 默认匹配
location：URI匹配规则
这里将它分为两部分便于理解location = 文件路径 {}，括号外称为外部，括号内称为内部
外部：告诉nignx去哪里执行任务
location = 文件路径{} 精确匹配某一个文件路径
location ^~/文件路径 {} 匹配以此文件路径开头的路径
location ~后缀名 $ {} 匹配以此后缀名结尾的文件路径并区分大小写
location ~*(jpg|png|gif) $ 匹配图片文件不区分大小写
location /api {} 匹配以/api开头的文件路径
优先级：精确匹配 > ^~前缀匹配 > 正则匹配 > 普通前缀匹配
内部：
proxy_pass 打包信息并转发
proxy_pass 协议：//地址（域名）：端口 /可选路径
若在结尾处添加/则替换前缀（将匹配到的location前缀替换为该URI），不添加/则原样转发完整URI
例如：
proxy_pass http://127.0.0.1:8080
将信息打包并转发给本机端口为8080的程序
proxy_set_header 添加/修改请求头
通常与proxy_pass连用
proxy_set_header 标签名 标签值
常用的标签名与标签值：
Host $ host 传递用户的原始域名
X-Real-IP $ remote_addr 传递客户端的真实ip
X-Forwarded-For $ proxy_add_x_forwarded_for 记录客户端完整的代理链路ip
X-Forwarded-Proto $ scheme 告知原始请求协议（http或https）
负载均衡：
upstream backend {}
{}依次填写的值为
顺序：
轮询 默认为此方式，即什么都不写
least_conn 最少连接数（注意是下划线）
ip_hash 计算ip哈希值
随后填写以下内容，这里举一个例子：
server 10.0.0.1:8080 weight=3; 此ip地址上端口为8080的服务器它所接受的请求是其他服务器的三倍，weight表示权重，后面的数字即为接受多少倍的请求，一般配置在性能较强的服务器上
server 10.0.0.2:8080 ; 此ip此ip地址上端口为8080的服务器它正常接受请求
server 10.0.0.3:8080 backup; 当以上服务器都不可用时，此ip此ip地址上端口为8080的服务器它可以接受请求
如此完整的配置组合起来为：
upstream backend {
server 10.0.0.1:8080 weight=3;
server 10.0.0.2:8080;
server 10.0.0.3:8080 backup;
}
root/alias 静态文件路径
location /images {
root /var/www/html
}
在访问/images下的文件时，将其路径拼接为访问/var/www/html/images下的文件
location /images {
alias /var/www/html
}
在访问/images下的文件时改为访问/var/www/html/下的文件
try_files 
u
r
i
uri uri/ /index.html
其含义为查找真实文件，如果不存在就去对应文件夹下查找，如果还不存在就返回index.html界面，此操作通常与root连用，是SPA单页应用防刷新404的核心配置
接下来演示一个具体的nginx的配置：
server {
listen 80;
server_name example.com http://www.example.com;
# 静态文件服务
location / {
root /var/www/example;
index index.html;
try_files 
u
r
i
uri uri/ /index.html;
}
# API 反向代理
location /api/ {
proxy_pass http://127.0.0.1:3000/;
proxy_set_header Host $ host;
proxy_set_header X-Real-IP $ remote_addr;
proxy_set_header X-Forwarded-For $ proxy_add_x_forwarded_for;
}
# 静态资源缓存
location ~* (css|js|jpg|png|gif|ico) $ {
root /var/www/example;
expires 30d;
add_header Cache-Control "public, no-transform";
}
# 自定义错误页
error_page 404 /404.html;
location = /404.html {
root /var/www/example;
internal;
}
# 访问日志
access_log /var/log/nginx/example_access.log;
error_log /var/log/nginx/example_error.log;
}
注意当配置完成后，使用nginx -t查看是否存在语法错误，随后nginx -s reload平滑重载服务（不中断现有连接），严禁直接systemctl restart nginx重启服务





























