# 关于/etc/group #
/etc/group 的内容包括用户组（Group）、用户组口令、GID及该用户组所包含的用户（User），每个用户组一条记录。

格式如下：**group_name:passwd:GID:user_list**。

在/etc/group 中的每条记录分四个字段：

第一字段：用户组名称；

第二字段：用户组密码；

第三字段：GID；

第四字段：用户列表，每个用户之间用逗号(,)分割；本字段可以为空；如果字段为空表示用户组为GID的用户名；

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/group%E7%94%A8%E6%88%B7%E7%BB%84.png)

# 关于/etc/password #

/etc/passwd中一行记录对应着一个用户，每行记录又被冒号(:)分隔为7个字段，其格式和具体含义如下：

*用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell*

如下图所示：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/etc_password.png)

解释如下：

- **用户名**

	代表用户帐号的字符串，就不多说了。

- **口令**
	
	一些系统中，存放着加密后的用户口令。虽然这个字段存放的只是用户口令的加密串，不是明文，但是由于/etc/passwd文件对所有用户都可读，所以这仍是一个安全隐患。因此，现在许多 Linux 系统（如SVR4）都使用了shadow技术，把真正的加密后的用户口令字存放到/etc/shadow文件中，而在/etc/passwd文件的口令字段中只存放一个特殊的字符，例如“x”或者“*”。比如上面截图，/etc/passwd展示的都是x。

- **用户标识号**
	
	一个整数，系统内部用它来标识用户。一般情况下它与用户名是一一对应的。如果几个用户名对应的用户标识号是一样的，系统内部将把它们视为同一个用户，但是它们可以有不同的口令、不同的主目录以及不同的登录Shell等。

- **组标识号**

	记录的是用户所属的用户组。它对应着/etc/group文件中的一条记录。 

- **注释性描述**

	记录着用户的一些个人情况，例如用户的真实姓名、电话、地址等，这个字段并没有什么实际的用途。

- **主目录**

	用户的起始工作目录，它是用户在登录到系统之后所处的目录。各用户对自己的主目录有读、写、执行（搜索）权限，其他用户对此目录的访问权限则根据具体情况设置。

- **登录Shell**

	用户登录后，要启动一个进程，负责将用户的操作传给内核，这个进程是用户登录到系统后运行的命令解释器或某个特定的程序，即Shell。
	
	Shell 是用户与Linux系统之间的接口。Linux的Shell有许多种，每种都有不同的特点。常用的有sh(Bourne Shell), csh(C Shell), ksh(Korn Shell), tcsh(TENEX/TOPS-20 type C Shell), bash(Bourne Again Shell)等。系统管理员可以根据系统情况和用户习惯为用户指定某个Shell。如果不指定Shell，那么系统使用sh为默认的登录Shell，即这个字段的值为/bin/sh。

	用户的登录Shell也可以指定为某个特定的程序（此程序不是一个命令解释器）。利用这一特点，我们可以限制用户只能运行指定的应用程序，在该应用程序运行结束后，用户就自动退出了系统。有些Linux 系统要求只有那些在系统中登记了的程序才能出现在这个字段中。比如上面截图中出现的`shutdown`用户，指定登录shell为`/sbin/shutdown`。

# 关于操作账户的一些命令 #

## useradd ##
**useradd**是Linux添加新用户的命令，这个命令提供了一次性创建新用户账户的方法。

- `useradd -D`

	查看系统的默认值。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/useradd_d.png)

	解释如下：

	1）新用户添加到GID为100的公共组
 
	2）新用户的HOME目录将会位于/homoe/username 

	3）新用户账户密码在过期后不会被禁用 

	4）新用户账户未被设置为某个日期后就过期 

	5）新用户账户将bash shell作为默认shell 

	6）系统会将/etc/skel目录下的内容复制到用户的HOME目录下 

	7）系统为该用户账户在mail目录下创建一个用于接收邮件的文件