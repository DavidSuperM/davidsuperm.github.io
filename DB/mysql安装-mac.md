# mysql安装-mac

1. brew install mysql  //默认安装8.版本
或者 brew install mysql@5.7  //指定安装5.7版本
2. ln -sfv /usr/local/opt/mysql@5.7/*.plist ~/Library/LaunchAgents  //设置开机启动 没试过
3. brew services start mysql@5.7 // 启动服务 
4. brew services stop mysql@5.7     //停止
5. brew services restart mysql@5.7 //重启


3. vi ~/.zshrc
   export PATH="/usr/local/Cellar/mysql@5.7/5.7.32/bin:$PATH"
   export PATH=${PATH}:/usr/local/Cellar/mysql@5.7/5.7.24/bin
   source ~/.zshrc 
   // 修改环境变量(好像不设置也没事)

4. mysql.server start    //启动服务


5. mysql_secure_installation  // 设置密码等其他设置  一定要先启动mysql服务，否则会报 Can't connect to local MySQL server through socket '/tmp/mysql.sock' 这个错