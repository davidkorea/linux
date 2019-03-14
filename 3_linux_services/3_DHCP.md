# DHCP
1. DHCP服务器工作原理
2. 使用DHCP为局域网中的机器分配IP地址
3. 使用DHCP为服务器分配固定IP地址
4. ntpdate加计划任务同步服务器时间


配置文件：/etc/dhcp/dhcpd.conf	部分配置解释
```shell
# option definitions common to all supported networks...    ＃定义全局配置，通用于所有支持的网络选项.
option domain-name "example.org";    #为客户端指定所属的域，可填写可不填
option domain-name-servers ns1.example.org, ns2.example.org;  #为客户端指定DNS服务器地址，配置dhcp ip的同时，可以获取dns

default-lease-time number(数字)
  default-lease-time 600;
作用：定义默认IP 租约时间，以秒为单位的租约时间。
50%:续约。(续不上继续用)
87.5%:再次续约。(续不上找别人)
DHCP工作站除了在开机的时候发出 DHCPrequest 请求之外，在租约期限一半的时候也会发出 DHCPrequest ，如果此时得不到 DHCP服务器的确认的话，工作站还可以继续使用该IP；当租约期过了87.5%时，如果客户机仍然无法与当初的DHCP服务器联系上，它将与其它 DHCP服务器通信。如果网络上再没有任何DHCP协议服务器在运行时，该客户机必须停止使用该IP地址，并从发送一个Dhcpdiscover数据包开 始，再一次重复整个过程。要是您想退租，可以随时送出 DHCPRELEASE 命令解约，就算您的租约在前一秒钟才获得的。

max-lease-time 7200; (数字)
作用：定义客户端IP租约时间的最大值，当客户端超过租约时间，却尚未更新IP 时，最长可以使用该IP 的时间；
例：
比如，机器在开机获得IP地址后，然后关机了。这时，当时间过了default-lease-time 600秒后，没有机器向DHCP续约，DHCP会保留7200秒，保留此IP地址不用于分配给其它机器。 当超过7200秒后，将不再保留此IP地址给此机器。
注意:（3）、（4）都是以秒为单位的租约时间，该项参数可以作用在全局配置中，也可以作用在局部配置中。

log-facility local7;   #定义日志类型为  local7

subnet：

声明一般用来指定IP 作用域、定义为客户端分配的IP 地址池等等
声明格式如下：
subnet 网络号 netmask 子网掩码 {
选项或参数
}
