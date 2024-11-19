## 前言
局域网内有很多台主机，IP难以记忆。
而修改hosts文件又难以做到配置共享和统一，需要一台内网的DNS服务器。

## 正文
### 一、安装
```bash
apt install -y bind9
```
### 二、配置文件说明

配置文件位于`/etc/bind`
有两类文件，一种是
- `name.conf`
- `name.conf.default-zones`
- `name.conf.options`
- `name.conf.local`
另一种是
- `db.127`
- `db.255`
- …
#### 1.named.conf
这个配置文件是最主要的，可以看到它里面其实是引入了其他文件，把实现都转到其他地方了
```vi
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian for information on the
// structure of BIND configuration files in Debian, *BEFORE* you customize
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
named.conf是最基本的，引入的内容都可以直接写在这个文件里面。
#### 2.named.conf.options
这个配置文件可以配置上游服务器，用于同步DNS基础信息
```vi
options {
        directory "/var/cache/bind";

        forwarders {
                223.5.5.5;
                61.128.128.68;
                114.114.114.114;
                8.8.8.8;
                61.128.192.68;
                180.76.76.76;
                119.29.29.29;
                182.254.116.116;
                1.1.1.1;
                45.90.28.249;
                1.2.4.8;
                210.2.4.8;
                4.2.2.1;
                101.226.4.6;
                123.125.81.6;
                202.38.93.153;
        };

        dnssec-validation auto;
        listen-on-v6 { any; };
        allow-query { any; };
};
```
#### 3.db.XXX
里面定义了实际的域名和IP映射关系。
### 三、实例
举个例子，这里为局域网配置一个顶级域名**cel**，以及一些子域名
#### 1. 添加cel顶级域名
```bash
vi /etc/bind/named.conf
```
就像下面这样
```vi
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian for information on the
// structure of BIND configuration files in Debian, *BEFORE* you customize
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";

zone "cel" {
        type master;
        file "/etc/bind/db.cel";
};
```
#### 2. 配置映射文件db.cel
```bash
vi /etc/bind/db.cel
```
配置如下
```vi
$TTL    1h

@       IN      SOA     cel. mlishu.outlook.com. (
                                2               ;Serial
                                3h              ; Refresh after 3 hours
                                1h              ; Retry after 1hour
                                100000  ; Expire after 1 week
                                1h              ; Negative caching TTL of 1 hour
                                )
@       IN      NS      localhost.

e9000.console.cel.      IN      A        192.168.31.20
```
（注意，域名结尾还有一个点，漏掉解析失败的，日志也会提醒）

这里的h后缀表示小时，也可以不要h后缀，单位是秒。

最需要关注的是TTL，可以改成0。

刷新时间（Refresh）：这个字段表示其他 DNS 服务器应该多久来重新获取该区域的 SOA 记录。单位是秒。
重试时间（Retry）：这个字段表示其他 DNS 服务器在刷新失败后应该等待多久来重试。单位是秒。
过期时间（Expire）：这个字段表示在这个时间之后，其他 DNS 服务器应该停止回答关于该区域的查询。单位是秒。
最小 TTL（Minimum TTL）：这个字段表示其他 DNS 服务器在缓存该区域的记录时应该使用的最小 TTL 值。单位是秒。
#### 3. 关闭验证
```bash
vi /etc/bind/named.conf.options
```
将dnssec-validation设为no，否则会用不了（因为要验证签名，感觉很麻烦，局域网自己用一下得了不会有问题的）。
内容如下：
```vi
options {
        directory "/var/cache/bind";

        forwarders {
                223.5.5.5;
                61.128.128.68;
                114.114.114.114;
                8.8.8.8;
                61.128.192.68;
                180.76.76.76;
                119.29.29.29;
                182.254.116.116;
                1.1.1.1;
                45.90.28.249;
                1.2.4.8;
                210.2.4.8;
                4.2.2.1;
                101.226.4.6;
                123.125.81.6;
                202.38.93.153;
        };

        dnssec-validation no;
        listen-on-v6 { any; };
        allow-query { any; };
};
```
#### 4. 修改DNS
先启动我们的DNS服务器:
```bash
systemctl start named
```
启动了最好看一下状态，如果有红色报错或黄色警告，说明有问题，用不了。
```bash
systemctl status named
```
可以以本机为例验证，或者在用其他主机测试：
```bash
vi /etc/resolv.conf
```
修改DNS服务器：
nameserver XXX.XXX.XXX.XXX
这里XXX是你的DNS所在的主机的IP，本机测试填127.0.0.1就可以。
最后，记得防火墙开放53号端口（TCP和UDP都要）。