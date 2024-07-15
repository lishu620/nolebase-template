---
comment: true
tags:
  - 路由与交换
---
# DHCP
## 问题

在路由器中正确配置DHCP服务，使PC能自动获取IP地址

## 拓扑图

![[public/img/luyou-change/Pasted image 20240715170416.png]]
## 路由表


|        | AR1           |
| ------ | ------------- |
| G0/0/0 | 192.168.1.254 |

## 配置命令

### 1.给路由器配置IP地址

```bash
sys
int g0/0/0
ip add 192.168.1.254 24
```
### 2.启动路由器DHCP功能

```bash
dhcp enable
```
### 3.选择使用DHCP服务的接口

```bash
dhcp select interface
```
### 4.验证DHCP功能

![[public/img/luyou-change/Pasted image 20240715171037.png]]