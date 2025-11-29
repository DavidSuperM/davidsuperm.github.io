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
   mybatisX、 ide eval reset、maven helper、python、sonarlint、vue.js,easyyapi) 
   datagrip
   3.2 homebrew(官网复制命令安装，可能unaccess to github.. 可用镜像源安装   /bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"  )
      （m1的mac homebrew是安装在/opt/homebrew位置，而intel的是安装在/usr/local所以环境变量配置也不一样，m1为例，需要vim ~/.bash_profile  添加 export PATH=${PATH}:/opt/homebrew/bin，不添加的话比如brew安装mysql，在非安装目录下无法 mysql -uroot -p 使用）
      homebrew（参考<https://zhuanlan.zhihu.com/p/90508170>）、git (命令行：brew install git)
   3.3 brew安装java(java直接通过idea下载吧)（正常是 brew search openjdk, brew install openjdk@8, 但是openjdk没有1.8的版本，要么brew install openjdk@11,要么去azul官网下载jdk8安装包，网站：<https://www.azul.com/core-post-download/?endpoint=zulu&uuid=3646391e-f4e3-4eac-bc39-40de6ce28ffe>）
4. iterm （参考：<https://sysin.org/blog/macos-zsh/> <https://www.jianshu.com/p/a91b8d75a6d7>）
5. sublime  笔记有注册码
6. postman 
7. 关闭Spotlight快捷键(系统设置->键盘-键盘快捷键->聚焦)、切换输入法快捷键为command+空格 
8. mysql (brew install mysql@5.7)(启动mysql:brew services start mysql@5.7，关闭mysql：brew services stop mysql@5.7)（启动后可以运行 mysql_secure_installation 可以设置密码） 
9. 设置easycode
10. 
11. 禁止工作区更换位置<https://www.zhihu.com/question/49530172> 系统设置->桌面与程序坞->调度中心
12. iterm设为默认终端（）
13. 鼠标滚轮和外接键盘ctrl设置(会影响触控板)（<https://www.edianyun.com/wenda_czxt_macos/162.html><https://blog.csdn.net/Nick_Zhang_CSDN/article/details/99494240https://blog.csdn.net/Nick_Zhang_CSDN/article/details/99494240>）
14. 安装npm(brew install npm)
15. 安装python、(brew search python,brew install python@3.11,设置环境变量 )
16. brew install maven,配置环境变量
17. 安装redis
18. 安装ntfs硬盘读写工具 mounty (请看其他.md)(或者赤友ntfs，印象笔记有有注册码)
19. 安装maczip，keka压缩工具，mac自带的压缩会有多余的缓存文件。 maczip搜官网下载。keka用brew instal keka安装
20. bartender4 (5也有,需要最新系统，收费，为了管理状态栏多的图标 看不见,可以用这个展开 )  笔记有注册码
21. runcat


### idea设置
idea设置主体 settings->appearance&behavior->appearance->theme->darcula
idea设置utf8  
2024idea恢复2021版ui界面    settings->appearance&behavior->new ui-> 取消勾选 enable new ui
设置maven目录
file->project structure -> SDK,language

@Data注解不生效，代码里报红
打开 IDEA → Preferences 
搜索 annotation processors
勾选 Enable annotation processing
点击 Apply → OK

maven设置
/Users/david/config/settings.xml
/Users/david/config/repository

utf8设置
editor-file encodings- 改为utf8 ,不勾选 translate to ascii



### sublime设置
sublime key bindings:
[
{ "keys": ["command+option+j"], "command": "pretty_json" },
{ "keys": ["command+option+l"], "command": "join_lines" }
]

settings：
{
   "ignored_packages":
   [
   "Vintage",
   ],
   "index_files": true,
   "update_check": false,
   "open_files_in_new_window": false
}

### chrome设置
打开链接是本页面打开改为 使用新页面打开
安装插件 Open Links in New Tab

### 文件夹中打开iterm
系统设置->键盘->键盘快捷键->服务->文件和文件夹中勾选iterm
这样文件夹中右击，选择服务，能打开iterm


#### git pull push 失败， ping  github.com 超时  但是网页能访问
是因为git没有走代理，命令行设置
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
一般是这个，具体要看vpn的监听地址和端口，设置里或者 yaml配置文件里能看到





