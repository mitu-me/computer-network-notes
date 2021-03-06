## DHCP动态主机配置协议

连接在互联网上的计算机的协议软件需要配置的项目包括：
1. IP地址
2. 子网掩码
3. 默认路由器的IP地址
4. 域名服务器的IP地址

为了省去给计算机配置IP地址的麻烦，应当采用自动协议配置的方法，互联网上使用广泛的是**动态主机配置协议DHCP(Dynamic Host Configuration Protocol)**，它提供了一种机制，称为**即插即用联网（plug and play networking）**。这种机制允许一台计算机加入新的网络和获取IP地址并不需要手工参入，DHCP最新的RFC文档是RFC 2131和RFC2132，目前还是互联网草案标准。

DHCP 对运行客户软件和服务器软件的计算机都适用。当运行客户软件的计算机移至一
个新的网络时，就可使用 DHCP 获取其配置信息而不需要手工干预。DHCP 给运行服务器
软件而位置固定的计算机指派一个永久地址，而当这计算机重新启动时其地址不改变。

DHCP使用**客户服务器**方式。需要IP地址的主机在启动的时候就向DHCP服务器广播发送**发现报文（DHCPDISCOVER）(将目的IP地址设置为全1，即255.255.255.255)**，这时候该主机就成为DHCP客户，发送广播报文是因为现在还不知道DHCP服务器在哪个地方，因此要发现（DISCOVER）DHCP服务器的IP地址。这台主机目前还没有自己的IP地址，因此它把IP数据报的源IP地址全部设置为全0，这样，在本地网络上的所有主机都可以收到这个广播报文，但只有DHCP服务器才会对此做出应答。DHCP服务器先在其数据库中查找该计算机的配置信息，如果找到，则返回找到的信息。如果找不到，则从DHCP服务器的IP地址池（Address Pool）中取出一个地址分配给这个计算机。DHCP服务器的回答报文叫做**提供报文（DHCPOFFER）**，表示“提供”了IP地址等配置信息。

但是我们并不愿意为每一个网络都设置一个DHCP服务器，因为这样会使得DHCP服务器数量太多。因此现在是使每一个网络至少拥有一个**DHCP中继代理（relay agent）(通常是一个路由器)**,如图6-19，它配置了DHCP服务器的IP地址信息。

![image](https://img2020.cnblogs.com/blog/2361214/202109/2361214-20210911182920974-1838943837.png)

当DHCP中继代理收到主机A以**广播**形式发送的报文后，就以**单播**的形式向DHCP服务器转发此报文，并等待其回答，收到DHCP回答的提供报文后，DHCP中继代理再把此提供报文发回给主机A。需要注意的是，实际上，**DHCP报文只是UDP用户数据报的数据**，他还要加上UDP首部、IP数据报的首部，以及以太网MAC帧的首部和尾部，才能在链路上传送。

DHCP服务器分配给客户端的IP地址是临时的，因为DHCP客户只能在一段有限的时间使用这个分配到的IP地址。DHCP协议称这段时间为**租用期(lease period)**,但是并没有规定这个租用期到底有多长，这个数值由DHCP服务器自己决定。

DHCP客户常使用的UDP端口号为68，DHCP服务器使用的UDP端口号为67。DHCP详细工作过程如图6-20：

![image](https://img2020.cnblogs.com/blog/2361214/202109/2361214-20210911182942462-1782628510.png)

1. DHCP服务器被动打开UDP67端口，等待客户端发来的报文。
2. DHCP客户端从UDP68端口发送DHCP**发现报文**。
3. 凡是收到DHCP发现报文的DHCP服务器都发出DHCP提供报文，因此DHCP客户端可能会收到多个DHCP**提供报文**。
4. DHCP客户端从几个DHCP服务器中选择其中的一个，并向所选择的DHCP服务器发送DHCP**请求报文**。
5. 被选择的DHCP服务器发送**确认报文DHCPACK**，从这时候起，DHCP客户就可以使用这个IP地址了，这种状态叫做**已绑定状态**。因为DHCP客户端MAC地址和IP地址已经完成了绑定，并且现在可以使用这个临时的DHCP地址了。DHCP客户现在要根据服务器提供的租用期T设置两个计时器T1和T2，它们的超时时间分别为0.5T和0.875T，当超时时间到了就要请求更新租用期。
6. 租用期过了一半（T1时间到），DHCP客户就发送DHCP请求报文（DHCPREQUEST）要求更新租用期。
7. DHCP服务器若同意，则发回确认报文DHCPACK。DHCP客户端就得到了新的租用期，重新设置计时器。
8. DHCP服务器若不同意，则发回**否认报文DHCPNACK**，这时候DHCP客户端就必须停止使用现在的IP地址，并重新申请IP地址。（回到步骤2）。若DHCP服务器不响应T1时间的请求报文，则在租用期过了T2时间，DHCP就必须向DHCP服务器重新发送DHCP请求报文(DHCPREQUEST)，重复步骤6。
9. DHCP客户可以提前终止服务器提供的租用期，这时候只需要向DHCP服务器发送**释放报文(DHCPRELEASE)**就可以了。

DHCP很适合经常移动的计算机。一般在windows系统中网络设置采用**自动获取IP地址和自动获取DNS地址**就是采用的是DHCP协议。
