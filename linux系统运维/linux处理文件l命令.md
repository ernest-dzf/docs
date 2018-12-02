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
