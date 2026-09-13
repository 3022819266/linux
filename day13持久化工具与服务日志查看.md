持久化工具
nohup 防挂断
当终端断开后可以让程序继续在后台运行，适用于一次性无需交互的快速后台任务，比如简单的数据备份或文件传输
nohup 命令/服务/程序 > 要写入的日志文件 选项 &
选项:
0 输入
1 输出
2 报错
2>&1 将输入与报错合并到一起写入
例如
nohup python 1.py > 1.txt 2>&1 &
执行1.python这个文件并将其执行的输出与报错写入到 1.txt文件中

screen/tmux 会话保持
当终端断开后，保留当前工作状态，重连后可以恢复到之前的工作状态

screen
screen -S 终端名 创建一个终端
ctrl+a 然后按d 退出终端
screen -r 终端名 重连终端

tmux
tmux new -s 终端名 创建终端
ctrl+b 然后按d 退出终端
tmux attach -t 终端名 重连终端

scree/tmux适用于长时间运行，需要随时回来查看输出或进行交互的复杂操作
tmux比screen更推荐

服务日志查看
jorunalctl是systemed的集中式日志查询工具，用于检索有systemd-journald收集的结构化日志
journalctl -u 服务名 查看服务的所有日志
journalctl -u 服务名 -f 可以同tail -f 一样监控服务的最新日志输出
journalctl -u 服务名 --since “时间段” / -p err 可以查看服务对应时间段的日志输出/错误的日志












