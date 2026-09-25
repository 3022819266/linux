
# Linux 查找与压缩解压命令速查手册

> 按功能分类整理，覆盖文件查找（find / locate / which）、内容过滤（grep）与压缩解压（gzip / zip / tar）。

---

## 目录

- [文件查找（find）](#文件查找find)
- [快速定位（locate）](#快速定位locate)
- [查看命令路径（which）](#查看命令路径which)
- [内容过滤查找（grep）](#内容过滤查找grep)
- [压缩与解压（gzip）](#压缩与解压gzip)
- [压缩与解压（zip）](#压缩与解压zip)
- [打包与压缩（tar）](#打包与压缩tar)
- [速记小结](#速记小结)
- [勘误说明](#勘误说明)

---

## 文件查找（find）

`find` 从指定目录开始**向下递归遍历**查找文件，实时搜索，结果精确但速度较慢。

**语法：** `find 搜索范围 选项`

| 选项 | 说明 |
| :--- | :--- |
| `-name` | 按文件名查找 |
| `-user` | 按文件拥有者查找 |
| `-size` | 按文件大小查找 |

**示例：**

```bash
# 查找 /home 目录下名为 1.txt 的文件
find /home -name 1.txt

# 查找 /home 目录下拥有者为 lzy 的文件
find /home -user lzy

# 查找 /home 目录下大小为 100M 的文件
find /home -size 100M
```

> **注：** `find` 的 `-size` 单位需**大写** `M`（`+100M` 表示大于 100M，`-100M` 表示小于 100M，`100M` 表示恰好）。原文 `100m` 为笔误，小写 `m` 不被识别。

---

## 快速定位（locate）

`locate` 通过预建数据库定位文件所在目录，**速度极快**，但结果依赖数据库，可能不含最新创建的文件。

> **前置条件：** 使用前必须先执行 `updatedb` 创建/更新数据库。

**语法：** `locate 文件名`

```bash
# 首次使用或数据库过期时，先更新索引
sudo updatedb

# 定位文件
locate 1.txt
```

---

## 查看命令路径（which）

`which` 用于查看某个命令（可执行文件）实际所在的目录路径。

```bash
# 查看 ls 命令所在目录
which ls

# 查看 python3 命令所在目录
which python3
```

---

## 内容过滤查找（grep）

`grep` 用于按关键词过滤查找，通常与管道符 `|` 连用。

**语法：** `grep 选项 查找的内容 源文件`

| 选项 | 说明 |
| :--- | :--- |
| `-n` | 显示匹配行的行号 |
| `-i` | 忽略大小写 |
| `-v` | 反向匹配（显示不含关键词的行） |

**示例：** 在 `1.txt` 中查找含 `yes` 的行并显示行号

```bash
# 方式一：直接对文件操作
grep -n "yes" 1.txt

# 方式二：配合管道符
cat 1.txt | grep -n "yes"
```

---

## 压缩与解压（gzip）

`gzip` 仅能压缩**单个文件**，生成 `.gz` 压缩包，压缩后**原文件会被删除**。

```bash
# 压缩：将 /home/1.txt 压缩为 1.txt.gz
gzip /home/1.txt

# 解压：将 1.txt.gz 还原为 1.txt
gunzip /home/1.txt.gz
```

> **注意：** `gzip` 不能压缩目录，若需压缩整个目录请改用 `tar` 或 `zip`。

---

## 压缩与解压（zip）

`zip` 可压缩**文件及目录**，生成 `.zip` 压缩包，压缩后**保留原文件**。

**压缩语法：** `zip [选项] 压缩包名.zip 源文件/目录`

| 选项 | 说明 |
| :--- | :--- |
| `-r` | 递归压缩目录及其所有内容 |

**解压语法：** `unzip [选项] 文件名.zip`

| 选项 | 说明 |
| :--- | :--- |
| `-d` | 指定解压目标目录 |

**示例：**

```bash
# 将 1.txt 压缩为 111.zip
zip 111.zip 1.txt

# 压缩目录需加 -r（递归）
zip -r 111.zip /home/lzy/

# 解压到当前目录
unzip 111.zip

# 解压到指定目录
unzip -d /home/lzy /home/111.zip
```

---

## 打包与压缩（tar）

`tar` 是 Linux 中最常用的打包与压缩工具，一般压缩与解压都用它。

| 选项 | 含义 |
| :--- | :--- |
| `-z` | 使用 gzip 压缩/解压 |
| `-c` | 创建压缩包（compress） |
| `-x` | 解压（extract） |
| `-v` | 显示过程（verbose） |
| `-f` | 指定文件名（file） |
| `-C` | 指定解压目标目录（**大写 C**） |

**示例：**

```bash
# 压缩单个文件
tar -zcvf backup.tar.gz /home/1.txt

# 压缩多个文件（空格隔开）
tar -zcvf backup.tar.gz /home/1.txt /home/2.txt /home/3.txt

# 压缩整个目录
tar -zcvf mydir.tar.gz /home/lzy/mydir/

# 解压到当前目录
tar -zxvf backup.tar.gz

# 解压到指定目录（注意 -C 是大写）
tar -zxvf backup.tar.gz -C /home/lzy/
```

---

## 速记小结

| 场景 | 推荐命令 | 特点 |
| :--- | :--- | :--- |
| 精确递归查找 | `find` | 实时遍历，慢但准 |
| 快速定位已知文件名 | `locate` | 查库，需先 `updatedb` |
| 查命令所在路径 | `which` | 直接给出可执行文件路径 |
| 按内容过滤 | `grep` | 常配 `\|` 管道 |
| 单文件压缩 | `gzip` | 生成 `.gz`，删原文件 |
| 文件/目录压缩 | `zip` | 生成 `.zip`，留原文件 |
| 多文件打包压缩 | `tar` | 生成 `.tar.gz`，最常用 |

---


