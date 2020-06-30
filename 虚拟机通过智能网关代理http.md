在mac上的虚拟机，我们怎么访问google呢？

moa上的NGA代理打通之后，会在mac本机上监听12639端口，但是这个12639端口只能用于mac上的应用去连，因为是在localhost上进行监听的。

我们可以另起一个12638端口，让其在vmnet8 地址上进行监听，然后转发流量到本机的12639端口。

```shell
ssh -fN4L 192.168.6.1:12638:127.0.0.1:12639 victordong@127.0.0.1
```



上面 `192.168.6.1`就是mac宿主机的vmnet8地址，

```
vmnet8: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether 00:50:56:c0:00:08
	inet 192.168.6.1 netmask 0xffffff00 broadcast 192.168.6.255
```

这样mac上的虚拟机就可以设置http代理为`192.168.6.1:12638`了。