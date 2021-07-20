## No data appears on the server

I'm trying to use the [Network plugin](https://collectd.org/wiki/index.php/Plugin:Network) to send data from one instance of *collectd* to another and the [RRDtool plugin](https://collectd.org/wiki/index.php/Plugin:RRDtool) to store the data on the server. It doesn't work, though.

- Are RRD-files created on the server at all?

  - If so, the data was successfully received by the server at least once. You should probably continue with [#Graphs are empty](https://collectd.org/wiki/index.php/Troubleshooting#Graphs_are_empty) below.

- Are the packets actually sent by the client and received by the server?

  `tcpdump -i eth0 -p -n -s 1500 udp port 25826`


  - On the client, you should see outgoing packets with roughly the frequency specified by the *interval* setting. Collectd does not want to waste bandwidth with packet overhead and will buffer multiple measurements until an UDP-packet is close to full so if you have few sensors it might take multiple intervals before a packet is sent.
  - In case you only have a few active sensors and a long interval, restarting collectd after at least one(1) *interval* has passed should cause a flush of any gathered measurements giving you a packet.
  - If you do not see incoming data on the server, this is a problem with your network, not with *collectd*. Check that firewalls allow communication on the appropriate port. See that servers use the correct interface to send data.

- Is the receiving socket opened by the server?

  `netstat -lnp | grep collectd`


  - If not, the configuration of the *Network plugin* is incorrect or something went wrong during start-up.

- If your client sends packets and a receiving socket is open, make sure you listen to the correct protocol

  - IPv4: `<Listen "0.0.0.0" "25826">`
  - IPv6: `<Listen "::" "25826">`





## 服务器上没有数据出现

我正在尝试使用[网络插件](https://collectd.org/wiki/index.php/Plugin:Network)将数据从一个*collectd*实例发送到另一个实例，并使用[RRDtool 插件](https://collectd.org/wiki/index.php/Plugin:RRDtool)将数据存储在服务器上。但是，它不起作用。

- 是否在服务器上创建了 RRD 文件？

  - 如果是这样，则服务器至少成功接收了一次数据。您应该继续使用[#Graphs are empty](https://collectd.org/wiki/index.php/Troubleshooting#Graphs_are_empty)下面。

- 数据包实际上是由客户端发送并由服务器接收吗？


  - 在客户端上，您应该看到输出数据包的频率大致由*间隔*设置指定。Collectd 不想因数据包开销而浪费带宽，并且会缓冲多次测量，直到 UDP 数据包接近满为止，因此如果您的传感器很少，则在发送数据包之前可能需要多个时间间隔。
  - 如果您只有几个活动传感器且时间间隔很长，则在至少一 (1) 个*时间间隔*过后重新启动 collectd应该会导致刷新所有收集的测量值，为您提供一个数据包。
  - 如果您在服务器上看不到传入数据，则这是您的网络问题，而不是*collectd 问题*。检查防火墙是否允许在适当的端口上进行通信。查看服务器使用正确的接口发送数据。

- 接收套接字是否由服务器打开？


  - 如果不是，则*网络插件*的配置不正确或启动过程中出现问题。

- 如果您的客户端发送数据包并且接收套接字已打开，请确保您侦听正确的协议

  

## Plugin `network`

The Network plugin sends data to a remote instance of collectd, receives data from a remote instance, or both at the same time. Data which has been received from the network is usually not transmitted again, but this can be activated, see the **Forward** option below.

The default IPv6 multicast group is `ff18::efc0:4a42`. The default IPv4 multicast group is `239.192.74.66`. The default *UDP* port is **25826**.

Both, **Server** and **Listen** can be used as single option or as block. When used as block, given options are valid for this socket only. The following example will export the metrics twice: Once to an "internal" server (without encryption and signing) and one to an external server (with cryptographic signature):

网络插件将数据发送到RecolleDD的远程实例，从远程实例或同时接收数据。 从网络接收的数据通常不会再次传输，但可以激活此操作，请参阅下面的前进选项。

默认的IPv6组播组是FF18 :: EFC0：4A42。 默认的IPv4组播组是239.192.74.66。 默认的UDP端口为25826。

两个，服务器和侦听都可以用作单个选项或作为块。 当用作块时，给定选项仅对此套接字有效。 以下示例将导出两次指标：一次到“内部”服务器（不加密和签名）和一个到外部服务器（带加密签名）：

```
 <Plugin "network">
   # Export to an internal server
   # (demonstrates usage without additional options)
   Server "collectd.internal.tld"
   # Export to an external server
   # (demonstrates usage with signature options)
   <Server "collectd.external.tld">
     SecurityLevel "sign"
     Username "myhostname"
     Password "ohl0eQue"
   </Server>
 </Plugin>
```

- **ServerHost  [ Port ] **

  The **Server** statement/block sets the server to send datagrams to. The statement may occur multiple times to send each datagram to multiple destinations.The argument *Host* may be a hostname, an IPv4 address or an IPv6 address. The optional second argument specifies a port number or a service name. If not given, the default, **25826**, is used.The following options are recognized within **Server** blocks:

  服务器语句/块设置服务器以发送数据报。 该语句可能多次发生以将每个数据报发送给多个目的地。

  参数主机可以是主机名，IPv4地址或IPv6地址。 可选的第二个参数指定端口号或服务名称。 如果未给出，则使用默认值为25826。

  在服务器块中识别以下选项：

  

  ***\*SecurityLevel\** \**Encrypt\**|\**Sign\**|\**None\****Set the security you require for network communication. When the security level has been set to **Encrypt**, data sent over the network will be encrypted using *AES-256*. The integrity of encrypted packets is ensured using *SHA-1*. When set to **Sign**, transmitted data is signed using the *HMAC-SHA-256* message authentication code. When set to **None**, data is sent without any security.This feature is only available if the *network* plugin was linked with *libgcrypt*.

  设置网络通信所需的安全性。 当安全级别设置为加密时，通过网络发送的数据将使用AES-256加密。 使用SHA-1确保加密数据包的完整性。 设置为签名时，使用HMAC-SHA-256消息认证码签名传输数据。 设置为无后，无需任何安全性就会发送数据。

  仅当网络插件与Libgcrypt链接时，此功能才可用。

  

  ***\*Username\** \*Username\***

  Sets the username to transmit. This is used by the server to lookup the password. See **AuthFile** below. All security levels except **None** require this setting.

  This feature is only available if the *network* plugin was linked with *libgcrypt*.

  设置要传输的用户名。 这是服务器使用来查找密码。 请参阅下面的Authfile。 除非无需此设置之外的所有安全级别。

  仅当网络插件与Libgcrypt链接时，此功能才可用。

  

  ***\*Password\** \*Password\***

  Sets a password (shared secret) for this socket. All security levels except **None** require this setting.This feature is only available if the *network* plugin was linked with *libgcrypt*.

  为此套接字设置密码（共享秘密）。 除非无需此设置之外的所有安全级别。

  仅当网络插件与Libgcrypt链接时，此功能才可用。

  

  ***\*Interface\** \*Interface name\***

  Set the outgoing interface for IP packets. This applies at least to IPv6 packets and if possible to IPv4. If this option is not applicable, undefined or a non-existent interface name is specified, the default behavior is to let the kernel choose the appropriate interface. Be warned that the manual selection of an interface for unicast traffic is only necessary in rare cases.

  设置IP数据包的输出接口。 这至少适用于IPv6数据包，如果可能的话是IPv4。 如果不适用此选项，则指定未定义或不存在的接口名称，默认行为是让内核选择相应的界面。 被警告说，只有在极少数情况下只需要手动选择单播交通的界面。

  

  ***\*ResolveInterval\** \*Seconds\***

  Sets the interval at which to re-resolve the DNS for the *Host*. This is useful to force a regular DNS lookup to support a high availability setup. If not specified, re-resolves are never attempted.

  设置重新解析主机的DNS的间隔。 强制常规DNS查找是有用的，以支持高可用性设置。 如果未指定，则不尝试重新解析。

  

- **<Listen Host  [Port]>**

  The **Listen** statement sets the interfaces to bind to. When multiple statements are found the daemon will bind to multiple interfaces.The argument *Host* may be a hostname, an IPv4 address or an IPv6 address. If the argument is a multicast address the daemon will join that multicast group. The optional second argument specifies a port number or a service name. If not given, the default, **25826**, is used.

  The following options are recognized within `<Listen>`blocks:

  listen语句设置绑定的接口。 找到多个语句时，守护程序将绑定到多个接口。

  参数主机可以是主机名，IPv4地址或IPv6地址。 如果参数是多播地址，守护程序将加入该组播组。 可选的第二个参数指定端口号或服务名称。 如果未给出，则使用默认值为25826。

  以下选项在`<listen>`块中识别：

  

  ***\*SecurityLevel\** \**Encrypt\**|\**Sign\**|\**None\****

  Set the security you require for network communication. When the security level has been set to **Encrypt**, only encrypted data will be accepted. The integrity of encrypted packets is ensured using *SHA-1*. When set to **Sign**, only signed and encrypted data is accepted. When set to **None**, all data will be accepted. If an **AuthFile** option was given (see below), encrypted data is decrypted if possible.

  This feature is only available if the *network* plugin was linked with *libgcrypt*.

  设置网络通信所需的安全性。 当安全级别设置为加密时，只接受加密数据。 使用SHA-1确保加密数据包的完整性。 设置签名时，只接受签名和加密数据。 设置为无时，将接受所有数据。 如果给出了AuthFile选项（见下文），则可以在可能的情况下解密加密数据。

  仅当网络插件与Libgcrypt链接时，此功能才可用。

  

  ***\*AuthFile\** \*Filename\***

  Sets a file in which usernames are mapped to passwords. These passwords are used to verify signatures and to decrypt encrypted network packets. If **SecurityLevel** is set to **None**, this is optional. If given, signed data is verified and encrypted packets are decrypted. Otherwise, signed data is accepted without checking the signature and encrypted data cannot be decrypted. For the other security levels this option is mandatory.The file format is very simple: Each line consists of a username followed by a colon and any number of spaces followed by the password. To demonstrate, an example file could look like this:`  user0: foo  user1: bar`

  Each time a packet is received, the modification time of the file is checked using *stat(2)*. If the file has been changed, the contents is re-read. While the file is being read, it is locked using *fcntl(2)*.

  设置一个文件，其中用户名映射到密码。 这些密码用于验证签名并解密加密的网络数据包。 如果SecurityLevel设置为None，则这是可选的。 如果给定，则验证签名数据并解密加密数据包。 否则，签名数据被接受而不检查签名，加密数据无法解密。 对于其他安全级别，此选项是强制性的。

  文件格式非常简单：每行由用户名组成，后跟冒号和任何数量的空格，然后是密码。 要演示，示例文件可以如下所示：

  

  每次接收到数据包时，都会使用stat（2）检查文件的修改时间。 如果文件已更改，则重新读取内容。 读取文件时，它将使用FCNTL（2）锁定。

  

  ***\*Interface\** \*Interface name\***

  Set the incoming interface for IP packets explicitly. This applies at least to IPv6 packets and if possible to IPv4. If this option is not applicable, undefined or a non-existent interface name is specified, the default behavior is, to let the kernel choose the appropriate interface. Thus incoming traffic gets only accepted, if it arrives on the given interface.

  显式设置IP数据包的传入接口。 这至少适用于IPv6数据包，如果可能的话是IPv4。 如果不适用此选项，则指定未定义或不存在的接口名称，默认行为是，以让内核选择相应的界面。 因此，如果它到达给定的界面，则只接受传入的流量。

  

- ***\*TimeToLive\** \*1-255\***

  Set the time-to-live of sent packets. This applies to all, unicast and multicast, and IPv4 and IPv6 packets. The default is to not change this value. That means that multicast packets will be sent with a TTL of `1` (one) on most operating systems.

  设置已发送数据包的时间。 这适用于所有，单播和多播和IPv4和IPv6数据包。 默认值是不更改此值。 这意味着将在大多数操作系统上以1（一）的TTL发送多播数据包。

- ***\*MaxPacketSize\** \*1024-65535\***

  Set the maximum size for datagrams received over the network. Packets larger than this will be truncated. Defaults to 1452 bytes, which is the maximum payload size that can be transmitted in one Ethernet frame using IPv6 / UDP.On the server side, this limit should be set to the largest value used on *any* client. Likewise, the value on the client must not be larger than the value on the server, or data will be lost.**Compatibility:** Versions prior to *version 4.8* used a fixed sized buffer of 1024 bytes. Versions *4.8*, *4.9* and *4.10* used a default value of 1024 bytes to avoid problems when sending data to an older server.

  设置通过网络接收的数据报的最大大小。 大于此的数据包将被截断。 默认为1452个字节，这是可以使用IPv6 / UDP在一个以太网帧中传输的最大有效载荷大小。

  在服务器端，此限制应设置为任何客户端上使用的最大值。 同样，客户端上的值不得大于服务器上的值，或者数据将丢失。

  兼容性：版本4.8之前的版本使用了1024个字节的固定大小缓冲区。 版本4.8,4.9和4.10使用了1024个字节的默认值，以避免将数据发送到旧服务器时出现问题。

- ***\*Forward\** \*true|false\***

  If set to *true*, write packets that were received via the network plugin to the sending sockets. This should only be activated when the **Listen**- and **Server**-statements differ. Otherwise packets may be send multiple times to the same multicast group. While this results in more network traffic than necessary it's not a huge problem since the plugin has a duplicate detection, so the values will not loop.

  如果设置为true，则通过网络插件收到的写数据包到发送套接字。 只有当侦听和服务器 - 语句的不同时，才应激活。 否则，可以多次向同一组播组发送分组。 虽然这导致更多的网络流量而不是必要的，但由于插件具有复制检测，因此它不是一个巨大的问题，因此值不会循环。

- ***\*ReportStats\** \**true\**|\**false\****

  The network plugin cannot only receive and send statistics, it can also create statistics about itself. Collectd data included the number of received and sent octets and packets, the length of the receive queue and the number of values handled. When set to **true**, the *Network plugin* will make these statistics available. Defaults to **false**.

网络插件不能只接收并发送统计信息，它也可以创建有关自己的统计信息。 CollectD数据包括接收和发送八位字节和数据包的数量，接收队列的长度和处理的值数。 设置为true时，网络插件将使这些统计信息可用。 默认为false。