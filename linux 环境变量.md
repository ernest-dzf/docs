# linux 环境变量 #
## /etc/profile ##

此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。也就是说，这里修改的内容是对所有用户起作用的。

并从/etc/profile.d目录的配置文件中搜集shell的设置。

在profile文件添加或修改的内容需要注销系统才能生效。或者source一下，`source /etc/profile`。

## /etc/bashrc ##

为每一个运行bash shell的用户执行此文件，当bash shell被打开时,该文件被读取。也就是说，当用户shell执行了bash时，运行这个文件。

## ~/.bashrc ##

该文件存储的是专属于个人bash shell的信息，当登录时以及每次打开一个新的shell时,执行这个文件。在这个文件里可以自定义用户专属的个人信息。

## ~/.bash_logout ##

当每次退出系统(退出bash shell)时,执行该文件。