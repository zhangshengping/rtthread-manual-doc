# lwIP - 轻型TCP/IP协议栈 #

## IPv4协议栈概述 ##

互联网协议版本4（Internet Protocol version 4，IPv4）是互联网协议开发过程中的第四个修订版本(网际网协四版)，也是此协议第一个被广泛部署的版本。IPv4与IPv6均是标准化互联网络的核心部分。IPv4依然是使用最广泛的互联网协议版本，直到2011年，IPv6仍处在部署的初期。

IPv4在IETF于1981年9月发布的[RFC791](http://tools.ietf.org/html/rfc791)中被描述，此RFC替换了于1980年1月发布的RFC 760。

IPv4是一种无连接的协议，操作在使用分组交换的链路层（如以太网）上。此协议会尽最大努力交付分组，意即它不保证任何分组均能送达目的地，也不保证所有分组均按照正确的顺序无重复地到达。这些方面是由上层的传输协议（如传输控制协议）处理的。

IPv4使用32位（4字节）地址（如IPv4报文格式中的源地址（Source Address）和目的地址（Destination Address)），因此地址空间中只有4,294,967,296（232）个地址。IP地址通常被写作点分十进制的形式，即四个字节被分开用十进制写出，中间用点分隔，例如：192.168.0.1

互联网上的主机通常被其名字（如www.rt-thread.org等）而不是IP地址识别，但IP报文的路由是由IP地址而不是这些名字决定的。这就需要将名字翻译（解析）成地址。域名系统（DNS协议）提供了这样一个将名字转换为地址和将地址转换为名字的系统。与CIDR相像，DNS也有一个层级的结构，使不同的名字空间可被再委托给其它DNS服务器。域名系统经常被描述为电话系统中的黄页：在那里人们可以把名字和电话号码对应起来。

IPv4报文的头部格式如下所示，其中包含14个字段，其中13个是必须的，第14个是可选的（Options）。首部中的字段均以大端序包装。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |Version|  IHL  |Type of Service|          Total Length         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |         Identification        |Flags|      Fragment Offset    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Time to Live |    Protocol   |         Header Checksum       |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                       Source Address                          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Destination Address                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Options                    |    Padding    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

版本 -- IP报文首部的第一个字段是4位版本字段。对IPv4来说，这个字段的值是4。

首部长度（IHL）-- 第二个字段是4位首部长度，说明首部有多少32位字长。由于IPv4首部可能包含数目不定的选项，这个字段也用来确定数据的偏移量。这个字段的最小值是5（RFC 791），最大值是15。

DiffServ（DSCP） -- 最初被定义为服务类型字段，但被RFC 2474重定义为DiffServ。新的需要实时数据流的技术会应用这个字段，一个例子是VoIP。

显式拥塞通告（ECN） -- 在RFC 3168中定义，允许在不丢弃报文的同时通知对方网络拥塞的发生。ECN是一种可选的功能，仅当两端都支持并希望使用，且底层网络支持时才被使用。

总长 -- 这个16位字段定义了报文总长，包含首部和数据，单位为字节。这个字段的最小值是20（20字节首部+0字节数据），最大值是65,535。所有主机都必须支持最小576字节的报文，但大多数现代主机支持更大的报文。有时候子网会限制报文的大小，这时报文就必须被分片。

标识符 -- 这个字段主要被用来唯一地标识一个报文的所有分片。一些实验性的工作建议将此字段用于其它目的，例如增加报文跟踪信息以协助探测伪造的源地址。


标志 -- 这个3位字段用于控制和识别分片，它们是：

* 位0：保留，必须为0；
* 位1：禁止分片（DF）；
* 位2：更多分片（MF）。

如果DF标志被设置但路由要求必须分片报文，此报文会被丢弃。这个标志可被用于发往没有能力组装分片的主机。当一个报文被分片，除了最后一片外的所有分片都设置MF标志。不被分片的报文不设置MF标志：它是它自己的最后一片。

分片偏移 -- 这个13位字段指明了每个分片相对于原始报文开头的偏移量，以8字节作单位。

存活时间（TTL） -- 这个8位字段避免报文在互联网中永远存在（例如陷入路由环路）。存活时间以秒为单位，但小于一秒的时间均向上取整到一秒。在现实中，这实际上成了一个跳数计数器：报文经过的每个路由器都将此字段减一，当此字段等于0时，报文不再向下一跳传送并被丢弃。常规地，一份ICMP报文被发回报文发送端说明其发送的报文已被丢弃。这也是traceroute的核心原理。

协议 -- 这个字段定义了该报文数据区使用的协议。IANA维护着一份协议列表（最初由RFC 790定义）。

首部检验和 -- 这个16位检验和字段用于对首部查错。在每一跳，计算出的首部检验和必须与此字段进行比对，如果不一致，此报文被丢弃。值得注意的是，数据区的错误留待上层协议处理——用户数据报协议和传输控制协议都有检验和字段。因为生存时间字段在每一跳都会发生变化，意味着检验和必须被重新计算，RFC 1071这样定义计算检验和的方法：

    The checksum field is the 16-bit one's complement of the one's complement sum of all 16-bit words in the header. For purposes of computing the checksum, the value of the checksum field is zero.

源地址 -- 一个IPv4地址由四个字节共32位构成，此字段的值是将每个字节转为二进制并拼在一起所得到的32位值。这个地址是报文的发送端。但请注意，因为NAT的存在，这个地址并不总是报文的真实发送端，因此发往此地址的报文会被送往NAT设备，并由它被翻译为真实的地址。

目的地址 -- 与源地址格式相同，但指出报文的接收端。

选项 -- 附加的首部字段可能跟在目的地址之后，但这并不被经常使用。请注意首部长度字段必须包括足够的32位字来放下所有的选项（包括任何必须的填充以使首部长度能够被32位整除）。当选项列表的结尾不是首部的结尾时，EOL（选项列表结束，0x00）选项被插入列表末尾。

在IP头之后的是传输的数据，数据字段不是首部的一部分，因此并不被包含在检验和中。数据的格式在协议首部字段中被指明，并可以是任意的传输层协议。一些常见协议的协议字段值如下：

-------------------------------------
协议字段值  协议名              缩写
----------  ------------------- -----
1           互联网控制消息协议  ICMP

2           互联网组管理协议    IGMP

6           传输控制协议        TCP

17          用户数据报协议      UDP

41          IPv6封装	        -

89          开放式最短路径优先  OSPF

132         流控制传输协议      SCTP
-------------------------------------

## lwIP协议栈移植概况##

## RT-Thread以太网卡驱动模型 ##

## UDP编程指南 ##

### UDP原理 ###

### UDP编程示例 ###

## TCP编程指南 ##

### TCP原理，TCP的状态 ###

由[RFC793](http://tools.ietf.org/html/rfc793)所规定

TCPv4报文格式

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |          Source Port          |       Destination Port        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                        Sequence Number                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Acknowledgment Number                      |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Data |           |U|A|P|R|S|F|                               |
    | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
    |       |           |G|K|H|T|N|N|                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |           Checksum            |         Urgent Pointer        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Options                    |    Padding    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                             data                              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


### TCP编程示例 ###

## lwIP直接网络连接 ##

## lwIP如何定位问题 ##

## 扩展:: IPv6 ##

## 简介 ##
[lwIP（light-weight IP)](http://savannah.nongnu.org/projects/lwip/)最初由瑞士计算机科学院（Swedish Institute of Computer Science）的Adam Dunkels开发，现在由Kieran Mansley领导的一个全球开发团队开发、维护的一套用于嵌入式系统的开放源代码TCP/IP协议栈，它在包含完整的TCP协议的基础上实现了小型化的资源占用，因此它十分适合于应用到嵌入式设备中，其占用的资源体积RAM大概为几十kB，ROM大概为40KB。

lwIP结构精简，功能完善，因而用户群较为广泛。RT-Thread实时操作系统就采用了lwIP做为默认的TCP/IP协议栈，同时根据小型设备的特点对lwIP进行了再次优化，使其资源占用体积进一步地缩小，RAM 的占用可缩小到5kB附近（未计算上层应用使用TCP/IP协议时的空间占用量）。本章内容将为您讲述lwIP在RT-Thread中的使用方法。

## lwIP在RT-Thread中的结构 ##

由于原版的lwIP更适合于在无操作系统的情况下运行，所以RT-Thread在移植lwIP的过程中根据RT-Thread的特点进行了适当调整。其结构如下图所示：

RT-Thread操作系统中的lwIP是从lwIP发布原始版本移植过来，然后添加了设备层以替换原来的驱动层。不同于原版，这里RT-Thread对于以太网数据的收发采用了独立的双线程（erx线程与etx线程）结构：

 * erx线程用于以太网报文的接收──当以太网硬件设备收到网络报文产生中断时，中断服务例程将会通过邮箱的形式唤醒erx线程，让erx线程主动进行以太网报文收取过程，当erx线程收到有效网络报文后，它通过邮箱的形式通知给LwIP的主线程（tcp线程）；
 * tcp的发送操作则是通过邮箱的形式唤醒etx线程进行实际的以太网硬件写入。在正常情况下，erx线程和etx线程的优先级是相同的，用户可以根据自身实际要求进行微调以侧重接收或发送。

## 协议初始化 ##

在使用LwIP 协议栈之前，需要初始化协议栈。协议栈本身会启动一个TCP的线程，与协议相关的处理都会放在这个线程中完成。

~~~{.c}
#include <rtthread.h>

#ifdef RT_USING_LWIP
/* 包含LwIP相关的头文件 */
#include <lwip/sys.h>
#include <netif/ethernetif.h>
#endif

/* 初始化线程入口*/
void rt_init_thread_entry(void *parameter)
{
  /* LwIP 初始化 */
#ifdef RT_USING_LWIP
	{
		extern void lwip_sys_init(void);

		/* 初始化以太网线程 */
		eth_system_device_init();

		/* 注册以太网接口驱动 */
		rt_hw_stm32_eth_init();

		/* 初始化LwIP系统 */
		lwip_sys_init();
		rt_kprintf("TCP/IP initialized!\n");
	}
#endif
}

int rt_application_init()
{
	rt_thread_t tid;

	/* 以动态线程方式，创建初始化线程 */
	tid = rt_thread_create("init",
		rt_init_thread_entry, RT_NULL,
		2048, 10, 5);

	/* 启动线程*/
	if (tid != RT_NULL)
		rt_thread_startup(tid);

	return 0;
}
~~~

另外，在RT-Thread操作系统中为了使用lwIP协议栈需要在rtconfig.h头文件中定义使用lwIP的宏：

~~~{.c}
/* 定义RT_USING_LWIP以支持lightweight TCP/IP 协议栈 */
#define RT_USING_LWIP
~~~

在lwIP协议栈中主线程TCP的参数（优先级，信箱大小，栈空间大小）也可以在rtconfig.h头文件中定义：

~~~{.c}
/* TCP线程选项 */
#define RT_LWIP_TCPTHREAD_PRIORITY 	20		/* TCP线程的优先级 */
#define RT_LWIP TCPTHREAD_MBOX_SIZE 	10		/* TCP中使用到的邮箱大小 */
#define RT_LWIP_TCPTHREAD_STACKSIZE 	1024		/* TCP线程的栈大小 */
~~~

默认的IP地址，网关地址，子网掩码也可以在rtconfig.h头文件中定义（如果要使用DHCP方式分配，则需要定义RT_USING_DHCP宏）：

~~~{.c}
/* 目标板IP地址设置为192.168.1.30 */
#define RT_LWIP_IPADDR0     	192
#define RT_LWIP_IPADDR1     	168
#define RT_LWIP_IPADDR2     	1
#define RT_LWIP_IPADDR3     	30

/* 网关地址设置为192.168.1.1 */
#define RT_LWIP_GWADDR0   	192
#define RT_LWIP_GWADDR1   	168
#define RT_LWIP_GWADDR2   	1
#define RT_LWIP_GWADDR3   	1

/* 子网掩码设置为255.255.255.0 */
#define RT_LWIP_MSKADDR0  	255
#define RT_LWIP_MSKADDR1  	255
#define RT_LWIP_MSKADDR2  	255
#define RT_LWIP_MSKADDR3  	0
~~~

## 网络编程示例 ##
### UDP使用示例 ###

下面是一个在RT-Thread上使用BSD socket接口的UDP服务端例子，当把这个代码加入到RT-Thread操作系统时，它会自动向finsh命令行添加一个udpserv命令，在finsh上执行udpserv()函数即可启动这个UDP服务端，该UDP服务端在端口5000上进行监听。

当服务端接收到数据时，它将把数据打印到控制终端中；如果服务端接收到exit字符串时，那么服务端将退出服务。

~~~{.c}
/*
 * 代码清单：UDP服务端例子
 */
#include <rtthread.h>
#include <lwip/sockets.h> /* 使用BSD socket，需要包含sockets.h头文件 */

void udpserv(void* paramemter)
{
  int sock;
	int bytes_read;
	char *recv_data;
	rt_uint32_t addr_len;
	struct sockaddr_in server_addr, client_addr;

	/* 分配接收用的数据缓冲 */
	recv_data = rt_malloc(1024);
	if (recv_data == RT_NULL)
	{
		/* 分配内存失败，返回 */
		rt_kprintf("No memory\n");
		return;
	}

	/* 创建一个socket，类型是SOCK_DGRAM，UDP类型 */
	if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
	{
		rt_kprintf("Socket error\n");

		/* 释放接收用的数据缓冲 */
		rt_free(recv_data);
		return;
	}

	/* 初始化服务端地址 */
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(5000);
	server_addr.sin_addr.s_addr = INADDR_ANY;
	rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

	/* 绑定socket到服务端地址 */
	if (bind(sock, (struct sockaddr *) &server_addr, sizeof(struct sockaddr))
			== -1)
	{
		/* 绑定地址失败 */
		rt_kprintf("Bind error\n");

		/* 释放接收用的数据缓冲 */
		rt_free(recv_data);
		return;
	}

	addr_len = sizeof(struct sockaddr);
	rt_kprintf("UDPServer Waiting for client on port 5000...\n");

	while (1)
	{
		/* 从sock中收取最大1024字节数据 */
		bytes_read = recvfrom(sock, recv_data, 1024, 0,
				(struct sockaddr *) &client_addr, &addr_len);
		/* UDP不同于TCP，它基本不会出现收取的数据失败的情况，除非设置了超时等待 */

		recv_data[bytes_read] = '\0'; /* 把末端清零 */

		/* 输出接收的数据 */
		rt_kprintf("\n(%s , %d) said : ", inet_ntoa(client_addr.sin_addr),
				ntohs(client_addr.sin_port));
		rt_kprintf("%s", recv_data);

		/* 如果接收数据是exit，退出 */
		if (strcmp(recv_data, "exit") == 0)
		{
			lwip_close(sock);

			/* 释放接收用的数据缓冲 */
			rt_free(recv_data);
			break;
		}
	}

	return;
}

#ifdef RT_USING_FINSH
#include <finsh.h>
/* 输出udpserv函数到finsh shell中 */
FINSH_FUNCTION_EXPORT(udpserv, startup udp server);
#endif
~~~

下面是另一个在RT-Thread上使用BSD socket接口的UDP客户端例子。当把这个代码加入到RT-Thread时，它会自动向finsh命令行添加一个udpclient命令，在finsh上执行udpclient(url,port)函数即可启动这个UDP客户端，url指定了这个客户端连接到的服务端地址或域名，port是相应的端口号。

当UDP客户端启动后，它将连续发送5次“This is UDP Client from RT-Thread.”的字符串给服务端，然后退出。

~~~{.c}
/*
 * 程序清单：UDP客户端例子
 */
#include <rtthread.h>
#include <lwip/netdb.h> /* 为了解析主机名，需要包含netdb.h头文件 */
#include <lwip/sockets.h> /* 使用BSD socket，需要包含sockets.h头文件 */

/* 发送用到的数据 */
ALIGN(4)
const char send_data[] = "This is UDP Client from RT-Thread.\n";
void udpclient(const char* url, int port, int count)
{
	int sock;
	struct hostent *host;
	struct sockaddr_in server_addr;

	/* 通过函数入口参数url获得host地址（如果是域名，会做域名解析） */
	host = (struct hostent *) gethostbyname(url);

	/* 创建一个socket，类型是SOCK_DGRAM，UDP类型 */
	if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
	{
		rt_kprintf("Socket error\n");
		return;
	}

	/* 初始化预连接的服务端地址 */
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(port);
	server_addr.sin_addr = *((struct in_addr *) host->h_addr);
	rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

	/* 总计发送count次数据 */
	while (count)
	{
		/* 发送数据到服务远端 */
		sendto(sock, send_data, strlen(send_data), 0,
				(struct sockaddr *) &server_addr, sizeof(struct sockaddr));

		/* 线程休眠一段时间 */
		rt_thread_delay(50);

		/* 计数值减一 */
		count--;
	}

	/* 关闭这个socket */
	lwip_close(sock);
}

#ifdef RT_USING_FINSH
#include <finsh.h>
/* 输出udpclient函数到finsh shell中 */
FINSH_FUNCTION_EXPORT(udpclient, startup udp client);
#endif
~~~

### TCP使用示例 ###

下面是一个在RT-Thread上使用BSD socket接口的TCP服务端例子。当把这个代码加入到RT-Thread时，它会自动向finsh命令行添加一个tcpserv命令，在finsh上执行tcpserv()函数即可启动这个TCP服务端，该TCP服务端在端口5000上进行监听。
当有TCP客户向这个服务端进行连接后，只要服务端接收到数据，它就会立即向客户端发送“This is TCP Server from RT-Thread.”的字符串。

如果服务端接收到q或Q字符串时，服务器将主动关闭这个TCP连接。如果服务端接收到exit字符串时，那么将退出服务。

~~~{.c}
/*
 * 程序清单：TCP服务端例子
 */
#include <rtthread.h>
#include <lwip/sockets.h> /* 使用BSD Socket接口必须包含sockets.h这个头文件 */

/* 发送用到的数据 */
ALIGN(4)
static const char send_data[] = "This is TCP Server from RT-Thread.";
void tcpserv(void* parameter)
{
	char *recv_data; /* 用于接收的指针，后面会做一次动态分配以请求可用内存 */
	rt_uint32_t sin_size;
	int sock, connected, bytes_received;
	struct sockaddr_in server_addr, client_addr;
	rt_bool_t stop = RT_FALSE; /* 停止标志 */

	recv_data = rt_malloc(1024); /* 分配接收用的数据缓冲 */
	if (recv_data == RT_NULL)
	{
		rt_kprintf("No memory\n");
		return;
	}

	/* 一个socket在使用前，需要预先创建出来，指定SOCK_STREAM为TCP的socket */
	if ((sock = socket(AF_INET, SOCK_STREAM, 0)) == -1)
	{
		/* 创建失败的错误处理 */
		rt_kprintf("Socket error\n");

		/* 释放已分配的接收缓冲 */
		rt_free(recv_data);
		return;
	}

	/* 初始化服务端地址 */
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(5000); /* 服务端工作的端口 */
	server_addr.sin_addr.s_addr = INADDR_ANY;
	rt_memset(&(server_addr.sin_zero), 8, sizeof(server_addr.sin_zero));

	/* 绑定socket到服务端地址 */
	if (bind(sock, (struct sockaddr *) &server_addr, sizeof(struct sockaddr))
			== -1)
	{
		/* 绑定失败 */
		rt_kprintf("Unable to bind\n");

		/* 释放已分配的接收缓冲 */
		rt_free(recv_data);
		return;
	}

	/* 在socket上进行监听 */
	if (listen(sock, 5) == -1)
	{
		rt_kprintf("Listen error\n");

		/* release recv buffer */
		rt_free(recv_data);
		return;
	}

	rt_kprintf("\nTCPServer Waiting for client on port 5000...\n");
	while (stop != RT_TRUE)
	{
		sin_size = sizeof(struct sockaddr_in);

		/* 接受一个客户端连接socket的请求，这个函数调用是阻塞式的 */
		connected = accept(sock, (struct sockaddr *) &client_addr, &sin_size);
		/* 返回的是连接成功的socket */

		/* 接受返回的client_addr指向了客户端的地址信息 */
		rt_kprintf("I got a connection from (%s , %d)\n", inet_ntoa(
				client_addr.sin_addr), ntohs(client_addr.sin_port));

		/* 客户端连接的处理 */
		while (1)
		{
			/* 发送数据到connected socket */
			send(connected, send_data, strlen(send_data), 0);

			/*
			 * 从connected socket中接收数据，接收buffer是1024大小，
			 * 但并不一定能够收到1024大小的数据
			 */
			bytes_received = recv(connected, recv_data, 1024, 0);
			if (bytes_received < 0)
			{
				/* 接收失败，关闭这个connected socket */
				lwip_close(connected);
				break;
			}

			/* 有接收到数据，把末端清零 */
			recv_data[bytes_received] = '\0';
			if (strcmp(recv_data, "q") == 0 || strcmp(recv_data, "Q") == 0)
			{
				/* 如果是首字母是q或Q，关闭这个连接 */
				lwip_close(connected);
				break;
			}
			else if (strcmp(recv_data, "exit") == 0)
			{
				/* 如果接收的是exit，则关闭整个服务端 */
				lwip_close(connected);
				stop = RT_TRUE;
				break;
			}
			else
			{
				/* 在控制终端显示收到的数据 */
				rt_kprintf("RECIEVED DATA = %s \n", recv_data);
			}
		}
	}

	/* 退出服务 */
	lwip_close(sock);

	/* 释放接收缓冲 */
	rt_free(recv_data);

	return;
}

#ifdef RT_USING_FINSH
#include <finsh.h>
/* 输出tcpserv函数到finsh shell中 */
FINSH_FUNCTION_EXPORT(tcpserv, startup tcp server);
#endif
~~~

下面则是另一个如在RT-Thread上使用BSD socket接口的TCP客户端例子。当把这个代码加入到RT-Thread时，它会自动向finsh 命令行添加一个tcpclient命令，在finsh上执行tcpclient(url,port)函数即可启动这个TCP服务端，url指定了这个客户端连接到的服务端地址或域名，port是相应的端口号。
当TCP客户端连接成功时，它会接收服务端传过来的数据。当有数据接收到时，如果是以q或Q开头，它将主动断开这个连接；否则会把接收的数据在控制终端中打印出来，然后发送“This is TCP Client from RT-Thread.”的字符串。

~~~{.c}
/*
 * 程序清单：TCP客户端例子
 */
#include <rtthread.h>
#include <lwip/netdb.h> /* 为了解析主机名，需要包含netdb.h头文件 */
#include <lwip/sockets.h> /* 使用BSD socket，需要包含sockets.h头文件 */

/* 发送用到的数据 */
ALIGN(4)
static const char send_data[] = "This is TCP Client from RT-Thread.";
void tcpclient(const char* url, int port)
{
	char *recv_data;
	struct hostent *host;
	int sock, bytes_received;
	struct sockaddr_in server_addr;

	/* 通过函数入口参数url获得host地址（如果是域名，会做域名解析） */
	host = gethostbyname(url);

	/* 分配用于存放接收数据的缓冲 */
	recv_data = rt_malloc(1024);
	if (recv_data == RT_NULL)
	{
		rt_kprintf("No memory\n");
		return;
	}

	/* 创建一个socket，类型是SOCKET_STREAM，TCP类型 */
	if ((sock = socket(AF_INET, SOCK_STREAM, 0)) == -1)
	{
		/* 创建socket失败 */
		rt_kprintf("Socket error\n");

		/* 释放接收缓冲 */
		rt_free(recv_data);
		return;
	}

	/* 初始化预连接的服务端地址 */
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(port);
	server_addr.sin_addr = *((struct in_addr *) host->h_addr);
	rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

	/* 连接到服务端 */
	if (connect(sock, (struct sockaddr *) &server_addr, 
		sizeof(struct sockaddr)) == -1)
	{
		/* 连接失败 */
		rt_kprintf("Connect error\n");

		/*释放接收缓冲 */
		rt_free(recv_data);
		return;
	}

	while (1)
	{
		/* 从sock连接中接收最大1024字节数据 */
		bytes_received = recv(sock, recv_data, 1024, 0);
		if (bytes_received < 0)
		{
			/* 接收失败，关闭这个连接 */
			lwip_close(sock);

			/* 释放接收缓冲 */
			rt_free(recv_data);
			break;
		}

		/* 有接收到数据，把末端清零 */
		recv_data[bytes_received] = '\0';

		if (strcmp(recv_data, "q") == 0 || strcmp(recv_data, "Q") == 0)
		{
			/* 如果是首字母是q或Q，关闭这个连接 */
			lwip_close(sock);

			/* 释放接收缓冲 */
			rt_free(recv_data);
			break;
		}
		else
		{
			/* 在控制终端显示收到的数据 */
			rt_kprintf("\nRecieved data = %s ", recv_data);
		}

		/* 发送数据到sock连接 */
		send(sock, send_data, strlen(send_data), 0);
	}

	return;
}

#ifdef RT_USING_FINSH
#include <finsh.h>
/* 输出tcpclient函数到finsh shell中 */
FINSH_FUNCTION_EXPORT(tcpclient, startup tcp client);
#endif
~~~
