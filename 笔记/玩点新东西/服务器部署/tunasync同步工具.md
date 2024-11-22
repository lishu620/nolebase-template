
> [!NOTE] 提示
> 作为镜像站的同步后端

### 一、安装go语言
```bash
apt install -y golang git
```
### 二、拷贝tunasync程序
```bash
cd
wget https://github.com/tuna/tunasync/releases/download/v0.7.1/tunasync-linux-amd64-bin.tar.gz
tar zxvf tunasync-linux-amd64-bin.tar.gz -C /usr/local/bin/
mkdir tunasync_demo
```
### 三、配置工作目录
创建并编辑worker.conf文件
```bash
vi tunasync_demo/worker.conf
```
配置：
```vi
[global]
name = "test_worker"
log_dir = "/tmp/tunasync/log/tunasync/{{.Name}}"
mirror_dir = "/tmp/tunasync"
concurrent = 10
interval = 1
[manager]
api_base = "http://localhost:12345"
token = ""
ca_cert = ""
[cgroup]
enable = false
base_path = "/sys/fs/cgroup"
group = "tunasync"
[server]
hostname = "localhost"
listen_addr = "127.0.0.1"
listen_port = 6000
ssl_cert = ""
ssl_key = ""
[[mirrors]]
name = "bazel-apt"
provider = "rsync"
upstream = "rsync://mirrors.tuna.tsinghua.edu.cn/bazel-apt/"
use_ipv6 = false
```
创建并编辑maneger.conf文件
```bash
vi tunasync_demo/manager.conf
```
配置
```vi
debug = false
[server]
addr = "127.0.0.1"
port = 12345
ssl_cert = ""
ssl_key = ""
[files]
db_type = "bolt"
db_file = "/tmp/tunasync/manager.db"
ca_cert = ""
```
### 四、运行开始同步
安装必要组件
```bash
mkdir /tmp/tunasync
apt install -y rsync
```
开始运行同步
```bash
tunasync manager --config tunasync_demo/manager.conf & tunasync worker --config tunasync_demo/worker.conf
```
查看同步状态
```bash
tunasynctl list -p 12345 --all
```