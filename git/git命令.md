## 1.合作开发分支流程
1. master是稳定的主分支。
2. 有新功能需求时
```
master: git checkout -b feature/2020-04-22_feature_name_1
/**
问：为什么出现一个新需求时要建一个新的分支来做而不是所有的变动一直用cat这个分支来做？
答：因为如果用cat做新需求的途中commit多次（因为新需求功能比较多，做完一个子功能commit一次），然后在中途发现项目两个bug，commit两个修复bug，然后又commit多次完成新需求，然后等窗口一起发布上线，但是上线后发现修复的bug没问题，新需求有问题，于是回滚，1.回滚了后修复的bug代码也回滚了，导致线上还是有bug，2.发布窗口到了，新需求还没完成但是要修复线上bug，无法发布，因为修复bug的commit和新需求的commit交杂在一起了，
结论： bug修复单独一个分支，新需求单独一个分支
*/
```

3. 开发
```
feature/2020-04-22_feature_name_1 : git add .
// 建议用idea的ui界面commit，这样是为了防止无意提交了不该提交的文件
feature/2020-04-22_feature_name_1 : git commit -m "" 
feature/2020-04-22_feature_name_1 : git push 
//git push需要先和远程分支建立连接
```
4. 发布
```
master : git pull
master : git checkout feature/2020-04-22_feature_name_1
feature/2020-04-22_feature_name_1 : git merge master
feature/2020-04-22_feature_name_1 : git checkout -b release/2020-05-12_feature_name_1
release/2020-05-12_feature_name_1: git push
```
5. release分支发布后生产验证没问题
```
master : git merge release/2020-05-12_feature_name_1
```

## 2.git命令
- 克隆项目
```
用idea的VCS->git->clone
git clone git@github.com:michaelliao/gitskills.git
```
- 切换分支cat
```
git checkout cat
git checkout -    //切换到上一个分支
```
- 创建并切换分支
```
git checkout -b cat
```
- 本地分支和远程分支建立连接
```
git branch --set-upstream-to=origin/cat cat
```
- 推送本地分支到远程仓库并建立连接
```
git push origin cat:cat  (git push origin <local_branch_name>:<remote_branch_name>)
```
- 拉取远程分支到本地
```
git fetch //同步一下仓库，可有可无，如果拉取不到远程分支，则是必要的
git checkout -b cat origin/cat //第一种
git fetch origin cat:cat  //第二种
```
- 本地分支与远程分支关联
```
git branch --set-upstre


am-to=origin/branch_name branch_name
```
- 查看当前的本地分支与远程分支的关联关系
```
git branch -vv
```
- 查看所有分支
```
git branch -a
git branch --all  
```
- 删除本地分支
```
git checkout master // 切换到master分支
git branch -d cat // 删除本地cat分支
// 删除远程分支
git push origin --delete branch_name
```

- 查看工作区，暂存区，版本库的区别
```
git status
//不如直接用idea的VCS->git->commit file的可视化界面来的清晰直观
```
- 查看历史版本
```
git log      // 看到所有提交的版本，但是 git reset删除的版本则看不到
git reflog    // 看到每一次提交的命令，包括reset的版本
git log –graph
//不如用idea的Version Control -> log -> 选择branch 界面来的清晰直观
```
- 回退版本
```
git log //查看提交记录
git reflog
git reset --hard HEAD^   //回退上一个版本
git reset –-hard HEAD^^   //回退上上一个版本
git reset –-hard 3628164   //回退到指定版本 后面的版本号可从上一条命令的log中得到
git revert HEAD   //  撤销前一次 commit
git revert HEAD^  // 撤销前前一次 commit
git revert abcsdsa   // 撤销指定commit id的版本
```
> reset和revert的区别，reset是将指针后移，删除了最新的提交版本，revert是指针前移，做了一次新的commit，只不过提交内容刚好和上一次commmit的相反。
> 参考<https://blog.csdn.net/yxlshk/article/details/79944535>
><https://www.cnblogs.com/0616--ataozhijia/p/3709917.html>
- 记录每一次的命令
```
git reflog
```
- 另存工作区的改动
```
git stash show   //查看另存区的内容
git stash list
git stash           //目前已改动的地方放入另存区
git stash pop    //取回另存区的改动,stash里的内容会被删除
git stash drop    // 删除stash的内容,不指定stash_id，则默认删除最新的存储进度。
git stash clear     //删除所有存储的进度。
参考<https://blog.csdn.net/daguanjia11/article/details/73810577>

// 进阶
git stash save 'message...'   // 添加注释
git stash pop --index     // 恢复最新的进度到工作区和暂存区。
git stash pop stash@{1}    // 恢复指定的进度到工作区。stash_id是通过git stash list命令得到的
git stash apply [–index] [stash_id]   // 除了不删除恢复的进度之外，其余和git stash pop 命令一样。
```
- 放弃本地修改
```
git checkout .   // 好像是新增的文件的不会删除，下面的命令就可以删除
git checkout . && git clean -xdf
```
- 本地仓库和远程仓库相关联
```
1. 远程新建个空仓库
2. 本地建好项目后，根目录下执行 
git init
git remote add origin git@github.com:SuperDavidM/test1.git
git remote -v
git pull origin master:master
git branch --set-upstream-to=origin/master master
git branch -vv
git pull 
git push
```

- 删除和远程仓库的关联
```
git remote -v
git remote rm origin
```

- 查看分支关系
```
gitk --simplify-by-decoration --all
// 或者idea里git窗口
// gitk如果没有安装，用 brew install git-gui 安装
```

- 回退版本后强制提交
```
git reset --hard HEAD^   // 回退
git reflog  //打印你记录你的每一次操作记录
git push -f origin <branch name>(分支名称); //强行将本地回退后的分支提交到远程分支；
git fetch --all // 别人提交或拉取代码可能会因为我们回退了版本而出现错误或者无法拉取，解决命令
```

- 修改最后一次提交的注释
  应该是还没push的情况下
```
git commit --amend
```
参考<https://blog.csdn.net/lxf0613050210/article/details/52525083>

- 本地分支重命名
```
git branch -m oldName newName
```
- 远程分支重命名
```
git branch -m oldName newName      //  本地重命名
git push --delete origin oldName   //   删除远程分支
git push origin newName            //    推送新名字的本地分支到远程
git branch --set-upstream-to origin/newName      //  建立连接
```
