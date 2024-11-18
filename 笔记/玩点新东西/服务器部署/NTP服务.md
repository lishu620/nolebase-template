## Linux的两个时钟

> [!NOTE] 提示
> **硬件时钟RTC (Real Time Clock)**：即BIOS时钟，也就是我们主板中用电池供电的时钟，是将时间写入到BIOS中，系统断电后时间不会丢失，可以在开机时通过主板程序中进行设置
> **系统时钟 (System Clock)** ：顾名思义也就是Linux系统内核时钟、软件时钟，是由Linux内核来提供的，系统时钟是基于内存，如果系统断电时间就会丢失
### 查看硬件时间
```bash
[root@server ~]# hwclock
2024-11-18 09:44:28.247872-05:00
```
### 查看系统时间
```bash
[root@server ~]# date
Mon Nov 18 09:45:45 AM EST 2024
```
### 设置日期时间
```bash
[root@server ~]# timedatectl
	Local time: Mon 2024-11-18 09:53:40 EST   # 本地时间
	Universal time: Mon 2024-11-18 14:53:40 UTC   # 世界时间
	RTC time: Mon 2024-11-18 14:53:41   # 硬件时间
	Time zone: US/Eastern (EST, -0500)   # 时区
	System clock synchronized: yes   # 时间是否已同步
	NTP service: active   # 时间同步服务已启动
	RTC in local TZ: no   # no表示硬件时钟设置为协调世界时（UTC），yes表示硬件时钟设置为本地时间
```

### 查看时间同步服务状态

> [!NOTE] 提示
> 由于默认使用chrony服务同步时间，不再使用ntp服务

```bash
[root@server ~]# systemctl status chronyd
```

![[Pasted image 20241118235559.png]]
若出现该界面表示已经成功安装

若出现报错`Unit chronyd.service could not be found.`则说明未安装chronyd

```bash
apt install -y chrony
```

通过上述bash安装chrony服务，重新验证

```bash
[root@server ~]# systemctl status chronyd
```

## 配置同步服务器
输入`[root@server ~]# vi /etc/chrony/chrony.conf`进入配置文件
### 使用阿里云NTP服务器配置
链接：https://developer.aliyun.com/mirror/NTP?spm=a2c6h.13651102.0.0.59021b11IIu5Fx
删除全部内容，输入
```bash
server ntp.aliyun.com iburst
stratumweight 0
driftfile /var/lib/chrony/drift
rtcsync
makestep 10 3
bindcmdaddress 127.0.0.1
bindcmdaddress ::1
keyfile /etc/chrony.keys
commandkey 1
generatecommandkey
logchange 0.5
logdir /var/log/chrony
```
### 重启服务
```bash
systemctl restart chronyd
```
### 重启服务
```bash
[root@server ~]# chronyc  sources  -v
​
  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address        Stratum Poll Reach LastRx Last sample            
==========================================================================
^* 203.107.6.88             2   6    37     0   +586us[ +635us] +/-   47ms
```
### 查看时间是否同步
```bash
[root@server ~]# timedatectl status
	Local time: Mon 2024-11-18 11:06:23 EST
	Universal time: Mon 2024-11-18 16:06:23 UTC
	RTC time: Mon 2024-11-18 16:06:23
	Time zone: US/Eastern (EST, -0500)
	System clock synchronized: yes
	NTP service: active
	RTC in local TZ: no
```

---

> [!NOTE] 提示
> 文章摘录于：[chrony服务器（linux）](https://blog.csdn.net/ymtianyu/article/details/105973953)


