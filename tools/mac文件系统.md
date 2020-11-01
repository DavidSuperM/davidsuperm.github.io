## 修改hosts
```
vim /etc/hosts
```

## mac下的环境变量配置文件
```
a. /etc/profile 
b. /etc/paths 
c. ~/.bash_profile 
d. ~/.bash_login 
e. ~/.profile 
f. ~/.bashrc 
```
>1. 其中a和b是系统级别的，系统启动就会加载，其余是用户接别的。
>2. c,d,e按照从前往后的顺序读取，如果c文件存在，则后面的几个文件就会被忽略不读了，以此类推。
>3. ~/.bashrc没有上述规则，它是bash shell打开的时候载入的。
>4. 这里建议在c中添加环境变量

## mac原生terminal添加环境变量
```
vim ~/.bash_profile 
export PATH=/usr/local/apache-maven-3.6.3/bin:$PATH
source ~/.bash_profile    // 立马生效
```

## mac安装了iterm2，zsh，添加环境变量
因为zsh不会加载~/.bash_profile，而是加载~/.zshrc，所以原方法没用
需要
```
1. vim ~/.zshrc
2. 添加 source ~/.bash_profile
3. vim ~/.bash_profile
4. export PATH=/usr/local/apache-maven-3.6.3/bin:$PATH
```
这样关闭打开的iterm2，重新打开iterm2就生效了，也不用再额外手动执行source ~/.bash_profile



