# tcpdump使用

常用的选项如下：

1. `-i interface`，指定tcpudmp需要抓包的网卡，如果需要抓取loopback接口，使用`tcpdump -i lo`
2. 指定主机和端口，`tcpdump tcp port 22`
3. 指定来源
4. tcpdump使用的时候最好加上`-s0`选项，当你不适用该参数选项的时候，你抓的网络数据通过wireshark来分析的时候，可能出现“packet size limited during capture”
5. 抓的包保存到文件，`tcpdump -w data.pcap`
6. 仅仅抓目的端口是8889的包，`tcpdump -i en0 tcp dst port 8889 -w 8889_1.pcap &`
7. 仅仅抓源端口是8889的包，`tcpdump -i en0 tcp src port 8889 -w 8889_4.pcap`
8. 仅仅抓源ip是124.156.182.201的包，`tcpdump -i en0 src host 124.156.182.201 -w 8889_7.pcap`
9. 仅仅抓目的ip是124.156.182.201的包，`tcpdump -i en0 dst host 124.156.182.201 -w 8889_7.pcap`
10. tcpdump -Z root, 如果你是使用root用户运行tcpdump的话，那么在tcpdump打开输出文件前，会将用户切为默认的tcpdump用户，将用户组切为tcpdump用户的primary group。这可能会让你无法保存输出的抓包文件。  可以通过`tcpdump -Z root`将这个切换动作disable掉。