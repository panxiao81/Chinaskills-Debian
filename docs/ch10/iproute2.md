iproute2 为现代 Linux 发行版自带的软件包，他代替了传统的 net-tools 软件包，后者已经不再维护。

| Legacy utility                                                           | Replacement command     | Note                                |
| ------------------------------------------------------------------------ | ----------------------- | ----------------------------------- |
| [ifconfig](https://en.wikipedia.org/wiki/Ifconfig "Ifconfig")            | ip addr, ip link, ip -s | Address and link configuration      |
| [route](https://en.wikipedia.org/wiki/Route_(command) "Route (command)") | ip route                | Routing tables                      |
| arp                                                                      | ip neigh                | Neighbors                           |
| iptunnel                                                                 | ip tunnel               | Tunnels                             |
| nameif, ifrename                                                         | ip link set name        | Rename network interfaces           |
| ipmaddr                                                                  | ip maddr                | Multicast                           |
| [netstat](https://en.wikipedia.org/wiki/Netstat "Netstat")               | ip -s, ss, ip route     | Show various networking statistics  |
| brctl                                                                    | bridge                  | Handle bridge addresses and devices |

由于各种原因，不应再使用旧的 net-tools 包，但 netstat 依然通过其他软件包提供。