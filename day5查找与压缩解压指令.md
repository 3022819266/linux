查找指令：
find 从指定目录下开始向下遍历查找
用法：
find 搜索范范围 选项
选项如下：
-name 按文件名查找
-user 按拥有者查找
-size 按文件大小查找
例如：查找/home 目录下的1.txt文件
find /home -name 1.txt
查找/home目录下拥有者为lzy的文件
find /home -user lzy
查找/home目录下大小为100m的文件
find /home -size 100m

locate 定位文件所在的目录，速度极快
但在使用locate之前是必须使用updatedb创建数据库
用法：
locate 文件名

which可以查看命令在那个目录下

grep用于过滤查找通常与 | 连用
用法：
grep 选项 查找的内容 源文件
例如：在1.txt文件中查找yes的所在行并且显示行号
grep -n "yes" 1.txt 或者
cat 1.txt | grep -n "yes"

压缩与解压指令：
gzip 用于压缩文件，且只能将文件压缩为后缀为.gz的压缩包
例如：将/home/1.txt压缩
gzip /home/1.txt

gunzip用于解压缩文件
例如：将1.txt.gz解压缩
gunzip /home/1.txt.gz

zip用于压缩文件和文件所在目录
用法：
zip 文件名.zip 源文件名
例如：
将1.txt文件压缩为111.zip
zip 111.zip 1.txt

zip -r 文件名.zip 源文件名
将源目录以及其目录下的所有文件压缩为压缩包
例如：
zip -r 111.zip /home/1.txt

unzip 解压压缩包
用法：
unzip 选项 文件名.zip
将压缩包解压到当前目录下
例如：
unzip 1.zip

unzip -d 指定目录 压缩包所在的目录
将压缩包解压到指定目录
例如：
unzip -d /home/lzy /home/11.zip

tar 打包指令，一般压缩与解压缩都是用这个
tar -zcvf 压缩后的文件名 文件所在位置
用于将多个文件压缩为指定的文件名如果要压缩多个文件只需要用空格隔开

tar -zxvf 压缩包所在的位置 -c 指定文件目录
将压缩包解压至指定目录，C为大写










