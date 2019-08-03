# 文本处理 #
## 处理文本文件的linux常用命令 ##
- `wc -l`
	
	获取文本文件的行数
- `cat file1.txt >> file2.txt`

	将file1.txt追加到file2.txt

- `split -b 10k date.file`

	将date.file分割为10k大小的文件
## sed ##

## awk ##

- 按照空格对一行文本分段，打印某一段内容
	
		[user_00@TENCENT64 ~]$ echo "1234 3456 8765"|awk -F '[ ]' '{print $2}'
		3456
		[user_00@TENCENT64 ~]$ 

- 日志文件中一般会以特定字符串作为开头，但是一条日志记录可能跨越多行。使用awk将每条日志记录变为单行

		[user_00@TENCENT64 ~]$ cat test.txt 
		[2018 hello,how are you
		fine thank you 
			you are welcome
		
		[2018 6789%%%%#### 
		
		this is only a test
		[user_00@TENCENT64 ~]$ cat test.txt | awk '/^\[2018/{if (n++) print ""}{printf $0}'
		[2018 hello,how are youfine thank you 	you are welcome
		[2018 6789%%#### this is only a test[user_00@TENCENT64 ~]$ 

- 找出日志中所有调用的接口，并去重


	日志中会打印出接收到的请求，形式如下：

		[00:00:07 2019-05-22] ROOT : INFO [2208348][handler_interface.go:202]127.0.0.1:25841    POST    /interface                      17      handle request: {"eventId":101,"version":"1.1","callee":"cgw","caller":"yunapi","interface":{"interfaceName":"qcloud.tdsql.listInstance","para":{"userType":1,"appId":1253237097,"uin":"2739328207","uuids":["tdsql-893elvqz"]}}}       Response:      {"version":"1.1","caller":"yunapi","callee":"cgw","eventId":101,"timestamp":1558454407716,"returnValue":0,"returnMsg":"OK","returnData":{"totalCnt":1,"instances":[{"id":11855,"originSerialId":"set_1513924696_8207","serialId":"set_1513924696_8207","uuid":"tdsql-893elvqz","instancename":"prd-wrk","appid":1253237097,"projectId":1028503,"regionId":1,"zoneId":100003,"nettype":1,"vpcId":6830,"subnetId":21660,"status":2,"vip":"10.40.22.15","vport":3306,"wanDomain":"","wanVip":"","wanPort":"0","wanStatus":"0","createtime":"2017-12-22 14:37:38","updatetime":"2019-05-09 14:28:32","lockerdesc":"运行中","zkName":"zk-gz-online","clusterName":"noshard1","shard":0,"istmp":0,"srcId":0,"remark":"","autoRenewFlag":1,"deactiveInit":0,"oldflag":0,"nodeNum":2,"tgwVip":"","tgwIspid":0,"dbVersion":"10.1.9","exclusterId":"","paymode":0,"specConfigId":54,"periodendtime":"2019-06-15 14:37:38","validtime":"","uin":"2739328207","machine":"TS85","memory":2000,"storage":100000,"qps":2100,"typename":"标准版"," 


	我们要找到所有的接口名，类似`qcloud.tdsql.listInstance`这种。

	首先找到包含接口名的行，这一行中会包含`POST`关键字。

	然后再按照正则`interfaceName.*para`过滤一下，缩小需要处理的范围。

	最后按照`"`分割，取出第三段，就是接口名了。

		grep "POST" shark.log|grep -o -E "interfaceName.*para"|awk -F '["]' '{print $3}'|sort|uniq	