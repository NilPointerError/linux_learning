#### git介绍

git是一个免费、开源的分布式版本控制工具。

git占空间小，具有本地库、暂存区和多个工作流分支等特点

git bash 这个工具的本质，它是在 Windows 操作系统里面，安装了一个虚拟的 Linux bash shell 程序，为 git 的命令行功能提供环境。所以，这个 git bash 其实是一个假的 bash，是在 Windows 里面模拟 bash shell 环境的虚拟 bash。第三，虚拟 bash 里面虚拟了一个 Linux 环境，它有一个虚拟的家目录，这个家目录就读取 Windows 操作系统配置的用户根路径，默认是C:\Users\{当前登录的Windows账户名}

分布式与集中式版本控制工具区别？

集中式版本控制工具：SVN等。具有一个单一的中央服务器

分布式：git等。可以在本地做代码版本控制

1.服务器断网后也可以进行开发

2.每个客户端保存到也都是整个完整的项目（包含历史记录，更加安全），可以用客户端的本地库对远程库进行恢复

#### git安装

#### git命令

```shell
git config --global user.name   xxx  #设置用户签名，与远程库无关
git config --global user.email    xxx  #设置用户签名
git init  #初始化本地库
git status #查看本地库状态
git add xxx #提交到暂存区（跟踪文件） 
git restore xxx #恢复文件到暂存区
git rm --cached <file>  #弹出暂存区（取消跟踪）
git commit -m "message" <file>  #提交暂存区的文件到本地库
git log/ git reflog#查看本地库提交日志
git reset --hard <idNo> #版本切换、回退（本地库）
```

#### git分支

 ```shell
 git branch -v #查看分支
 git branch xxx # 创建分支
 git checkout hot-fix #切换到分支hot-fix
 git merge hot-fix #合并hot-fix分支 冲突合并
 ```

当master分支也有修改时，master分支合并hot-fix分支时会出现冲突合并，这时需要我们手动合并分支，即找到冲突的文件手动修改保存后再提交，过程如下：

<img src="https://raw.githubusercontent.com/NilPointerError/MdImage/main/img/image-20240313135324047.png" width="600px" height="500px" />

#### 团队协作

- 团队内协作

  <img src="https://raw.githubusercontent.com/NilPointerError/MdImage/main/img/image-20240313141829186.png" width="700px" />

所在库点击settings邀请别人加入并分享链接：https://github.com/NilPointerError/linux_learning/invitations

- 跨团队协作

<img src="https://raw.githubusercontent.com/NilPointerError/MdImage/main/img/image-20240313141905442.png" alt="image-20240313141905442" width="700px" />



#### github

````shell
git remote -v #查看当前远程地址别名
git remote add origin https://github.com/NilPointerError/test.git #添加远程库
git push 远程地址别名 本地分支 #推送本地分支到远程库
git pull 远程地址别名 远程分支 #拉取
git clone  https://github.com/NilPointerError/test.git #克隆远程库到本地（不需要登录）
````

- ssh免密登录

ssh与https只是连接方式，没有添加ssh密钥时，git push需要进行网页登录，而添加ssh密钥可以实现免密登录，进行ssh连接比https连接也更安全。生成ssh密钥对后，私钥保存到本地，公钥添加到账户。（**此账户可以不同于远程库所在的账户，此账户用户名和密码可以不同于本地设置的用户签名）**

```shell
#首先进入用户的主目录中
cd ~
#配置免密登录命令
ssh-keygen -t rsa -C "zhangsan@qq.com"
#按三次回车，免密登录文件会保存到电脑的以下目录
/c/Users/{Username}/.ssh/id_rsa
#输入cat命令查看秘钥
cat ~/.ssh/id_rsa.pub
#复制秘钥内容到git添加自己的公钥
#用命令验证是否认证成功
ssh -T git@gitee.com
#如果有 You've successfully authenticated,则验证成功
```

#### idea集成git

创建忽略规则文件xxx.ignore（建议是git.ignore）

配置文件 .gitconfig

```
[user]
	name = NilPointerError
	email = gond_123@163.com
[http]
	proxy = socks5://127.0.0.1:7890
[core]
	excludesfile = C:/Users/Admin/git.ignore
```

idea设置git

<img src="https://raw.githubusercontent.com/NilPointerError/MdImage/main/img/image-20240313170458396.png" alt="image-20240313170458396" width="800px" />



reset：使用 reset 的方式直接把 HEAD 以及 MASTER 一起拉回那个版本，并且会丢失目标版本之后的内容，虽然可以通过 reflog 找到后面的版本，但是这种方式比较强烈的，可能会造成数据丢失

而 IDEA 中，使用的是 git checkout -b <branch-name> <commit> 的方式，这里 <commit> 指的是版本号；等再切回 master 的时候，它还会使用 git branch -d 的方式销毁它临时创建的分支



回滚应该用下面的reset，再选择reset模式（视频没有讲soft、mixed和hard的区别），按视频中的使用checkout进行回退是很危险的，其实checkout的本意是移动HEAD，只是后接分支名可以直接切换分支，接版本号可以直接让HEAD指向某版本，但是如果让HEAD直接指向一个版本，再进行提交应该是很危险的操作。

checkout就是看一眼, 如要切换版本并修改, 用reset(真 · 回滚, 删除版本之后的commit), 或者revert(重做, 保留所有commit)

#### gitlab

局域网代码托管平台

gitlab服务器搭建和部署

idea集成gitlab



**扩展：**

一般的工作中 git 操作流程是这样的:
你接到业务开发需求, 在自己的电脑上拉取服务器最新的 master 分支到本地 master 分支, 并创建本地 dev 分支, 下班前 add 和 commit 代码到本地 dev 分支下.
第二天上班后拉取服务器 master 分支的最新代码到本地 master 分支, 并将本地 master 分支合并到本地的 dev 分支上并解决冲突, 下班前 add 和 commit 代码到本地 dev 分支下.
重复此过程直到业务需求开发完成.
然后将开发完的本地 dev 分支代码合并到本地 master 分支, 本地 master 分支代码 push 到服务器 master 分支, 删除本地 dev 分支.
至此一个业务需求开发 git 操作流程完成.



分支管理

版本管理