# crontab 命令 #

crontab命令基本格式：

```shell
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

```



```shell
* * * * * /usr/local/sa/agent/secu-tcs-agent-mon-safe.sh  > /dev/null 2>&1
*/20 * * * * /usr/sbin/ntpdate ntpupdate.tencentyun.com >/dev/null &
*/3 * * * * /usr/sbin/ldap_monitor.sh >/dev/null &

```



`* * * * *`	command

5个*号分别表示分、时、日、月、周以及命令。

- 第1列表示分钟，1～59， 每分钟用*或者 */1表示

- 第2列表示小时，0～23（0表示0点）


- 第3列表示日期，1～31

- 第4列表示月份，1～12

- 第5列标识号星期，0～6（0表示星期天）

- 第6列要运行的命令


## 例子

1. `30 21 * * * /usr/local/etc/rc.d/lighttpd restart`
2. `45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart`
3. `10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart`
4. `0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart`
5. `0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart`
6. `0 */1 * * * /usr/local/etc/rc.d/lighttpd restart`
7. `0 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart`
8. `0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart`

