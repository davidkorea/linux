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

```
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
