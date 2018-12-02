# mysql 字符集 #

这里先看下mysql和字符集相关的变量。以mysql 5.7为例，

	mysql> show variables like '%character_%';
	+--------------------------+----------------------------------------------------------------+
	| Variable_name            | Value                                                          |
	+--------------------------+----------------------------------------------------------------+
	| character_set_client     | utf8                                                           |
	| character_set_connection | utf8                                                           |
	| character_set_database   | latin1                                                         |
	| character_set_filesystem | binary                                                         |
	| character_set_results    | utf8                                                           |
	| character_set_server     | latin1                                                         |
	| character_set_system     | utf8                                                           |
	| character_sets_dir       | /usr/local/mysql-5.7.23-linux-glibc2.12-x86_64/share/charsets/ |
	+--------------------------+----------------------------------------------------------------+
	8 rows in set (0.00 sec)
	
	mysql> 


字符集的设置是通过环境变量来设置的，环境变量和linux中的环境变量是一个意思。

mysql的环境变量分为两种：session和global。session变量是仅在这次会话中有效，在mysql中，一次会话可以理解为当前连接（除非reload，否则，一次会话就只有一次连接）。global环境变量则是确定了下一个新建立的session的变量值。

使用show variables可以查看session值，如果要查看global的环境变量，则用show golbal variables语句。

设置session环境变量用set variablename=value,设置global环境变量用set global variablename=value。

环境变量可以在服务启动后用set来设置，有些（主要是和数据库服务器有关的）也可以在启动服务时用命令行参数指定，还可以在配置文件中设置（my.cnf）。

mysql提供了四个等级的默认字符集以及比较规则，分别是服务器、数据库、表、列级别的。原话如下：

> There are default settings for character sets and collations at four levels: server, database, table, and column. The description in the following sections may appear complex, but it has been found in practice that multiple-level defaulting leads to natural and obvious results.

一个字符集（character set）对应了一个默认的字符排序码规则（collation）,当改变了一个等级的默认字符集时，与它同等级的默认字符排序规则也会变成该字符集对应的字符排序规则。  

**除了有这四个等级的默认字符编码和排序规则，还可以指定具体某一段字符的编码以及他的排序规则**


如果要查看所有的字符集，用show characterset语句；查看所有的collation，用show collation语句。比如下面：

	mysql> show character set\G
	           Maxlen: 1
	*************************** 16. row ***************************
	          Charset: gb2312
	      Description: GB2312 Simplified Chinese
	Default collation: gb2312_chinese_ci
	           Maxlen: 2
	*************************** 17. row ***************************
	          Charset: greek
	      Description: ISO 8859-7 Greek
	Default collation: greek_general_ci
	           Maxlen: 1
	*************************** 18. row ***************************
	          Charset: cp1250
	      Description: Windows Central European
	Default collation: cp1250_general_ci
	           Maxlen: 1
	*************************** 19. row ***************************
	          Charset: gbk
	      Description: GBK Simplified Chinese
	Default collation: gbk_chinese_ci
	           Maxlen: 2
	*************************** 20. row ***************************
	          Charset: latin5
	      Description: ISO 8859-9 Turkish
	Default collation: latin5_turkish_ci
	           Maxlen: 1
	*************************** 21. row ***************************
	          Charset: armscii8
	      Description: ARMSCII-8 Armenian
	Default collation: armscii8_general_ci
	           Maxlen: 1
	*************************** 22. row ***************************
	          Charset: utf8
	      Description: UTF-8 Unicode
	Default coll
	Default collation: cp1257_general_ci
	           Maxlen: 1
	*************************** 36. row ***************************
	          Charset: utf32
	      Description: UTF-32 Unicode
	Default collation: utf32_general_ci
	           Maxlen: 4
	*************************** 37. row ***************************
	          Charset: binary
	      Description: Binary pseudo charset
	Default collation: binary
	           Maxlen: 1
	*************************** 38. row ***************************
	          Charset: geostd8
	      Description: GEOSTD8 Georgian
	Default collation: geostd8_general_ci
	           Maxlen: 1
	*************************** 39. row ***************************
	          Charset: cp932
	      Description: SJIS for Windows Japanese
	Default collation: cp932_japanese_ci
	           Maxlen: 2
	*************************** 40. row ***************************
	          Charset: eucjpms
	      Description: UJIS for Windows Japanese
	Default collation: eucjpms_japanese_ci
	           Maxlen: 3
	*************************** 41. row ***************************
	          Charset: gb18030
	      Description: China National Standard GB18030
	Default collation: gb18030_chinese_ci
	           Maxlen: 4
	41 rows in set (0.00 sec)
	
	mysql> 


字符集的设定不仅影响着存储，还会影响客户端和数据库服务器的通信，关于数据编码，mqsql中涉及到下面几个问题：

1. 客户端发过来的数据使用什么字符集编码的？
2. 接收到数据之后，应该用什么编码格式编码之后再将数据插入到mysql server中？
3. 执行查询之后，查询出来的结果应该用什么编码集编码之后再返回？
4. 数据库的各种表的数据，应该用什么字符集编码，以及它们用什么排序？
5. 查询语句的字符串比较时，应该在哪一个标准里面来比较，比如：'Mueller' = 'Müller'是为真还是假？
6. 数据库的各种元数据，包括表名、数据库名、密码、用户名、以及comment等，用什么字符集表示？

针对这些问题，mysql 提供了不同的环境变量用来跟踪。比如：

	mysql> show variables like '%character_set%'\G
	*************************** 1. row ***************************
	Variable_name: character_set_client
	        Value: latin1
	*************************** 2. row ***************************
	Variable_name: character_set_connection
	        Value: latin1
	*************************** 3. row ***************************
	Variable_name: character_set_database
	        Value: latin1
	*************************** 4. row ***************************
	Variable_name: character_set_filesystem
	        Value: binary
	*************************** 5. row ***************************
	Variable_name: character_set_results
	        Value: latin1
	*************************** 6. row ***************************
	Variable_name: character_set_server
	        Value: latin1
	*************************** 7. row ***************************
	Variable_name: character_set_system
	        Value: utf8
	*************************** 8. row ***************************
	Variable_name: character_sets_dir
	        Value: /usr/local/mysql-5.7.23-linux-glibc2.12-x86_64/share/charsets/
	8 rows in set (0.00 sec)
	
	mysql> 

我们着重关注上面的1,2,3,5,6,7行。


在mysql中，可以为数据库指定默认的字符编码（db.opt文件），成为该数据库中每个新建表的默认字符编码集，但是对于已经建立的表则不受影响。在新建一个表时，也可以使用DEFUALT CHARACTER SET=xxx来指定表的字符编码。

在数据库的查询（select、update、insert）操作中，涉及到的字符编码有**character_set_client**, **character_set_connnection**, **character_set_result**三个变量，这三个变量是需要建立连接之后进行设置的。

1. 针对第一个问题，使用**character_set_client**环境变量来回答

	**character_set_client** ，这是用户告诉服务器，客户端发过来的SQL语句是用的什么字符集，要和客户端发出去的字节流采用的编码集一致，如果是shell，那么就是和shell的编码集一致，中文windows的cmd就是gbk。但是对于使用_utf8'xxx'标记的字符，则用标记的字符集解码。
2. 针对第二个问题，使用character_set_connetion环境变量来回答

	MySQL server 接收到用户查询后，按照character_set_client将其转化为character_set_connection设定的字符集。

	这里有个重点，**服务器接收到查询后，需要将查询转换一下字符集**。转换成哪种字符集呢？character_set_connetion指定的字符集。
3. 执行查询之后，查询出来的结果应该用什么编码集编码之后再返回？

	character_set_results ， MySQL将存储的数据转换成character_set_results中设定的字符集发送给用户，客户端获取到的结果就是以这种形式编码的。

	这里总结一下上面三点，**character_set_client，character_set_connetion和character_set_results这3个参数值是由客户端每次连接进来设置的，和服务器端没关系**。

4. 针对第四个问题，使用上面提到的四个等级的默认字符集（即character_set_server、character_set_database以及建立表时的DEFAULT CHARACTER SET=xxx和指定字段时的DEFAULT CHARACTER SET=xxx）以及排序规则来回答：

	**character_set_server决定了服务器的默认编码**，character_set_database决定了新建数据库的默认字符集，而数据库的字符集又决定了新建表的默认字符集，而表的字符集又决定了字段的默认字符集，如果没有通过DEFAULT CHARACTER SET=xxx来改变表的字符集，则新表就使用character_set_database指定的字符集。

5. 针对第五个问题，使用collation_connection来回答

	collation_connection变量制定了比较的规则。collation_connection值的形式如下：字符集_语言_ci（大小不写敏感）或字符集_语言_cs（大小写敏感）。像中文这样的，没有大小写，所以只能是ci，比如set collatioin_connection=gbk_chinese_ci。就是设置成中文字典的排序规则。除了按具体语言排序，还可以按照二进制的位置排序，比如utf8_bin。


	character_set_connection和collatioin_connection是一体的，设置了character_set_connection之后，collation_connection会跟着变成对应的默认排序规则，反之亦然。如果要显示的设置排序规则，可以用`SET NAMES 'charset_name' COLLATE 'collation_name'`。

	但是如果查询语句的字符串和表的字段比较，则collation_connection不适用，因为表的字段有它自己的字符排序规则，而它自己的排序规则优先级高于collation_connection。

6. 针对第六个问题，使用character_set_system来回答

	character_set_system表示元数据的字符集，默认就是utf8，而且不要去更改它，否则，因为类似于用户名密码这种东西，可能用各种奇葩的字符去表示，只有utf8能够容纳它们。如果变成了别的字符集，那么用户名和密码就不能用你想要的字符去表示了。

	需要注意的是，这个character_set_system也好，character_set_dababase、character_set_server也好，都是在数据库内部的保存格式，而不是返回到客户端的编码格式，返回到客户端的结果都会转化为character_set_results指定的字符集之后再返回。

## MySQL中的字符集转换过程 ##

1. MySQL Server收到请求时将请求数据从character_set_client转换为character_set_connection
2. 进行内部操作前将请求数据从character_set_connection转换为内部操作字符集，其确定方法如下：
	- 使用每个数据字段的CHARACTER SET设定值
	- 若上述值不存在，则使用对应数据表的DEFAULT CHARACTER SET设定值(MySQL扩展，非SQL标准)
	- 若上述值不存在，则使用对应数据库的DEFAULT CHARACTER SET设定值
	- 若上述值不存在，则使用character_set_server设定值
3. 将操作结果从内部操作字符集转换为character_set_results

## 设置字符集的命令 ##

`set names 'charset_name'`等价于下面三条语句的结合：

	SET character_set_client = charset_name;
	SET character_set_results = charset_name;
	SET character_set_connection = charset_name;