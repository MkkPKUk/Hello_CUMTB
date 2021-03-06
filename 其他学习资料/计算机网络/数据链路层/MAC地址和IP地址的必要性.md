# 数据链路层

注意，此内容为《计算机网络自顶向下方法》中的内容。笔者看到了解其中道理兴奋所写，难免有错。如需深入了解可以参见书本的 302~308页

### 链路层寻址

总所周知，主机和路由器具有网络层地址，为什么又跑出来个链路层的地址呢？具体原因先留个悬念



**1 MAC地址**

事实上并不是主机和路由器具有的MAC地址，而是他们的适配器(即网络接口)具有链路层地址。**然而，重要的是注意到链路层交换机并不具有设配器，因为链路层交换机的任务就是单纯的在主机和路由器之间承载数据报。**交换机透明地执行该项任务，这就是说，主机或路由器不必明确地将帧寻址到其间的交换机上。

链路层地址也有不同的称呼：LAN地址，物理地址，或MAC地址。因为MAC地址较为流行所以这样称呼。

同时，MAC地址的广播地址是：FF-FF-FF-FF-FF-FF 48个连续的1来组成。 



那问题来了，我们既然都有了IP地址，为什么还需要这个MAC地址呢，他又什么用呢？

> 一：局域网是为了人一网络层协议设置的，并不只是为了IP和因特网。这个网络适配器中的MAC地址，这个`中性`的东西就能很好的支持其他协议。应为是烧入网卡中的BIOS里面的。
>
> 二：通过主机和路由器我们能找到网络表示，但是无法在一个局域网段中找到对应的主机。这就要我们的交换机通过内部的ARP表进行对比，通过对应的IP地址，然后找到对应的MAC地址，然后把信息传过去。(注意，路由器中的路由表只是负责转发网络标识，并不是负责转发到对应的主机，所以需要中间设备`交换机`出马！)



### ARP

如果理解了交换机的作用，那ARP就很简单了。

假设一个场景。一个局域网段的一个主机要发送一个信息到另一个网段，首先它肯定要知道默认网关(一般都是路由器)的MAC地址，不然没法发。然后这个主机会交给交换机然后广播一个ARP包。当然，这个交换机肯定会广播到默认网关上(路由器)。然后路由器的`网络适配器`收到一看，这个IP和我的IP一样，然后就把自己的MAC地址塞进去。然后交给交换机，前面说过，交换机维护一张ARP表。ARP表看懂源IP就知道要往哪里发送这个帧了。所以我们的主机就能知道默认网关(路由器)的MAC地址了。然后通过路由器的跳转。交换机通过完整的IP传送MAC地址。这样我们的报文就能成功发送到不同网段的目的主机了！！

需要注意的一点就是在路由器跳的过程中。目的IP和源IP不会变，但是目的MAC地址和源MAC地址会变的。所以我们主要依靠的是IP地址来找到网端，MAC找到具体的主机
