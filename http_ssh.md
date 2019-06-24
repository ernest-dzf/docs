# 通过http代理ssh连接cvm原理分析 #

在公司办公网或者开发网环境下使用xshell的时候，有时候连不上在腾讯云上购买的cvm。使用cvm公网ip连也不行。

在km上搜索一番，你可能会被告知使用`dev-proxy.oa.com`或者`web-proxy.oa.com`代理，然后就可以登录了。就像下面这样。
【http_ssh_1.png】

那这里背后的原理是什么呢？数据是如何流动的？

先抓下包。

【http_ssh_2.png】

- [249 - 251]，这三个是我们熟悉的TCP三次握手报文。猜想这三次握手是和http代理服务器握手的报文。可以看到目的ip是10.34.28.15。通过ping我们也验证了这个。
	【dev-proxy.png】
- [252 - 254]，这三个报文是xshell客户端请求http代理服务器去和cvm建立一个连接。发起请求的是报文[252]，通过http协议的CONNECT方法，请求http代理服务器，让http代理服务器去和150.109.76.79:22建立连接。然后http代理服务器通过http报文[254]告诉xshell，连接已经建立好。

- [256 - ]，接下来256后面的报文就是xshell和cvm通过ssh协议进行交互的过程了。 http代理服务器无脑将xshell发过来的tcp报文的payload传输给cvm的sshd进程。相当于打了一个隧道。

【http_ssh_3.png】
