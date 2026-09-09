ps 查看当前进程
ps -aux 可以详细查看进程
当输入以上命令时会显示如下列，每一列所代表的含义各不相同
  system v 展示风格
  user 用户名进程
  pid 进程号
  %cpu 进程占用的cpu百分比
  %MEM 进程占用物理内存的百分比
  vsz 进程占用虚拟内存的大小
  Rss 进程占用的物理内存大小
  TT 终端名称
  STAT 进程状态 其中：
    S 睡眠 s 该进程是会话的先导进程 N 低优先级 R 正在运行 D 短期等待 Z 僵死进程 T 被跟踪或者被停止的进程
  STARTED 进程启动时间
  TIME cpu时间，进程使用cpu的总时间
  Command 启动进程所用的命令参数，过长会被截断显示

  ps -ef 以全格式显示当前所有的进程，显示信息包括一下内容：
    UID 用户ID
    pid 进程ID
    ppid 父进程id
    c cpu用于计算优先级的引子，数字越大优先级越低，数字越小优先级越高
    stime 进程启动时间
    TTY 完整的终端名称
    CMD 启动进程所用的命令和参数

  终止进程
  kill 进程号
  killall 进程名称 这个命令终止进程时会终止其下的所有子进程
  使用命令时加上-9 可以强制终止命令

  pstree 选项 可以更加直观的观察进程信息
  -p 显示进程的pid
  -u 显示进程的所属用户

服务管理
  systemctl 服务 选项
  选项可选start stop restart reload 
  查看全部系统服务
  systemctl list-unit-files --type=service --all



  
  
