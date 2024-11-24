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

## 拓展（bind webadmin）
该方式安装的优点可以将相关应用配置在同一台物理主机，但对安装人员有一定技术要求
### 安装Mysql
####  1.更新软件包列表

首先，我们需要确保我们的软件包列表是最新的。这可以通过运行 `apt update` 命令来完成。
```bash
apt update
```
#### 2. 安装MySQL APT配置工具

接下来，我们需要从MySQL官方网站下载APT配置工具，并安装它以便能够通过APT仓库安装MySQL。
```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb
apt install gnupg
dpkg -i mysql-apt-config_0.8.32-1_all.deb
apt update
```
#### 3. 安装MySQL服务器

现在我们可以使用APT安装MySQL服务器。
```bash
apt install mysql-server
```
#### 4 . 更新软件包列表

再次更新软件包列表，以便APT可以识别新添加的MySQL仓库。

```bash
apt update
```
#### 5. 启动MySQL服务

启动MySQL服务并设置它在系统启动时自动启动。

sudo systemctl start mysql

sudo systemctl enable mysql

#### 6. 登录MySQL服务器

使用root用户登录到MySQL服务器。

mysql -u root -p

输入密码后，您将进入MySQL命令行。使用 `exit` 命令退出。

#### 7. 编辑MySQL配置文件

编辑MySQL的配置文件以允许来自任何IP地址的连接。
```bash
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
在文件中找到 `[mysqld]` 部分，并添加或修改 `bind-address` 为 `0.0.0.0`，这将允许MySQL监听所有的网络接口。
```vi
[mysqld]
bind-address = 0.0.0.0
```
### 8. 重启MySQL服务

保存并关闭文件后，重新启动MySQL服务以应用更改。
```bash
systemctl restart mysql
```
### 注意事项

- 在编辑配置文件之前，请确保了解更改的影响，特别是在设置 `bind-address` 时，因为它会影响服务器的安全性。

- 在生产环境中，应避免使用 `0.0.0.0`，而是指定特定的IP地址以限制访问。

- 始终保持系统和软件包的更新，以确保安全性。

- 使用强密码和适当的权限管理来保护MySQL服务器。
### 配置Mysql
将如入mysql初始化值配置导入mysql。
```vi
CREATE DATABASE IF NOT EXISTS `BindDB` DEFAULT CHARACTER SET utf8mb3;
USE `BindDB`;

-- 域名soa记录表。在注册域名时，需核查域名是否已存在，若存在，则不允许注册。
CREATE TABLE IF NOT EXISTS `Zones` (
    `id` int(11) NOT NULL AUTO_INCREMENT,                         /* 对应 ZoneData表中的zone_id */
    `user_name` varchar(128) NOT NULL,                            /* 新增项,对应UserList表中user_name档位。因为用户表中的id是会变化的，不利于后其数据备份和恢复 */
    `domain` varchar(128) NOT NULL DEFAULT '',                    -- 域名
    `host` varchar(128) NOT NULL DEFAULT '',                      -- 主NS，可以是ZoneData表中name档位的值(A记录类型)， 也可以是完整的域名(必须以点(.)为结尾
    `admin` varchar(128) NOT NULL DEFAULT 'info',                 -- 管理员
    `serial` int(11) NOT NULL DEFAULT '2024060100',               -- 格式为“年月日+修改次数”，即“YYMMDDxx”,其中xx为00至99。
    `expire` int(11) NOT NULL DEFAULT '86400',                    -- 记录过期时间，当slave一直未能成功与master取得联络，那么到这里就会放弃更新，同时这的数据也会过期。
    `refresh` int(11) NOT NULL DEFAULT '86400',                   -- 这里是告诉slave隔多久更新一次。
    `retry` int(11) NOT NULL DEFAULT '86400',                     -- slave在进行更新失败后，要隔多久再进行重试。
    `minimum` int(11) NOT NULL DEFAULT '86400',                   -- 预设的最小TTL值
    `ttl` int(11) NOT NULL DEFAULT '86400',                       -- TTL
    `writeable` tinyint(1) NOT NULL DEFAULT '1',                  -- bool类型。0(false),1(true),NULL
    `active` tinyint(1) NOT NULL DEFAULT '1',                     -- 解析是否有效。1可以解析，0不可以解析。
    `registe_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,   -- 注册时间
    `remark` varchar(128) NOT NULL DEFAULT '',                    /* 备注 */
    PRIMARY KEY (`id`),
    KEY `domain_idx` (`domain`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;

insert into `Zones`
    ( `user_name`,`domain`, `host`) VALUES
    ('admin','aidns.io', 'ns1.aidns.io.'),
    ('demo','example.com', 'ns1.aidns.io.'),
    ('demo','test.io',  'ns1.aidns.io.');
    
-- 域名普通记录表
CREATE TABLE IF NOT EXISTS `ZoneData` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `zone_id` int(11) NOT NULL,                                   -- 域名soa记录id,对应Zones表的id档位。
    `domain` varchar(128) NOT NULL,                               -- 新增项,对应Zones表中的domain项。因为Zones表id是变化的，不利于后其数据备份和恢复
    `user_name` varchar(128) NOT NULL,                            /* 新增项,对应UserList表中user_name档位 */
    `name` varchar(128) NOT NULL DEFAULT '',                      -- 记录名称
    `type` varchar(16) NOT NULL DEFAULT '',                       -- 记录的类型
    `data` varchar(256) NOT NULL DEFAULT '',                      -- 解析数据.所以在域名为值时，必须以"."点符号为结尾。
    `ttl` int(11) NOT NULL DEFAULT '120',                         -- TTL. 86400 = 60 * 60 * 24
    `active` tinyint(1) NOT NULL DEFAULT '1',                     -- 解析是否有效。1可以解析，0不可以解析。
    `token` varchar(128) NOT NULL DEFAULT '',                     /* 新增项,用于api方式更新域名解析。token中存储该域名的TTL值 */
    `registe_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,   -- 注册时间
    `remark` varchar(128) NOT NULL DEFAULT '',                    /* 备注,128表示是128个字符 */
    PRIMARY KEY (`id`),
    KEY `zone_idx` (`zone_id`),
    KEY `name_idx` (`zone_id`, `name`),
    KEY `type_idx` (`type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;


insert into `ZoneData`
    (`zone_id`, `domain`, `user_name`,`name`, `type`, `data`) VALUES
    (1,  'aidns.io','admin', '@', 'NS', 'ns1.aidns.io.'),
    (1,  'aidns.io','admin','@', 'NS', 'ns2.aidns.io.'),
    (1,  'aidns.io','admin', '@', 'A', '192.168.3.7'),
    (1,  'aidns.io','admin', 'dns', 'A', '192.168.3.7'),
    (1,  'aidns.io','admin', 'www', 'A', '192.168.3.7'),
    (1,  'aidns.io','admin', 'api', 'A', '192.168.3.7'),
    (1,  'aidns.io','admin','ns1', 'A', '192.168.3.7'),
    (1,  'aidns.io','admin','ns2', 'A', '192.168.3.7'),
    (2,  'example.com','demo', '@', 'NS', 'ns1.aidns.io.'),
    (2,  'example.com','demo','@', 'NS', 'ns2.aidns.io.'),
    (2,  'example.com','demo','@', 'MX', '10 mail.example.com.'),
    (2,  'example.com','demo', '@', 'A', '192.168.0.2'),
    (2,  'example.com','demo','@', 'TXT', '"v=spf1 ip:192.168.0.3 ~all"'),
    (2,  'example.com','demo', 'www', 'CNAME', 'example.com.'),
    (2,  'example.com','demo','mail', 'A', '192.168.0.3'),
    (3,  'test.io','demo', '@', 'NS', 'ns1.aidns.io.'),
    (3,  'test.io','demo','@', 'NS', 'ns2.aidns.io.'),
    (3,  'test.io','demo','@', 'MX', '10 mail.test.io.'),
    (3,  'test.io','demo','@', 'A', '192.168.0.2'),
    (3,  'test.io','demo','@', 'TXT', '"v=spf1 ip:192.168.0.3 ~all"'),
    (3,  'test.io','demo','mail', 'A', '192.168.0.3'),
    (3,  'test.io','demo','www', 'A', '192.168.1.222'),
    (3,  'test.io','demo','oa', 'A', '192.168.1.222');
-- 上表中的如下记录中的IP是bind webadmin服务器的 ns域名 中的A记录值IP
--   (1,  'aidns.io','admin', '@', 'A', '192.168.3.7'),
--   (1,  'aidns.io','admin', 'dns', 'A', '192.168.3.7'),
--   (1,  'aidns.io','admin', 'www', 'A', '192.168.3.7'),
--   (1,  'aidns.io','admin', 'api', 'A', '192.168.3.7'),
--   (1,  'aidns.io','admin','ns1', 'A', '192.168.3.7'),
--   (1,  'aidns.io','admin','ns2', 'A', '192.168.3.7'),
-- 需手工修改，分为如下两种情况
-- 情况一：当配置为公共DNS服务时，改为当前bind webadmin服务器的公网NAT映射IP
-- 情况二: 仅配置为私有DNS时，改为当前bind webadmin服务器的私有ip
-- 务必跟据自己主机真实情况而修改

-- 用户列表
CREATE TABLE IF NOT EXISTS `UserList` (
    `id` int(11) NOT NULL AUTO_INCREMENT,                         /* 用户id*/
    `username` varchar(128) NOT NULL DEFAULT '',                  -- 用户名称
    `password` varchar(128) NOT NULL DEFAULT '',                  -- 密码
    `address` varchar(128) NOT NULL DEFAULT '',                   -- 地址
    `email` varchar(128) NOT NULL DEFAULT '',                     -- email
    `mobile` varchar(128) NOT NULL DEFAULT '',                    -- mobile
    `company` varchar(128) NOT NULL DEFAULT '',                   -- 公司名称
    `website` varchar(128) NOT NULL DEFAULT '',                   -- 公司网站
    `registe_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,   -- 注册时间
    `remark` varchar(128) NOT NULL DEFAULT '',                    -- 备注 
    `active` tinyint(1) NOT NULL DEFAULT '1',                     -- 用户是否有效。1(true),0(false)
    `maxdomain` int(11) NOT NULL DEFAULT '50',                    -- 用户最多可以创建的域名数量
    `maxrecord` int(11) NOT NULL DEFAULT '2500',                  -- 用户最多可以创建的解析数量
    PRIMARY KEY (`id`),
    KEY `id_idx` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;

-- 默认帐号:admin/admin1111mm,demo/demo1111mm
insert UserList (username,password,address,email,mobile,company,website,remark) values 
                ('admin','c4052862747f010e17be96baceaff2f89d6fb4a35cafc7c94c63af2bbe085d5c','shenzhen china','guofs@139.com','13751090806','bind-webadmin','https://www.bind-webadmin.com/','程序员'),
                ('demo','7ea51c8be7251e7bc404864386bf861a4d0b191e6e232567f93cb75c2789762a','shenzhen china','guofs@139.com','13751090806','bind-webadmin','https://www.bind-webadmin.com/','程序员');
```
将上述初始化sql语句写入文本文件`init.sql`，直接导入即可。
```bash
# mysql -uroot -p < init.sql 
# mysql -uroot -p -hlocalhost
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| BindDB             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use BindDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_BindDB |
+------------------+
| UserList         |
| ZoneData         |
| Zones            |
+------------------+
3 rows in set (0.01 sec)

mysql> select * from UserList;
+----+----------+------------------------------------------------------------------+----------------+---------------+-------------+---------------+--------------------------------+---------------------+-----------+--------+-----------+-----------+
| id | username | password                                                         | address        | email         | mobile      | company       | website                        | registe_date        | remark    | active | maxdomain | maxrecord |
+----+----------+------------------------------------------------------------------+----------------+---------------+-------------+---------------+--------------------------------+---------------------+-----------+--------+-----------+-----------+
|  1 | admin    | c4052862747f010e17be96baceaff2f89d6fb4a35cafc7c94c63af2bbe085d5c | shenzhen china | guofs@139.com | 13751090806 | bind-webadmin | https://www.bind-webadmin.com/ | 2024-07-28 10:56:55 | 程序员    |      1 |        50 |      2500 |
|  2 | demo     | 7ea51c8be7251e7bc404864386bf861a4d0b191e6e232567f93cb75c2789762a | shenzhen china | guofs@139.com | 13751090806 | bind-webadmin | https://www.bind-webadmin.com/ | 2024-07-28 10:56:55 | 程序员    |      1 |        50 |      2500 |
+----+----------+------------------------------------------------------------------+----------------+---------------+-------------+---------------+--------------------------------+---------------------+-----------+--------+-----------+-----------+
2 rows in set (0.00 sec)
```
本例中，采用mysql的 `root` 帐号，密码为: `mysql+mm`
### `bind webadmin`后端安装
下载二进制文件

```
# wget https://data.bind-webadmin.com/dw/sr.tar.gz# tar zxvf sr.tar.gz# mv sr /usr/local/# curl https://data.bind-webadmin.com/getlisence/free -o /usr/local/sr/conf/lisence.txt
```

配置文件  
`/usr/local/sr/conf/app.conf`如下

```
#httpaddr = "127.0.0.1"httpport=9090#运行模式：dev、test、prodrunmode=devappname=beego[dev]ver="v0.1"# api_id 自动产生，不需要人工修改。api_id=""mysql_host="localhost"mysql_port=3306mysql_dbname="BindDB"mysql_user="root"mysql_password="mysql+mm"ns="aidns.io"# 数组配置方式# 域名类型domaintypes=".com .net .org .cn .io .top .com.cn .tech .info .in-addr.arpa"# 解析记录，暂不支持ns解析recordtypes="A MX CNAME TXT AAAA SRV PTR"   # TTL(必须有按照如下格式书写)ttl="1 1sec,5 5sec,30 30sec,60 1min,300 5min,1800 30min,3600 1h,43200 12h,86400 24h"#必须是16、24、32位字节的字串。此值必须提供给解密端#此值用于解析api的授权token的加解密。#前端生成apiToken加密,后端存储并解密。#KeyToken="abc123oweruwierw11111111"KeyToken="6s8MQjzqRfMR6ej80a8AuBjn"#页面认证key。此值由后端提供。#必须是16、24、32位字节的字串。此值必须提供给解密端#react解密使用，由后端提供。前后端务必保持一致。KeyAuth="JaWAhEzhzdYdqVUuUhRNK0o0"#后端使用#用户认证时，服务端创建的token时的TTL#1000表示1秒# 1分钟: 1# 1小时: 60# 1天  : 60*24 = 1440TTLAuth=1440#仅前端使用(该值暂时无用)#认端认证成功后存入前端的cookie中。#前端cookie的TTL值#1天:60*60*1000*24TTLCookie=86400000#仅前端使用#认端认证成功后存入前端的cookie中。#前端在产生api Token时为该token配置的TTL#1年=1000*60*60*24*365*1=31536000000TTLApiToken=63072000000#仅供前端使用。#Title，站点标题title="aiDNS"#该参数暂时无用#前端页脚配置Footter_line_sider_1="v0.1"Footter_line_sider_2="aidns.io"Footter_line_1="DNS自主系统"Footter_line_2="by aidns.io"#上面4个参数暂时无用#login页底部文字，可用于配置备案号信息等LoginText="备案号(示例)：深ICP备2020017848号-8"#该参数暂时无用
```

注意

- mysql参数，与需mysql服务中的配置相同
- ns参数，必须是`准备工作中确定的nsdomain值`

启动

```
# cat > /usr/lib/systemd/system/app.service << 'EOF'[Unit]Description=bind webadmin serviceWants=nss-lookup.targetBefore=nss-lookup.targetAfter=network.target network-online.targetAfter=syslog.targetAfter=mysqld.service[Service]Type=forkingExecStart=/bin/sh -c 'cd /usr/local/sr;nohup ./app & 2>/dev/null'[Install]WantedBy=multi-user.targetEOF# systemctl enable app.service# systemctl start app.service
```

测试 在任一台主机上采用如下命令测试，若能返回当前主机ip则说明正常。

```
C:\>curl http://192.168.3.7:9090/ip192.168.3.110
```
### `bind webadmin`前端安装[​](https://bind-webadmin.com/docs/docs/sider/guides/step3#bind-webadmin%E5%89%8D%E7%AB%AF%E5%AE%89%E8%A3%85 "bind-webadmin前端安装的直接链接")

此方式时需node.js(version >=12.22)支持  
安装node.js/npm/yarn

```
wget https://nodejs.org/dist/v20.16.0/node-v20.16.0-linux-x64.tar.xztar -xvf node-v20.16.0-linux-x64.tar.xzmv node-v20.16.0-linux-x64 /usr/local/nodeecho 'export PATH=/usr/local/node/bin:$PATH' >> /etc/profilesource /etc/profilenpm install -g yarn
```

安装bindwebadmin前端
```
git clone ssh://git@ssh.github.com:22/guofusheng007/bindwebadmin.git
cd bindwebadmin
yarn install   # 此过程花费时间较长
```
或
```
git clone https://github.com/guofusheng007/bindwebadmin.git
cd bindwebadmin
yarn install   # 此过程花费时间较长
```
配置`config/config.ts`

```
import { defineConfig } from "umi";import {UmiRouter,RouterListExtr} from "./router";export default defineConfig({  //devServer: false,  //路由配置  routes :[    ...UmiRouter,    ...RouterListExtr,  ],  define: {     // 在项目中，可以通过process.env.NODE_ENV 或者    // process.env.UMI_ENV 或者process.env.date得到对应环境的值    'process.env': {           //--------------------------------------------------------------------------------------------------      //源码发行时的值      REACT_APP_API_URL:'http://api.aidns.io/',   //必须是 / 为结束.后端接口,必须配置为域名方式。此域名必须可解析。      ICP:'粤ICP备2024267436号-1',      OTHERCODE:'粤公网安备44030002003924号',      LOGIN_FOOTER_TEXT:'Mobile: 13751090806    email: info@mm-dns.com',      APP_TITLE:'mmDNS',        //sider页脚      Footter_line_sider_1:'aiDNS',      Footter_line_sider_2:'aidns.io',      //普通页面页脚      Footter_line_1:'DNS自主系统',      Footter_line_2:'by aidns.io',      //前端在产生api Token时为该token配置的TTL      //#1年=1000*60*60*24*365*1=31536000000      TTLApiToken: 315360000000,      //前端所有cookie的TTL值      //1天:60*60*1000*24      TTLCookie: 86400000,          },  },  npmClient: 'yarn',});
```

build

```
yarn buildmv dist /opt/front
```

此时产生build目录，将该目录的内容采用nginx发布。

## nginx配置[​](https://bind-webadmin.com/docs/docs/sider/guides/step3#nginx%E9%85%8D%E7%BD%AE "nginx配置的直接链接")

代理前端页面和后端api接口。

提示

- 本例配置未采用https方式，用户可自行添加证书配置。
- 若采用https方式，则在前端代码`config/config.ts`中的`REACT_APP_API_URL`配置为https方式，并重新build

安装nginx

```
# cat > /etc/yum.repos.d/nginx.repo << 'EOF'[nginx-stable]name=nginx stable repobaseurl=http://nginx.org/packages/centos/$releasever/$basearch/gpgcheck=1enabled=0gpgkey=https://nginx.org/keys/nginx_signing.keymodule_hotfixes=true[nginx-mainline]name=nginx mainline repobaseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/gpgcheck=1enabled=1gpgkey=https://nginx.org/keys/nginx_signing.keymodule_hotfixes=trueEOF# yum makecache# yum -y install nginx nginx-module-*# nginx -vnginx version: nginx/1.27.0# systemctl enable nginx.service
```

配置

```
# cat > /etc/nginx/conf.d/dns.conf << 'EOF'server {  listen 80;  server_name aidns.io www.aidns.io dns.aidns.io;  proxy_set_header Host $http_host;  proxy_set_header X-Real-IP $remote_addr;  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  proxy_set_header X-Forwarded-Proto $scheme;  location / {     root  /opt/front;     try_files $uri $uri/ /index.html;  }  location /ip {    proxy_pass http://127.0.0.1:9090/ip;  }  location /json {    proxy_pass http://127.0.0.1:9090/json;  }  location /updatejson {    proxy_pass http://127.0.0.1:9090/updatejson;  }  location /updateheader {    proxy_pass http://127.0.0.1:9090/updateheader;  }}#------------------------------server {  listen 80;  server_name  api.aidns.io;  proxy_set_header Host $http_host;  proxy_set_header X-Real-IP $remote_addr;  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  proxy_set_header X-Forwarded-Proto $scheme;  location / {    proxy_pass http://127.0.0.1:9090;  }}EOF
```

## 测试[​](https://bind-webadmin.com/docs/docs/sider/guides/step3#%E6%B5%8B%E8%AF%95-1 "测试的直接链接")

在测试机本地网络中的DNS修改为当前安装bind webadmin服务器主机的IP.![Alt text](https://bind-webadmin.com/assets/images/1728125875504-d4bdde16933d9cec93f8f028c4db6baa.png)

采用chrome打开[http://aidns.io](http://aidns.io/)

默认帐号

- admin/admin1111mm
- demo/demo1111mm
https://bind-webadmin.com/docs/docs/sider/guides/step3#bind-webadmin%E5%89%8D%E7%AB%AF%E5%AE%89%E8%A3%85