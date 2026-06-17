---
title: Scapy函数的使用
date: 2022-09-25 11:12:45
categories:
  - 技术笔记
  - 渗透测试
tags:
  - Python
  - Scapy
  - 抓包
  - 《Python渗透测试实战》
---
# Scapy函数的使用 #
本文章为书上第三章的内容。

## Scapy简介 ##
Scapy是可以直接操作到数据包的工具。在Windows中可以在Python环境下将Scapy当作一个库来使用；在Linux中可以将Scapy当作一个独立的工具来使用。Scapy本身就是一个可以独立运行的工具，具备一个独立运行的环境，可以不依赖Python。
Scapy使用了“类+属性”的方法来构造数据包。每一个网络协议就是一个类，协议中的字段对应着属性。

## Scapy代码 ##
### 启动 ###
在Kali Linux下，直接启动终端输入命令*Scapy*，就可以启动Scapy编程环境。

### 创建数据包 ###
首先导入Scapy库，选择“*from 模块 import 类*”形式导入，如构造一个IP数据包:

    from scapy.all import IP
    pkt=IP()
    print(pkt)

Scapy采用分层方式构造数据包，最底层协议为Ether，然后是IP，再之后是TCP或UDP。分层是通过符号“/”实现的，按照协议由底而上从左向右排列。例如：
使用Ether()/IP()/TCP()构造一个TCP数据包。
    from scapy.all import *
    pkt=Ether()/IP()/TCP()
    ls(pkt)
终端会以“--”为分隔符，根据协议从左到右的顺序从上到下展示协议属性。

构造一个HTTP数据包。
    pkt=IP()/TCP()/"GET / HTTP/1.0\r\n\r\n    
### 查看数据包格式 ###
    from scapy.all import IP
    pkt=IP()
    ls(pkt)

执行上述程序，就可以查看数据包里的类设置参数，例如：
    Ether(dst="ff:ff:ff:ff:ff:ff")
*Ether类是最底层协议，这个类可以设置接收方和发送方的MAC地址*

### Scapy常用函数 ###
查看协议所有属性

    ls([协议])

查看所有函数

    lsc()    

row()函数表示以字节格式来显示数据包内容。

    pint(row([数据包]))    

hexdump()函数表示以十六进制数据表示的数据包内容

    print(hexdump([数据包]))

summary()函数使用不超过一行的摘要内容简单描述数据包

    [数据包].summary()

show()函数和show2()函数使用展开视图的方式显示数据包的详细信息，可以快速查看每一个属性的值。区别在于show2()会显示数据包的校验和。

    [数据包].show()

command()帮助列出构造一个数据包所需的命令，以供我们构造相同的数据包。

    [数据包].command()

wrpcap()函数可以将临时存放在pkt的数据包保存在指定文件中。

    wrpcap("temp.cap",pkt)    

rdpcap()函数可以从文件中读取数据包。

    [数据包]=rdpcap("temp.cap")

# 在Scapy中发送和接收数据包 #
## 发送数据包 ##
send()和sendp()函数可用于发送数据包。需要将MAC地址作为目标时，就使用sendp()函数；需要将IP地址作为目标时，就使用send()函数。
特点是只发不收，不会处理应答数据包。

    from scapy.all import *
    pkt=IP(dst="192.168.1.1")/ICMP()
    send(pkt)

    sendp(Ether(dst="ff:ff:ff:ff:ff:ff"))

## 接收数据包 ##
Scapy产生的数据包被发送出去之后，Scapy就会监听接收到的数据包，将其中对应的应答数据包筛选出来显示。
sr()、srl()和srp()函数可用于发送和接收数据包。sr()和srl()函数主要用于IP地址，srp()函数用于MAC()地址。 srl()函数和sr()函数相比只返回一个应答数据包。
sr()函数是Scapy的核心，它的返回值是两个列表，第一个列表包含收到了的应答数据包和对应的应答数据包，第二个列表包含未收到应答的数据包。
所以使用两个列表来保存sr()函数的返回值。

    from scapy.all import *
    pkt=IP(dst="192.168.1.1")/ICMP()
    ans,uans=sr(pkt)
    ans.summary()

*ans.summary可以查看发送的数据包和收到的应答数据包的内容*

# Scapy中的抓包函数 #
sniff()函数可以在程序中捕获经过本机网卡的数据包。函数完整格式为*sniff(filter="",iface="any",prn=function,count=N)*。

## 过滤语句 ##
第一个参数filter，用来对数据包进行过滤，如我们指定只捕获与192.168.1.1有关的数据包，就可以使用"host 192.168.1.1"。

    sniff(filter=" host 192.168.1.1")

伯克利包过滤法：采用和自然语言很接近的语法构成的字符串，确定保留哪些数据包和忽略哪些数据包。
空字符串表示匹配所有的数据包。如果字符串不为空，那么只有字符串表达式为“真”的数据包会被保留。字符串由一个或多个原语构成，每个原语由一个标识符（名称或数字）组成，后面跟着一个或多个限定符。
限定符有以下三种：
+ Type：表示指代的对象。如IP地址、子网或端口等。如*host(主机名和IP地址),net(子网),port(端口))。没有则默认为host*
+ Dir：表示传输的方向。常见的有src和st，默认为"src or dst"
+ Proto：表示与数据包匹配的协议类型。常见的是 Ehter,IP,TCP,ARP协议

可以使用and,or和not把多个原语组成一个更复杂的过滤语句。如*host 192.168.1.1 and port 8080*

## 指定网卡 ##
iface参数用来指定要使用的网卡，默认为第一块网卡。

## 处理捕获的数据包 ##
prn表示对捕获到的数据包进行处理的函数，可以使用Lambda表达式。例如：
调用函数，将获取到的数据包进行输出

    sniff(filter="icmp", prn=lambda x:x.summary())

也可以定义为回调函数。

    def packet_callback(pkt):
        print (pkt.summary)
    sniff(prn=packet_callback)

## 指定监听数据包的数量 ##
count参数用来指定监听到数据包的数量，到达指定数量就会停止监听。

# 设置一个综合性监听器 #
在网卡eth0上监听源地址或目标地址为 192.168.1.1的ICMP数据包并输出，当监听了3个这样的数据包后就会停止监听。

    sniff(filter="icmp and host 192.168.1.1",prn=lambda x:x.summary(),count=3)