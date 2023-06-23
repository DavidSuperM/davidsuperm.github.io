# mac重装系统

### 重装系统
1. 备份照片资料、idea工作目录，push代码，导出easycode模板，备份数据库数据
2. 关机
3. 开机，按"command+R"进入恢复界面
4. 选择磁盘工具
5. 选择主硬盘，抹掉。
6. 关闭对话框，返回上一个界面，选择重新安装macos
7. 后面就是继续、设置新电脑的界面。（装的的是最新版本的系统）

参考：
<https://zhuanlan.zhihu.com/p/269663212>
<https://zhuanlan.zhihu.com/p/150580764>

### 安装软件
1. 360极速浏览器、chrome、印象笔记、qq、微信、wps、clashx、omi ntfs、qq影音、qq音乐、网易云音乐、有道词典、fiddler
2. clipy <https://clipy-app.com/>
3. idea(2021.1.3专业版)（设置jdk）(安装插件 *.ignore、alibaba java coding guidliness、bashsupport pro、easycode-mybaitscodehelper、
   mybatisX、 ide eval reset、maven helper、python、sonarlint、vue.js) 
   datagrip
4. iterm （参考：<https://sysin.org/blog/macos-zsh/> <https://www.jianshu.com/p/a91b8d75a6d7>）
5. sublime text (安装 json插件、Log Highlight 日志查看插件)
6. postman
7. homebrew（参考<https://zhuanlan.zhihu.com/p/90508170>）、git (命令行：brew install git)
8. 关闭Spotlight快捷键(系统设置->键盘-键盘快捷键->聚焦)、切换输入法快捷键为command+空格
9. mysql (brew install mysql@5.7)(启动mysql:brew services start mysql@5.7，关闭mysql：brew services stop mysql@5.7)（启动后可以运行 mysql_secure_installation 可以设置密码）
10. 设置easycode
11. 禁止工作区更换位置<https://www.zhihu.com/question/49530172> 系统设置->桌面与程序坞->调度中心
12. iterm设为默认终端（）
13. 鼠标滚轮和外接键盘ctrl设置(会影响触控板)（<https://www.edianyun.com/wenda_czxt_macos/162.html><https://blog.csdn.net/Nick_Zhang_CSDN/article/details/99494240https://blog.csdn.net/Nick_Zhang_CSDN/article/details/99494240>）
14. 安装npm(brew install npm)
15. 安装python、(brew search python,brew install python@3.11,设置环境变量 )
16. brew intall maven,配置环境变量
17. 安装redis
18. 安装ntfs硬盘读写工具 mounty
