# 安卓运行lxc容器教程-备忘录

关键词 termux lxc android kernel

**本教程需要手机 kernel 支持，以及 root 权限**

首先安装模块 [进入releases](https://github.com/qc-wl/termux-lxc/releases "进入releases") 安装 LxcMagisk-core.zip 模块

## 进入Termux
记得 termux 要安装 **tsu** pkg install tsu 否则没有sudo命令，同时要给termux root权限，不然sudo 也执行不了
### 检测cgroup版本
分别执行以下命令
```bash
mount | grep cgroup2 
mount | grep cgroup1
```
如果第 1 条命令有输出信息，你就是cgroup2
如果第 2 条命令有输出信息，你就是cgroup1
如果你是cgroup1版本，执行
```bash
echo "lxc.init.cmd = /sbin/init systemd.unified_cgroup_hierarchy" >>; /data/lxc/share/lxc/config/common.conf
```
如果你是cgroup2版本，执行
```bash
echo "lxc.init.cmd = /sbin/init systemd.unified_cgroup_hierarchy=0" >> /data/lxc/share/lxc
```
**(可以直接使用MT文件管理器修改/data/lxc/share/lxc/config/common.conf里面的内容)**
### 配置一下lxc的网络
```bash
sed -i 's/lxc\.net\.0\.type = empty/lxc.net.0.type = none/g' /data/lxc/etc/lxc/default.conf
```
**(可以直接使用MT文件管理器修改/data/lxc/share/lxc/config/common.conf里面的内容)**
### 挂载cgroup 每次重启都要执行
```bash
sudo mount -t tmpfs -o mode=755 tmpfs /sys/fs/cgroup && sudo mkdir -p /sys/fs/cgroup/devices && sudo mount -t cgroup -o devices cgroup /sys/fs/cgroup/devices && sudo mkdir -p /sys/fs/cgroup/systemd && sudo mount -t cgroup cgroup -o none,name=systemd /sys/fs/cgroup/systemd
sudo lxc-setup-cgroups
```
### 准备创建lxc
```bash
su
cd /data/lxc
source env.sh
```
#### 下载镜像
```bash
sudo /data/lxc/bin/lxc-create -t download -n 你的容器名字 -- --server mirrors.tuna.tsinghua.edu.cn/lxc-images
例如
sudo /data/lxc/bin/lxc-create -t download -n debian -- --server mirrors.tuna.tsinghua.edu.cn/lxc-images
```
依次输入 debian bookworm arm64
#### 启动
```bash
sudo /data/lxc/bin/lxc-start -n 你容器的名字
例如
sudo /data/lxc/bin/lxc-start -n debian
```
#### 设置密码
```bash
sudo /data/lxc/bin/lxc-attach -n debian /bin/passwd root
```
#### 登录
```bash
sudo /data/lxc/bin/lxc-attach -n debian /bin/login
```
用户名root
### 进入到系统里面的工作
修改dns （ubuntu/debian）
```bash
 echo "nameserver 8.8.8.8" > /etc/resolv.conf
```
这个只能暂时修改dns 永久办法请自行查找
