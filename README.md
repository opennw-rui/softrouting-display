# Network-Display
Some scripts that make network display more intuitive on Linux SoftRouting
# 使用案例
## routel
Linux下存在iproute2包可以配置内核的ip，路由，接口等信息，其中，`ip route route show table all`命令可以看到所有路由表内的路由，但显示出的效果非常杂乱。  

默认情况下，存在一个`/usr/bin/routel`程序，使用`routel`命令可以有类似网络路由器那样的路由表显示，但还是存在部分缺陷。  

因此我更改了routel程序内容，增加了颜色分类和Metric显示，使用时直接把我存储库中的routel内容覆盖掉你`/usr/bin/routel`内容即可。  

原本的routel是python程序，在本项目刚创建时也使用python完成，但是发现在万级路由的情况下cpu消耗大和cpu运行时间长；随后使用C++重写程序并使用静态库，以确保在Linux上不受库的影响可直接使用。重写后的程序运行时的CPU消耗仅仅比ip route 命令查看时增加了部分，但是换来的却是美观整洁的界面和丰富的参数效果。

效果如下：  

<img width="1464" height="801" alt="image" src="https://github.com/user-attachments/assets/dc31c9c3-ba0f-4b52-8332-24167202928b" />  

默认情况下，直接使用`routel`命令会显示所有的IPv4路由，默认情况下显200条路由，超出部分可使用 `空格` 下翻，`Ctrl + c` 结束（就如同网络设备一样，为了防止路由过多导致单次查询消耗太多的cpu）  

------------------------------------------------------------------

**可以使用的参数：**  

1、routel -i ：筛选某条路由  

```bash
root@SoftRouting:~# routel -i 192.168.10.0/24
----------------------------------------------------------------------------------------------
|                                     IPv4 Routes: 5534                                      |
----------------------------------------------------------------------------------------------
| Dst                 Gateway       Prefsrc         Protocol  Scope   Metric  Dev      Table |
----------------------------------------------------------------------------------------------
| 192.168.10.0/24     /             192.168.10.254  kernel    link    425     lan      main  |
----------------------------------------------------------------------------------------------
```

2、routel -p 500：指定打印单页的路由数（默认200条），使用-p all显示当前协议下的所有路由  

```bash
root@SoftRouting:~# routel -p 500
root@SoftRouting:~# routel -p all
```

3、routel -4/-6：指定查询IPv4/IPv6路由  

```bash
root@SoftRouting:~# routel -6
-------------------------------------------------------------------------------------------------------------------------------
|                                                       IPv6 Routes: 21                                                       |
-------------------------------------------------------------------------------------------------------------------------------
| Dst                                     Gateway                   Prefsrc  Protocol  Scope   Metric  Dev              Table |
-------------------------------------------------------------------------------------------------------------------------------
| 240e:b8f:xxxx:xxxx::/64                 /                         /        ra        global  114     eth0             main  |
| ::/1                                    /                         /        kernel    global  1024    wg0              main  |
| fd00:10:10:40::/96                      /                         /        kernel    global  256     wg0              main  |
| fe80::/64                               /                         /        kernel    global  1024    eth0             main  |
| fe80::/64                               /                         /        kernel    global  1024    lan              main  |
| 8000::/1                                /                         /        kernel    global  1024    wg0              main  |
| default                                 fe80::11ad:7cc9:26ef:5ce  /        ra        global  114     eth0             main  |
| ::1                                     /                         /        kernel    host    0       lo               local |
| 240e:b8f:xxxx:xxxx::                    /                         /        kernel    host    0       eth0             local |
| 240e:b8f:xxxx:xxxx:xxxx:xxxx:fe02:1360  /                         /        kernel    host    0       eth0             local |
| fd00:10:10:40::                         /                         /        kernel    host    0       wg0              local |
| fd00:10:10:40::100                      /                         /        kernel    host    0       wg0              local |
| fe80::                                  /                         /        kernel    host    0       eth0             local |
| fe80::                                  /                         /        kernel    host    0       lan              local |
| fe80::499a:3536:69c6:d2bf               /                         /        kernel    host    0       lan              local |
| fe80::62be:b4ff:fe02:1360               /                         /        kernel    host    0       eth0             local |
| ff00::/8                                /                         /        kernel    link    256     eth0             local |
| ff00::/8                                /                         /        kernel    link    256     wg0              local |
| ff00::/8                                /                         /        kernel    link    256     eth1             local |
| ff00::/8                                /                         /        kernel    link    256     wlx0013ef6f25bd  local |
| ff00::/8                                /                         /        kernel    link    256     lan              local |
-------------------------------------------------------------------------------------------------------------------------------
```

4、使用 `空格` 翻页，`Ctrl + c` 结束进程  

5、routel -t：指定查询表的名称（默认显示所有表）  

```bash
root@SoftRouting:~# routel -6 -t local
-------------------------------------------------------------------------------------------------------------
|                                              IPv6 Routes: 14                                              |
-------------------------------------------------------------------------------------------------------------
| Dst                                     Gateway  Prefsrc  Protocol  Scope  Metric  Dev              Table |
-------------------------------------------------------------------------------------------------------------
| ::1                                     /        /        kernel    host   0       lo               local |
| 240e:b8f:xxxx:xxxx::                    /        /        kernel    host   0       eth0             local |
| 240e:b8f:xxxx:xxxx:xxxx:xxxx:fe02:1360  /        /        kernel    host   0       eth0             local |
| fd00:10:10:40::                         /        /        kernel    host   0       wg0              local |
| fd00:10:10:40::100                      /        /        kernel    host   0       wg0              local |
| fe80::                                  /        /        kernel    host   0       eth0             local |
| fe80::                                  /        /        kernel    host   0       lan              local |
| fe80::499a:3536:69c6:d2bf               /        /        kernel    host   0       lan              local |
| fe80::62be:b4ff:fe02:1360               /        /        kernel    host   0       eth0             local |
| ff00::/8                                /        /        kernel    link   256     eth0             local |
| ff00::/8                                /        /        kernel    link   256     wg0              local |
| ff00::/8                                /        /        kernel    link   256     eth1             local |
| ff00::/8                                /        /        kernel    link   256     wlx0013ef6f25bd  local |
| ff00::/8                                /        /        kernel    link   256     lan              local |
-------------------------------------------------------------------------------------------------------------
```

## xfromshow
在使用StrongSwan配置了IPSec VPN后，如果要查看xfrom策略，可使用 `ip xfrom policy` 命令，但显示出的内容过于混乱，因此我写了个脚本可以更直观的显示xfrom策略  

使用时，将我存储库中的xfromshow下载至 `/usr/bin/` 目录下，使用 `chmod + x /usr/bin/xfromshow` 命令添加可执行权限，重新登录后即可使用，效果如下 

------

<img width="1477" height="710" alt="image" src="https://github.com/user-attachments/assets/e7667d96-9b52-43b7-b08a-5755d7a73fec" />  

## ovpn-show
将本项目内的`ovpn-show`文件内容写入`/usr/bin/ovpn-show`内，并使用`chmod +x /usr/bin/ovpn-show`赋予可执行权限后即可使用，效果如下  

1、指定openvpn日志文件和status文件  
```bash
ovpn-show -c openvpn.log -s openvpn-status.log
-------------------------------------------------------------------------------------------------------------------
|                                          Total: 4          Online: 1                                            |
-------------------------------------------------------------------------------------------------------------------
| User               | Public IP       | VPN IP       | Up Date    | Up Time  | Down Date  | Down Time | Duration |
-------------------------------------------------------------------------------------------------------------------
| 1111111            | 180.164.xxx.xx  | 10.10.30.100 | 2025-10-27 | 20:27:22 | -          | -         | ONLINE   |
| 22222222222222     | 180.164.xxx.xx  | 10.10.30.51  | 2025-10-27 | 20:13:45 | 2025-10-27 | 20:26:45  | 0:13:00  |
| 333333333          | 223.166.xxx.xx  | 10.10.30.50  | 2025-10-27 | 20:08:39 | 2025-10-27 | 20:15:25  | 0:06:46  |
| 4444444            | 180.164.1xx.xx  | 10.10.30.100 | 2025-10-27 | 18:45:39 | 2025-10-27 | 20:27:09  | 1:41:30  |
| 5555555            | 180.164.1xx.xx  | 10.10.30.100 | 2025-10-27 | 18:45:39 | 2025-10-27 | 20:27:09  | 1:41:30  |
-------------------------------------------------------------------------------------------------------------------
```
2、默认只显示最近50条信息，使用-p可指定显示条数，-p all显示所有信息  

```bash
ovpn-show -c openvpn.log -s openvpn-status.log -p all
ovpn-show -c openvpn.log -s openvpn-status.log -p 200
```

3、如果想查看单独某个账户的信息，可以使用-i参数（默认也是显示最近50条，可以加入-p指定显示条目数或-p all显示全部）  

```bash
ovpn-show -c openvpn.log -s openvpn-status.log -i user
```

注意：  

1、以上脚本强制要求使用-c和-s参数以确保数据准确（因为极端情况下，客户端断开后服务端没有相关下线日志，只靠log文件内容可能无法正确判断在线状态，而status文件可以进行二次确认客户端是否真的在线）；  

2、最多显示的客户端记录为200000条，超出的就会丢弃。防止日志文件过大（但默认-p只显示50条，所以，其实不管日志多大，不使用-p指定的过大就没事）；  

3、在服务器端配置中加入下面的内容后重启，可在指定目录指定时间内记录在线用户

```bash
status log/openvpn-status.log 10			# 每10秒刷新一次在线用户
```

## 实时监控包转发速率和带宽

将脚本packets-show放入/usr/bin/，然后使用chmod + x赋予可执行权限，使用packs-show命令运行，效果如下  

<img width="1397" height="396" alt="image" src="https://github.com/user-attachments/assets/393bd525-396a-4c26-9c30-572b084fdf7f" />  

- **IFACE**：接口名称（如 lo、eth0、lan）。
- **rxpck/s**：每秒接收包数（Receive Packets per second）。
- **txpck/s**：每秒发送包数（Transmit Packets per second）。
- **rxMbps**：每秒接收带宽。
- **txMbps**：每秒发送带宽。
- **rxbcast/s**：每秒接收广播包数。
- **txbcast/s**：每秒发送广播包数。
- **rxmcst/s**：每秒接收组播包数（Receive Multicast packets per second，不包括广播）。
- t**xmcst/s**：每秒发送组播包数（Receive Multicast packets per second，不包括广播）。

使用-i指定要监控的网卡

```bash
root@SoftRouting:~# packs_show -i eth1
----------------------------------------------------------------------------------------------
|Interface   | rxpck/s| txpck/s|  rxMbps|  txMbps| rxbcast/s| txbcast/s| rxmcast/s| txmcast/s|
----------------------------------------------------------------------------------------------
|eth1        |      14|      20|    0.01|    0.02|         0|         0|         0|         0|
----------------------------------------------------------------------------------------------
```
