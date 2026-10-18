---
title: Linux 常用命令
tags:
  - 工具
  - Linux
  - Shell
---

## 介绍 / Introduction

Linux / macOS 下高频 Shell 命令速查，覆盖文件操作、进程管理、网络与权限。

!!! note
    Windows 用户推荐在 WSL2 或 Git Bash 下使用这些命令。

> 按"从进入系统到排查问题"的实际操作顺序组织，覆盖文件、文本、进程、网络、服务、磁盘、权限、日志等日常场景。带 `< >` 的部分需替换为实际值，`[ ]` 表示可选参数。示例基于 CentOS/RHEL 与 Ubuntu/Debian 通用语法，发行版差异处会单独标注。

---

## 一、登录、身份与会话

```bash
ssh user@192.168.1.10              # 远程登录
ssh -p 2222 -i ~/.ssh/id_rsa user@host   # 指定端口与密钥
ssh-copy-id user@host              # 把本机公钥追加到对方 authorized_keys
exit / logout                      # 退出登录
who                                # 当前在线用户
w                                  # 在线用户 + 在做什么 + 负载
last -n 20                         # 最近登录记录
lastb                              # 登录失败记录（需 root）
id                                 # 当前用户的 uid/gid/所属组
whoami                             # 当前用户名
hostnamectl                        # 主机名、系统、内核版本
uptime                             # 运行时长、负载均值
date '+%Y-%m-%d %H:%M:%S'          # 格式化当前时间
timedatectl                        # 时区与 NTP 同步状态
echo $SHELL; echo $LANG            # 当前 shell 与语言环境
```

---

## 二、目录与文件基础操作

### 定位与浏览

```bash
pwd                                # 当前所在目录绝对路径
ls -l                              # 长格式（权限、属主、大小、时间）
ls -lh                             # 大小以 K/M/G 显示
ls -a                              # 包含隐藏文件（.开头）
ls -lt                             # 按修改时间倒序（最新在前）
ls -S                              # 按大小排序
tree -L 2                          # 两层目录树
which nginx                        # 查命令的可执行文件路径
type -a python3                    # 命令来源（内置/别名/路径）
whereis mysql                      # 二进制、源码、man 页位置
```

### 增删改移

```bash
mkdir /data/logs                   # 创建目录
mkdir -p a/b/c                     # 递归创建多级（不存在才创建）
rmdir empty_dir                    # 删除空目录
cp a.txt /backup/                  # 复制文件到目录
cp -r src/ dst/                    # 递归复制目录
cp -a src/ dst/                    # 归档模式，保留权限、属主、时间戳、软链
cp -n a.txt /dst/                  # 目标已存在则不覆盖
mv a.txt b.txt                     # 改名
mv ./downloads/*.pdf /data/pdf/    # 移动
rm file.txt                        # 删除文件
rm -r old_dir/                     # 递归删除目录
rm -rf /data/tmp/*                 # 强制递归删除（⚠️ 高危，执行前务必核对路径）
ln -s /opt/app/current /app        # 创建软链接（推荐）
ln bigfile bigfile.hardlink        # 硬链接
readlink -f /app                   # 解析软链接到真实路径
touch newfile.conf                 # 创建空文件或更新时间戳
```

> ⚠️ `rm -rf` 不可恢复，且没有回收站。删除前建议先用 `ls -d <目标>/*` 确认范围，重要数据先 `mv` 到临时目录观察几天再删。绝不执行 `rm -rf /` 或带变量的 `rm -rf $DIR/`（`$DIR` 为空时等于删根）。

### 查看内容

```bash
cat f.txt                          # 全文输出（适合小文件）
cat -n f.txt                       # 带行号
tac f.txt                          # 倒序输出
head -n 20 f.txt                   # 前 20 行
tail -n 50 f.txt                   # 后 50 行
tail -f app.log                    # 实时跟踪（文件被轮转后要重开）
tail -F app.log                    # 跟踪并按名字重试，适合日志切割场景
less f.txt                         # 分页浏览：空格下翻页、b 上翻、/词 搜索、n 下一个、q 退出
more f.txt                         # 简单分页
nl f.txt                           # 显示行号（可配合 grep 定位）
od -c f.bin                        # 十六进制/字符查看二进制
file <路径>                         # 判断文件类型
```

---

## 三、查找与压缩

### 按名字、内容、属性查找

```bash
find /var -name "*.log"                       # 按名字（-iname 忽略大小写）
find /data -type f -size +100M                # 大于 100M 的文件
find /tmp -type d -mtime -1                   # 1 天内修改过的目录
find . -name "*.conf" -exec cp {} /backup/ \; # 找到就执行（{} 代指结果）
find /var/log -name "*.log" -mtime +30 -delete  # 删除 30 天前的日志（⚠️ 先去掉 -delete 看清单）
locate nginx.conf                             # 走数据库索引，快（需先 updatedb）
```

```bash
grep -n "ERROR" app.log                # 显示行号
grep -i "error" app.log                # 忽略大小写
grep -r "timeout" /etc/nginx/          # 递归搜索目录
grep -rn --include="*.py" "def main" . # 限定文件类型
grep -A 3 -B 3 "OOM" syslog            # 显示匹配行前后 3 行
grep -c "failed" auth.log              # 只统计条数
grep -v "DEBUG" app.log                # 反向过滤（排除）
grep -E "err|fail" app.log             # 扩展正则（等价 egrep）
zgrep "Out of memory" /var/log/*.gz    # 直接搜压缩包
```

### 压缩与解压

```bash
tar czf backup.tar.gz /data/db         # 打包并 gzip 压缩
tar xzf backup.tar.gz -C /restore/     # 解压到指定目录
tar tjf backup.tar.gz                  # 只列出内容不解压
tar cjf backup.tar.xz dir/             # xz 压缩率更高、更慢
tar -C /data -czf /tmp/logs.tar.gz logs/   # 指定基准目录，避免绝对路径前缀
unzip app.zip -d /opt/                 # 解压 zip
unrar x archive.rar                    # 需安装 rar unrar
gzip f.txt                             # 单文件压缩成 f.txt.gz（原文件消失）
gunzip f.txt.gz                        # 解压
zip -r site.zip ./dist                 # 递归打包成 zip
```

---

## 四、文本处理与统计（三剑客）

```bash
awk '{print $1, $3}' access.log        # 按列输出（默认空格分隔）
awk -F',' 'NR>1 {print $2}' data.csv   # 指定分隔符，跳过表头
awk '$9==502 {c++} END{print c}' access.log           # 统计 502 数量
awk '{sum+=$5} END{print sum/NR}' stats.txt           # 求第 5 列平均值
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
                                        # 访问来源 Top 10（组合技）
cut -d: -f1 /etc/passwd                # 按分隔符取字段
sort -k2 -n -r scores.txt              # 按第 2 列数值倒序
uniq -c                                # 去重并计数（必须先 sort）
wc -l f.txt                            # 行数（-c 字节、-m 字符、-w 词）
tr -d '\r' < win.txt > unix.txt        # 删掉 Windows 的回车符
paste a.txt b.txt                      # 按行并列合并
```

### 替换与编辑

```bash
sed -i.bak 's/http:/https:/g' nginx.conf    # 原地替换并备份
sed -n '10,20p' f.txt                       # 只打印 10-20 行
sed -i '/^#/d' config                       # 删除所有注释行
sed -i 's/foo/bar/gi' f.txt                 # g 全局、i 忽略大小写
awk 'NR>=100 && NR<=200' big.txt            # 截取行区间
```

---

## 五、权限、属主与特殊属性

```bash
chmod 644 f.txt                        # 属主读写、其他人只读
chmod 755 dir/                         # 目录需 x 才能进入
chmod u+x script.sh                    # 给属主加执行位
chmod -R go-w /data                    # 递归去掉组和其他人的写权限
chown appuser:appgroup /opt/app        # 改属主与属组
chown -R www:www /var/www              # 递归改
chgrp staff f.txt                      # 只改属组

ls -l /usr/bin/passwd                  # 看到 -r-sr-xr-x 即 SUID
chmod u+s /path/bin                    # SUID：以文件属主身份运行
chmod g+s dir/                         # SGID：目录内新文件继承目录属组
chmod 1777 /tmp                        # 粘滞位：只有属主能删自己的文件
```

| 数字 | 含义 |
|---|---|
| 7 | rwx（读 4 + 写 2 + 执行 1） |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 0 | --- |

> 排查"文件打不开/写不了"的完整顺序：`ls -l` 看权限位 → `id` 看自己身份 → `namei -l /完整路径`（逐级检查每一层目录是否有 x 权限）→ `getenforce` / `ausearch -m avc`（SELinux）→ `df -h`（磁盘满也会导致写入失败）。

---

## 六、进程、服务与任务计划

### 进程

```bash
ps aux | grep nginx                    # 查某进程详情
ps -ef --forest                        # 树形展示父子关系
ps -eo pid,ppid,user,etime,cmd --sort=-%mem | head   # 按内存排序、看运行时长
top                                    # 动态总览（M 按内存排序、P 按 CPU、1 看各核、k 杀进程、q 退出）
htop                                   # 更好用的 top（需安装）
pidof java                             # 按名字查 PID
kill 12345                             # 默认 SIGTERM，给进程优雅退出机会
kill -9 12345                          # SIGKILL 强制（无法清理资源，最后手段）
kill -HUP 12345                        # 重载配置（nginx、sshd 等约定用此信号）
pkill -f "app.py"                      # 按命令行模式杀
systemctl kill myservice.service       # 优先用服务方式停止
```

### systemd 服务

```bash
systemctl status nginx                 # 状态 + 最近日志
systemctl start nginx                # 启动（stop 停止、restart 重启）
systemctl reload nginx                 # 不中断地重载配置
systemctl enable nginx                 # 开机自启
systemctl disable nginx                # 取消自启
systemctl is-enabled nginx
systemctl list-units --type=service --state=running   # 正在运行的服务
systemctl list-units --failed          # 启动失败的服务（排障先看这里）
systemctl daemon-reload                # 改完 .service 文件后必须执行
journalctl -u nginx -f                 # 跟踪某服务日志
journalctl -u nginx --since "10 min ago"
journalctl -p err -b                   # 本次开机的错误级日志
journalctl --disk-usage                # 日志占用空间
journalctl --vacuum-size=200M          # 限制日志体积
```

### 定时任务

```bash
crontab -e                             # 编辑当前用户的定时任务
crontab -l                             # 查看
crontab -r                             # 删除全部（⚠️ 不可逆，先 -l 备份）

# 格式： 分 时 日 月 周  命令
0 3 * * * /usr/local/bin/backup.sh     # 每天 03:00
*/10 * * * * curl -s http://localhost/health  # 每 10 分钟
30 9 * * 1-5 report.sh                 # 工作日 09:30
0 0 1 * * monthly.sh                   # 每月 1 日 00:00
@reboot sync.sh                        # 开机执行一次

systemctl list-timers                  # 查看 systemd 定时器
```

> 注意：cron 里不加载交互式环境变量，命令务必用绝对路径；`%` 在 crontab 中是特殊字符需转义；日志默认发邮件，建议在行尾加 `>> /var/log/cron.log 2>&1`。

---

## 七、网络诊断与配置

```bash
ip a                                   # 查看网卡与地址（推荐）
ip route                               # 路由表
ip neigh                               # ARP 邻居
ss -tlnp                               # 监听中的 TCP 端口 + 占用进程（推荐）
ss -s                                  # 连接数总览
ss -tn state established               # 已建立连接
netstat -antp                          # 老命令，等价可用
ping -c 4 8.8.8.8                      # 连通性（发 4 个包）
traceroute -n host                     # 路径（可能跳数受限）
mtr -rwnc 20 host                      # ping+traceroute 合一，排障更好用
dig +short baidu.com                   # DNS 解析结果
nslookup baidu.com                     # 简化查询
host baidu.com
curl -sv https://api.local/health      # 看握手与响应头
curl -o file -L url                    # 跟随重定向下载
curl -X POST -H 'Content-Type: application/json' -d '{"a":1}' http://host/api
wget -c url                            # 支持断点续传下载
telnet host 80                         # 快速验证端口通不通
nc -lvnp 9000                          # 临时监听端口
nc -zv host 22                         # 只探测端口是否可达
```

### 本机网络信息与防火墙

```bash
cat /etc/resolv.conf                   # DNS 服务器
cat /etc/hosts                         # 本地域名映射
hostname -I                            # 本机所有 IP
ethtool eth0 | grep Speed              # 网卡协商速率
nstat -az | grep -i retrans            # TCP 重传计数（丢包排查）
iftop / nload / vnstat                 # 实时带宽监控（需安装）

firewall-cmd --state                   # CentOS/RHEL firewalld
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
iptables -L -n -v --line-numbers       # 查看规则（含命中计数）
ufw status verbose                     # Ubuntu
ufw allow 22/tcp
```

> 端口"服务起来了但外网连不上"的排查链路：本机 `ss -tlnp` 确认监听地址不是 `127.0.0.1` → `iptables`/`firewalld`/`ufw` 是否放行 → 云安全组/上层网络设备 → 对端 `telnet` 验证。

---

## 八、磁盘、内存与系统资源

```bash
df -h                                  # 各挂载点使用率（重点看 / 和 /data）
df -i                                  # inode 使用率（小文件过多会耗尽）
du -sh /var/log                        # 某目录总大小
du -h --max-depth=1 /data | sort -h | tail -20   # 定位占用大户
lsblk                                  # 块设备与分区树
findmnt                                # 当前挂载关系
mount /dev/sdb1 /data                  # 挂载
umount /data                           # 卸载（提示 busy 时用 `lsof +D /data` 找占用进程）
swapon --show                          # swap 使用情况
free -h                                # 内存与 swap（看 available 而不是 free）
vmstat 1 5                             # 每秒采样：r 运行队列、si/so 换页、wa IO 等待
iostat -xz 1                           # 磁盘 util/await（sysstat 包）
dmesg -T | tail -50                    # 内核消息（硬件错误、OOM Kill 都在这）
dmesg -T | grep -i "out of memory"
lsof -p 12345                          # 某进程打开的文件与连接
lsof -i :3306                          # 谁在用某端口
nproc                                  # CPU 核数
cat /proc/cpuinfo | grep "model name" | head -1
uname -a                               # 内核版本与架构
cat /etc/os-release                    # 发行版信息
```

---

## 九、软件安装

```bash
# Ubuntu / Debian
apt update                             # 刷新索引（安装前先跑）
apt install -y nginx                   # 安装
apt remove nginx                       # 卸载（保留配置）
apt purge nginx                        # 卸载并删配置
apt list --installed | grep nginx      # 已安装版本
apt search openssl                     # 搜索包

# CentOS / RHEL 8+（dnf）、7（yum）
dnf install -y nginx
dnf remove nginx
dnf provides "*/nginx"                 # 反查某文件属于哪个包
dnf repolist

# 通用源码安装三步
./configure --prefix=/opt/app
make -j$(nproc)
make install
```

---

## 十、用户、环境变量与登录配置

```bash
useradd -m -s /bin/bash deploy         # 新建用户并创建家目录
usermod -aG docker deploy              # ⚠️ 必须带 -a，否则会把其他组全部清空
usermod -L deploy                      # 锁定账号
passwd deploy                          # 设置密码
groupadd app                           # 新建组
deluser deploy                         # Ubuntu 删用户
getent group docker                    # 确认组存在与成员
sudo -u deploy whoami                  # 以其他用户身份执行
sudo -i                                # 切换到 root 的登录 shell
visudo                                 # 编辑 /etc/sudoers（自带语法检查）
```

```bash
echo $PATH                             # 当前命令搜索路径
export JAVA_HOME=/opt/jdk17            # 当前会话生效
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc   # 写入用户配置
source ~/.bashrc                       # 让改动立即生效
cat /etc/profile.d/*.sh                # 全局环境变量脚本
env | grep -i proxy                    # 代理设置
locale                                 # 语言环境（中文乱码先看这个）
```

> 改动配置的最佳顺序：先确认改的是哪个文件（`~/.bashrc` 只对交互 shell、`~/.profile` 对登录 shell、`/etc/environment` 对图形会话），再 `source` 或重新登录验证；用新 shell 测试能排除缓存干扰。

---

## 十一、日志与故障排查套路

```bash
ls -lh /var/log/                       # 日志目录总览（磁盘满常见元凶）
tail -f /var/log/syslog                # Ubuntu 系统日志
tail -f /var/log/messages              # CentOS 系统日志
journalctl -xe                         # 最近错误 + 跳转编辑器（systemd 首选）
journalctl --since today               # 今天全部
grep -i "error\|fail" app.log | tail -50

last -x | head                         # 登录与开关机历史
cat /var/log/auth.log | grep "Failed password"   # 爆破排查
```

### 常见现象 → 起手命令

| 现象 | 先查什么 |
|---|---|
| 磁盘满 | `df -h` → `du -h --max-depth=1` 定位 → 注意已删除但仍被占用的文件：`lsof | grep deleted` |
| 内存不足 / 服务被杀 | `free -h` → `dmesg -T | grep -i oom` 看被杀进程与 score |
| CPU 100% | `top` 找进程 → `ps -p <pid> -o pid,ppid,etime,pcpu,pmem,cmd` → 采样 `strace -p <pid> -f -tt` 或 `perf top` |
| 负载高但 CPU 不高 | `vmstat 1`、`iostat -xz 1` 看 `wa`，多半是 IO 瓶颈 |
| 连不上某端口 | `ss -tlnp` → 防火墙/安全组 → `nc -zv host port` |
| 域名解析失败 | `cat /etc/resolv.conf` → `dig +short` → `ss -unl | grep :53` 看本机 DNS 服务 |
| 文件"明明存在却打不开" | `namei -l` 逐级查权限、`getenforce` SELinux、`df -i` inode 耗尽 |
| 定时任务没执行 | `grep CRON /var/log/syslog`、检查脚本单独跑是否报错、脚本是否加执行位、路径是否绝对 |

---

## 十二、高频场景速查

| 场景 | 命令 |
|---|---|
| 找谁占着某个端口 | `ss -tlnp \| grep :8080` 或 `lsof -i :8080` |
| 查最近 5 分钟内修改的文件 | `find /data -type f -mmin -5` |
| 按大小清理历史日志 | `find /var/log -name "*.gz" -mtime +7 -delete`（先去 `-delete` 核对清单） |
| 实时看日志并过滤关键字 | `tail -F app.log \| grep --line-buffered ERROR` |
| 打包带日期命名的备份 | `tar czf backup-$(date +%F).tar.gz /data/app` |
| 传输目录到另一台机 | `rsync -avz --progress /data/ user@host:/backup/` |
| 中断后继续同步大文件 | `rsync -av --partial source/ dest/` 或 `wget -c url` |
| 递归改目录权限（Web 场景） | `find /var/www -type d -exec chmod 755 {} \;` 与 `-type f ... 644` |
| 查某命令属于哪个包 | `dnf provides "*/nginx"` / `apt-file search nginx.conf` |
| 看进程打开了哪些文件 | `lsof -p <pid>` 或 `ls -l /proc/<pid>/fd` |
| 统计访问量 Top IP | `awk '{print $1}' access.log \| sort \| uniq -c \| sort -rn \| head` |
| 判断服务能否开机自启 | `systemctl is-enabled <服务>` |
| 系统重启原因 | `uptime`、`last -x reboot shutdown \| head`、`journalctl -b -1 -e` |

---

## 十三、安全与操作习惯

1. **破坏性命令先"演练"**：`rm`/`mv`/`find -delete` 先把动词换成 `ls -d` 或去掉删除参数，看清目标范围再执行。
2. **远程操作别用裸密码**：优先 SSH 密钥；`ssh-copy-id` 添加公钥，确认 `~/.ssh` 目录 700、`authorized_keys` 600。
3. **不要随手 `chmod 777`**：给 web 目录写权限前，先确认属主是否就是服务运行用户；确需宽松权限时用组权限或 ACL（`setfacl -m u:www:rwx dir`）。
4. **改重要配置先备份**：`cp -a nginx.conf nginx.conf.$(date +%F)`，systemd 单元文件改完务必 `systemctl daemon-reload`。
5. **注意 `sudo` 权限边界**：`visudo` 而不是 `vim /etc/sudoers`（前者有语法检查，写错会导致所有人都无法提权）；生产环境不建议长期开放密码登录 `PasswordAuthentication`。
6. **SELinux/AppArmor 不是"关了就干净"**：权限异常先 `getenforce` 判断；确需临时排查可 `setenforce 0`，但应尽快用 `ausearch -m avc` 定位规则再放行，而不是长期关闭。
7. **服务器时间必须同步**：时间漂移会引发证书校验失败、日志对不上、Kerberos/分布式协议报错；用 `timedatectl set-ntp true` 确认同步生效。

---
