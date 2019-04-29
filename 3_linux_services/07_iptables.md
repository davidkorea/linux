- 安装主核心包
  - ```yum install -y iptables-services```
- 启动服务，安装后默认不启动
  - ```systemctl start iptables.service```
  - ```systemctl enable iptables.service```
  - ```systemctl status iptables.service```
- 配置文件 /etc/sysconfig/iptables，安装上述包之后才有这个文件，否则只有/etc/sysconfig/iptables-config
  ```
  [root@server162 ~]# cat /etc/sysconfig/iptables
  # sample configuration for iptables service
  # you can edit this manually or use system-config-firewall
  # please do not ask us to add additional ports/services to this default configuration
  *filter
  :INPUT ACCEPT [0:0]
  :FORWARD ACCEPT [0:0]
  :OUTPUT ACCEPT [0:0]
  -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
  -A INPUT -p icmp -j ACCEPT
  -A INPUT -i lo -j ACCEPT
  -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
  -A INPUT -j REJECT --reject-with icmp-host-prohibited
  -A FORWARD -j REJECT --reject-with icmp-host-prohibited
  COMMIT
  ```

```shell
--> PREROUTING  -->  [ROUTE]  -->  FORWARD  -->  POSTROUTING  -->
      mangle            |          mangle             ^  mangle
       nat              |          filter             |  nat
                        |                             |
                        |                             |
                        v                             |
                      INPUT                         OUTPUT
                        |  mangle                     ^  mangle
                        |  filter                     |  nat
                        v ----------->local---------->|  filter
```




- 启用内核路由转发功能：临时生效
  ```
  [root@xuegod63 ~]#echo "1" > /proc/sys/net/ipv4/ip_forward
  ```
- 永久生效：
  ```
  [root@xuegod63 ~]# vim /etc/sysctl.conf
  改：#net.ipv4.ip_forward = 0
  为： net.ipv4.ip_forward = 1
  ```
  改完使配置生效：
  ```
  [root@xuegod63 ~]# sysctl -p
  ```

- 配置之前，把前面的规则去除了，否认会达不到一定的效果
  ```
  iptables -P INPUT ACCEPT
  iptables -F
  ```
- 配置SNAT：
  ```
  [root@xuegod63 ~]#iptables -t nat -A POSTROUTING -s 192.168.2.0/24   -j  SNAT  --to 192.168.1.63
  或：
  [root@xuegod63 ~]#iptables -t nat -A POSTROUTING -s 192.168.2.0/24  -o eth0  -j MASQUERADE 
  ```

