

[toc]

# 配置git私钥

## 1.初始化自己的用户名和邮箱，命令如下。

```sh
git config --global user.name “输入你的用户名”
git config --global user.email “输入你的邮箱”
```

## 2.打开 bash，使用git命令。

```sh
# 生产密钥，命令如下 一路回车。
ssh-keygen -t rsa -C "7111479@qq.com"
（首先 ssh-keygen 会确认   
密钥的存储位置（默认是 .ssh/id_rsa），
然后它会要求你输入两次密钥口令。 
如果你不想在使用密钥时输入口令，
将其留空即可。 
然而，如果你使用了密码，那么请确保添加了 -o 选项，
它会以比默认格式更能抗暴力破解的格式保存私钥。 
你也可以用 ssh-agent 工具来避免每次都要输入密码）。
# 检查关联测试
ssh -T git@github.com
```

## 3.查看密钥 命令如下。

```sh
cd  ~/.ssh
# 输出密钥
cat id_rsa.pub
```

## 4.复制密钥

* 打开给gitlab,点击自己头像settings>>ssh Keys>>在key添加密钥。title设置自己的名称。

# Git 语法

```sh
# 开始一个工作区
cd directory					# 进入目录
git init      				# 创建一个空的 Git 仓库或重新初始化一个已存在的仓库
git add             	# 添加文件内容至索引
git mv        				# 移动或重命名一个文件、目录或符号链接
git estore   					# 恢复工作区文件
git rm        				# 从工作区和索引中删除文件
# 克隆仓库到本地
git clone 链接地址      # 把连接地址克隆仓库到当前新目录
# 检查历史和状态（参见：git help revisions）
git   bisect    通过二分查找定位引入 bug 的提交
git   diff      显示提交之间、提交和工作区之间等的差异
git   grep      输出和模式匹配的行
git   log       显示提交日志
git   show      显示各种类型的对象
git   status    显示工作区状态
# 扩展、标记和调校您的历史记录
git   branch    列出、创建或删除分支
git   commit    记录变更到仓库
git   merge     合并两个或更多开发历史
git   rebase    在另一个分支上重新应用提交
git   reset     重置当前 HEAD 到指定状态
git   switch    切换分支
git   tag       创建、列出、删除或校验一个 GPG 签名的标签对象
# 协同（参见：git help workflows）
git   fetch     从另外一个仓库下载对象和引用
git   pull      获取并整合另外的仓库或一个本地分支
git   push      更新远程引用和相关的对象

```

# 初始化仓库

```sh 
git init
```

# 设置签名 

* 配置文件  
  * 本地：.git/config 
  * 全局家目录下：.gitconfig

```sh
# 本地库范围签名 用户名
git config user.name 'itwuge'
# 本地库范围签名 邮箱
git config  user.email '7111479@qq.com'

# 全局库范围签名 用户名
git config --global user.name 'itwuge'
# 全局库范围签名 邮箱
git config --global user.email '7111479@qq.com'
```

# 显示工作区状态

```sh
# 查看缓存区情况 
git   status

# 位于分支 master 分支
on branch master

# 尚无提交
No commits yet

# 要提交的变更 
Changes to be submitted（使用 "git rm --cached <文件>..." 以取消暂存）

# 暂存区 
nothing to commit (使用 "git add <文件>..." 以包含要提交的内容)

```

# 添加 暂存区 中文件

```sh 
git add 要添加的内容
```

#  删除 暂存区中文件

```sh
git rm --cached  要移除的内容      				
```

# 查看历史操作

```sh 
# 显示全部操作
git log
# 以行显示历史内容
git log --pretty=oneline
# 显示最前面的短哈希值
git log --oneline 
# 显示可以后退的步数 HEAD@{移动到当前版本需要的步数}
git reflog
```

# 前进后退版本(找回文档 )

```sh
git reflog
# 索引操作
git reset --hard  最前面的哈希值
# ^ 操作  只能后退 一个 ^ 符号后退一步 
git reset --hard HEAD^ 
# ～操作  只能后退 ～后面跟要后退步数
git reset --hard HEAD～3
```

# 从缓存区提交到本地库

```sh
git commit 要提交到文件
# 编写这次提交的注解信息
git commit -m "提交的注解解信息" 要提交到文件
```

# 分支

```sh 
# 查看分支
git branch -v
# 创建分支
git branch 分支名
# 切换分支
git checkout 分支名
# 合并分支
# 1.切换到要合并到分支上
git checkout 要合并过来的分支名(被合并的分支)
# 2.合并分支
git merge 另一个分支的分支名(有新内容的分支)
```

# 推送到远程库

```sh 
# 查看远程库起别名
git remote -v
# 给远程库起别名 origin
git remote add wuge git@github.com:itwuge/Actions-OpenWrt.git
(fetch) 取回地址
(push)	推送地址
# 推送本地库到远程库
git push 远程库别名
```

# 克隆到本地库

```sh
# 把连接地址克隆仓库到当前新目录
git clone 链接地址     
```

### 基本使用

```sh
# 添加到缓存
git add .
# 查看缓存
git status
# 提交到本地
git commit -m "提交的注解解信息"
# 修改分支名
git branch -M 新分支名
# 给远程库起别名 origin
git remote add origin 远程仓库
# 提交到网络主分支
git push origin
# 提交到分支
git push -u origin main

```



