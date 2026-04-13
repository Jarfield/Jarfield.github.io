---
title: Shell 常用命令整理
cover: /images/Covers/sky.jpg
date: 2026-04-13 12:00:00
tags:
  - Shell
  - Linux
  - 工具使用
categories:
  - 学习笔记
mathjax: false
---

**金培晟** *Jarfield*

Shell 的真正价值不在于“记住很多命令”，而在于把常见操作压缩成一套可重复、可组合、可自动化的工作流。  
这篇文章不追求面面俱到，而是整理我最常用、也最值得长期记住的一批命令。

<!-- more -->

## 1. 文件与目录操作

### 1.1 查看当前位置与目录内容

```bash
pwd
ls
ls -l
ls -la
```

- `pwd`：显示当前工作目录。
- `ls`：列出当前目录文件。
- `ls -l`：显示详细信息。
- `ls -la`：包含隐藏文件。

通常我会直接用：

```bash
ls -lah
```

这样更适合快速看权限、大小和时间。

### 1.2 切换目录

```bash
cd /path/to/dir
cd ..
cd ~
cd -
```

- `cd ..`：回到上一级。
- `cd ~`：回到用户主目录。
- `cd -`：回到上一个目录，来回跳很方便。

### 1.3 创建、复制、移动、删除

```bash
mkdir notes
mkdir -p data/raw/2026

cp file.txt backup.txt
cp -r src_dir dst_dir

mv old.txt new.txt
mv logs /tmp/

rm file.txt
rm -r old_dir
rm -rf build/
```

- `mkdir -p`：递归创建多级目录。
- `cp -r`：复制目录。
- `mv`：既能改名，也能移动。
- `rm -rf`：危险命令，删除前必须确认路径。

真正工作时，我会尽量先用：

```bash
rm -ri target_dir
```

先交互确认，避免误删。

### 1.4 创建空文件与快速写入

```bash
touch run.log
echo "hello" > demo.txt
echo "append line" >> demo.txt
```

- `touch`：新建空文件，或更新时间戳。
- `>`：覆盖写入。
- `>>`：追加写入。

## 2. 查看文件内容

### 2.1 快速看头部、尾部与分页

```bash
cat README.md
head -n 20 train.log
tail -n 50 train.log
tail -f train.log
less train.log
```

- `head`：看前几行。
- `tail`：看后几行。
- `tail -f`：持续追踪日志输出。
- `less`：适合长文件翻页查看，按 `q` 退出。

如果只是看日志，最常用的是：

```bash
tail -f nohup.out
```

## 3. 查找文件与文本

### 3.1 按文件名查找

```bash
find . -name "*.py"
find . -type d -name "checkpoints"
```

- `find . -name "*.py"`：当前目录递归找所有 Python 文件。
- `-type d`：只找目录。

### 3.2 按内容搜索

```bash
grep -R "TODO" .
grep -R "learning rate" logs/
```

但在代码仓库里，我更推荐：

```bash
rg "TODO"
rg "main\(" src/
rg --files
```

- `rg` 是 ripgrep，速度通常比 `grep` 更快。
- `rg --files`：快速列出仓库文件。

## 4. 压缩、解压与传输

### 4.1 tar

```bash
tar -czvf archive.tar.gz data/
tar -xzvf archive.tar.gz
```

- `c`：创建归档。
- `x`：解压。
- `z`：gzip。
- `v`：显示过程。
- `f`：指定文件名。

### 4.2 zip / unzip

```bash
zip -r result.zip result/
unzip result.zip
```

### 4.3 scp 远程拷贝

```bash
scp local.txt user@server:/path/
scp -r project/ user@server:/path/
scp user@server:/path/train.log .
```

这个在服务器和本地之间搬日志、模型、实验脚本时非常常见。

## 5. 权限与环境

### 5.1 修改权限

```bash
chmod +x run.sh
chmod 644 config.yaml
chmod -R 755 scripts/
```

- `chmod +x run.sh`：让脚本可执行。
- `644`：文件常见权限。
- `755`：目录和可执行脚本常见权限。

### 5.2 查看当前机器与环境

```bash
whoami
hostname
uname -a
env
```

### 5.3 查看磁盘与内存

```bash
df -h
du -sh checkpoints/
free -h
```

- `df -h`：看磁盘整体使用。
- `du -sh dir/`：看某个目录占多大。
- `free -h`：看内存。

## 6. 进程与后台运行

这一部分是最实用的。  
很多训练、部署、下载、爬取都不会在前台傻等。

### 6.1 最基础：在后台运行

```bash
python train.py &
```

- `&`：把命令放到后台。
- 但如果你直接关闭终端，程序通常也会被挂掉。

所以单独使用 `&` 通常不够稳。

### 6.2 nohup

```bash
nohup python train.py > train.log 2>&1 &
```

这是最常见的长任务启动方式之一。

含义拆开看：

- `nohup`：终端关闭后仍继续运行。
- `> train.log`：标准输出写到日志。
- `2>&1`：标准错误并到标准输出。
- `&`：放后台。

启动后常用检查方式：

```bash
ps -ef | grep train.py
tail -f train.log
```

如果你没手动指定日志，默认输出通常会进 `nohup.out`。

### 6.3 jobs / fg / bg

```bash
jobs
bg %1
fg %1
```

- `jobs`：查看当前 shell 的后台任务。
- `bg`：继续后台运行。
- `fg`：切回前台。

这更适合临时挂起和恢复，而不是长期跑任务。

### 6.4 screen

如果任务要跑很久，而且你希望之后重新接回会话，`screen` 很有用。

#### 创建会话

```bash
screen -S train
```

#### 在会话里执行命令

```bash
python train.py
```

#### 暂时离开会话

按：

```text
Ctrl + A, D
```

这叫 detach，会话和程序都会继续运行。

#### 查看已有会话

```bash
screen -ls
```

#### 重新进入会话

```bash
screen -r train
```

或者：

```bash
screen -r 会话ID
```

#### 结束会话

在会话内直接退出程序，再执行：

```bash
exit
```

### 6.5 tmux

很多人会用 `tmux` 代替 `screen`，因为窗口管理更舒服。

常见最小操作是：

```bash
tmux new -s train
tmux ls
tmux attach -t train
```

如果你的服务器没装 `screen`，可以看一下是否有 `tmux`。

## 7. 查看和结束进程

### 7.1 查进程

```bash
ps -ef | grep python
pgrep -af train.py
top
htop
```

- `ps -ef | grep xxx`：老办法，通用。
- `pgrep -af xxx`：更干净。
- `top` / `htop`：实时看资源。

### 7.2 杀进程

```bash
kill PID
kill -9 PID
pkill -f train.py
```

- `kill PID`：先优雅结束。
- `kill -9 PID`：强制结束，不到必要别先上。
- `pkill -f train.py`：按命令行匹配杀进程。

## 8. 一个比较稳的训练任务流程

我自己更常用下面这套：

```bash
mkdir -p logs
nohup python train.py --config configs/train.yaml > logs/train_20260413.log 2>&1 &
tail -f logs/train_20260413.log
```

如果是需要长期维护、反复回到同一环境的任务，我会改用：

```bash
screen -S experiment
python train.py --config configs/train.yaml
```

这样中途断开 SSH 也不影响。

## 9. 如何把更新同步到你这个网站

你现在这个仓库已经改成了：

- 本地写 Hexo 源码
- 推送到 GitHub
- GitHub Actions 自动构建并发布到 Pages

因此，更新一篇文章的流程是：

### 9.1 本地预览

```bash
npm run server
```

浏览器打开本地地址，确认文章内容和格式没问题。

### 9.2 本地构建检查

```bash
npm run build
```

确保 Hexo 能正常生成静态文件。

### 9.3 提交并推送

```bash
git add .
git commit -m "Add shell commands note"
git push
```

### 9.4 GitHub 自动发布

推送之后，GitHub 仓库里的 Actions 会自动运行。  
工作流成功后，网站就会更新到：

```text
https://Jarfield.github.io/
```

## 10. 最后给自己留一份最小速查表

```bash
# 文件
pwd
ls -lah
mkdir -p dir/subdir
cp -r src dst
mv old new
rm -ri target

# 查找
find . -name "*.py"
rg "keyword"
tail -f train.log

# 压缩
tar -czvf archive.tar.gz data/
tar -xzvf archive.tar.gz

# 后台运行
nohup python train.py > train.log 2>&1 &
screen -S train
screen -ls
screen -r train

# 进程
ps -ef | grep python
pgrep -af train.py
kill PID
pkill -f train.py

# 网站同步
npm run build
git add .
git commit -m "update post"
git push
```

这篇文章的重点不是“命令越多越好”，而是把真正高频、容易忘、容易出错的那一批先固定下来。  
等后面用得更多，再补 Docker、SSH、rsync、conda 和系统监控也不迟。
