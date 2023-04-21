
### mac安装python3
1. brew search python
2. brew install python@3.11
   主要看python安装位置，我是/usr/local/bin/python3
   
3. vim ~/.zshrc
   source ~/.bash_profile
   vim ~/.bash_profile
   export PATH=/usr/local/bin
   alias python="/usr/local/bin/python3"

4. 关闭iterm2，重启，输入python测试