mac本身安装了ssh服务，默认情况下不会开机自启

1. 启动

   ```shell
   sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
   ```

   

2. 停止

   ```shell
   sudo launchctl unload -w /System/Library/LaunchDaemons/ssh.plist
   ```

3. 查看

   ```shell
   sudo launchctl list | grep ssh
   ```

   

