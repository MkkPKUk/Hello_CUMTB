**啥是DNS劫持**：打开的目标网站不是原来的内容，反而跳转到了未知的页面，即使终端用户输入正确的网址也会被指向跳转至那些恶意网站，或者本来能正常访问的页面，突然就打不开了，这就是DNS劫持，亦可称域名劫持。比如运营商可能因为利益原因，恶意劫持DNS弹出广告

**如何解决**：

1.	手动设置DNS服务器国内的114.114.114.114，谷歌提供的8.8.8.8。

2.	在宽带路由器上配置DHCP服务器，在DHCP服务器中指定好DNS服务器地址，连接该路由器的所有设备都将使用该指定的DNS服务器地址。

闲扯: 我记得好想在知乎上看到有个人通过一系列测试发现小米路由器劫持了他的网络请求包，让每次返回的网页都有广告。详细请看： https://zhuanlan.zhihu.com/p/22478237?utm_source=qq&utm_medium=social&utm_oi=1124232026961846272 
