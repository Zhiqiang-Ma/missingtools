## 三，git 操作

### 1，仓库管理

* 代码仓的四种状态

  * Untracked： 未跟踪。此文件在文件夹中，但是没有加入到git库，不参与版本控制。可以通过 git add 变为staged 状态。
  * Unmodify： 文件已经入库，但没有修改，即版本库中的文件快照内容和文件夹中的一致。这类文件如果被修改，变为modified，如果使用 git rm --cached file，则会移除版本库，变为untracked 状态。
  * Modified：文件已经被修改，并没有进行其他操作。此文件两种去处，通过 git add 加入暂存staged 状态，使用 git checkout 丢弃修改返回到unmodify 状态。git checkout filename 即从库中取出文件，覆盖当前的修改。
  * staged：暂存状态。执行git commit 将修改同步到库中，此时库中文件和本地文件变为一致。文件变为unimodify 状态。执行 git reset HEAD filename 取消暂存，文件变为modified 状态。

* 代码仓分类

  * workspace：工作区。存放项目代码的目录
  * index/stage：暂存区。用于临时存放对文件的修改，它只是一个文件，保存即将提交到文件列表信息。
  * repository：仓库区，即版本库。就是安全存放数据的位置，有提交到所有版本的数据。其中HEAD指向最新放入仓库的版本。
  * remote：远程仓库，托管代码的服务器。需要使用push 和pull 同步本地代码和远程代码。

  PS：本地仓库和远程的仓库共同维护commit ID，需要使用 push 和pull  将本地仓库和远程的仓库

**注意：在修改代码前，一定要先同步远程分支，再修改**

### 2，remote 远程仓库链接

* 查看本地和远程分支对应关系

~~~shell
git remote -v
~~~

* 添加远程链接

~~~shell
git remote add <name> <url>
~~~

此命令和直接的配置/.git/config 是同样的效果。

思考： 这里的名字到底意味着什么？ origin 并不是指得是远程的仓库，而是指得是远程仓库在本地的一个指针

* 删除远程仓库和重新命名

~~~shell
git remote rm <name>
git remote rename <old-name> <new-name>
~~~

删除 name

### 3，clone 克隆仓库

* -b 克隆指定分支

~~~shell
git clone -b develop url
~~~

* 克隆后重命名

~~~shell
git clone -b develop url new_name
~~~

* --depth 浅克隆

~~~shell
git clone -b develop --depth 1 url new_name
~~~

### 4，add 添加到暂存状态

* 基本使用

~~~shell
git add .
~~~

* reset 撤销，回到修改状态

~~~shell
git reset HEAD	# 回退当前暂存的所有文件
~~~

* add 错误文件，回退指定文件

~~~shell
git reset files	# 回退指定文件
~~~

### 5，reset 代码回退

将代码回退到指定的commit ID，有三种的方式

* soft 

~~~shell
git reset --soft commit ID
~~~

将指定commit id 撤回之后所有的内容放入暂存区。即git add 之后的状态，但是workspace，并没有变化。

使用场景为：合并两次commit提交。第一次提交之后，再次提交，合并两个commit。相当于撤回了第一次的commit，以第二次提交的commit 为准，因为soft 并不改变workspzce 的代码。

* mixed

~~~shell
git reset --mixed commit ID
~~~

将指定 commit id 撤回之后所有内容全部放进工作区中。即git add 之前的状态，但是workspace 并没有变化.

* hard

~~~shell
git reset --hard commit ID
~~~

使用场景：本地代码丢弃，使用版本库的代码为基准开发。将指定 commit id 撤回并清空工作目录及暂存区所有修改。

### 6， push 推送本地分支

push 只将 git add 后提交到本地仓库中的可追踪文件，推送到远程版本仓库中。未被追踪（红色）的本地workspace 文件，不能被push推送到远程版本库

新建分支并推送到远程仓库

~~~shell
git checkout -b dev-person1
git push origin dev-person1
~~~

远程版本仓版本比本地仓库提前，此时git push 报错！ 可以使用-f 强制，可以将本地的版本信息同步到远程版本仓。

~~~shell
git push --force --set-upstream origin dev-name
~~~

### 7，merge 合并分支

合并步骤：

* 进入要合并的分支（如开发分支dev 合并到master，则进入master目录）

~~~shell
git checkout master
git pull
~~~

* 查看所有分支是否都pull下来了

~~~shell
git branch -a
~~~

* 使用merge合并开发分支

~~~shell
git merge 分支名
~~~

* 有冲突的话，通过IDE解决冲突

* 解决冲突之后，将冲突文件提交暂存区

~~~shell
git add 冲突文件
~~~

* 提交merge之后的结果

~~~shell
git commit -m "merge develop"
~~~

如果不是使用git commit -m “备注” ，那么git会自动将合并的结果作为备注，提交本地仓库；

* 本地仓库代码提交远程仓库

~~~shell
git push
~~~

注意：退出nohao 编辑器是ctrl+x ，退出编辑，并回车选择保存的文件

### 8，pull 更新代码

查看本地分支和远程分支对应关系

~~~shell
git branch -vv
~~~

* 方式一：将远程的代码先拉取到本地的临时分支 tmp，然后手动的进行合并merge。掌控代码

~~~shell
git branch -b tmp origin/tmp // 基于远程的分支新建本地分支
git pull origin dev-person:tmp
git checkout tmp
git diff tmp
git merge tmp
# 如果有冲突，合并冲突后，再git add ，git commit
~~~

有冲突，利用IDEA 合并后，git add, git commit 进行提交。

思考，能否不新建分支，直接的拉下来，因为新建分支是基于当前分支克隆的。

* 方式二：使用pull 直接与本地代码merge

~~~shell
git pull origin dev-person:dev-person
~~~

只有pull 之后才能把别人更新的代码和log 同步到workspace。

注意pull 是直接的使用远程的分值代码，覆盖了本地 workspace 的代码。这种情况只适用于在别人的基础上进行修改。

~~~shell
# 强行的覆盖掉本地的代码
git pull --force  <远程主机名> <远程分支名>:<本地分支名> 
git pull --rebase
~~~

### 9， log 查看日志

* 以一行的形式查看

~~~shell
git log --oneline
~~~

* 以树状形式显示

~~~shell
git log --all --graph --decorate --oneline
~~~

### 10，diff 查看不同改动

* 比较当前所处分支的改动（即HEAD指向的分支），修改了那些内容

~~~shell
git diff file_name
~~~

* 指定hash code，与具体的一次commit 提交进行比较

~~~shell
git diff hash code file_name
~~~

### 11，branch 分支操作

* 查看本地分支和远程分支的对应关系

~~~shell
git branch -vv
~~~

* 设置本地与远程分支对应关系

~~~shell
git branch --set-upstream-to origin/分支名
~~~

* 创建分支

~~~shell
git checkout -b dev				// 基于本地创建分支
git checkout -b dev origin/dev  // 基于远程分支创建本地分支
~~~

参数 -b 命令相当于

~~~shell
git branch dev 		// 创建分支
git checkout dev	// 切换分支
~~~

* 删除分支

~~~shell
git branch -D dev						// 删除本地分支
git push origin --delete <branchName>	// 删除远程分支
~~~

* 重命名分支

~~~shell
git branch -m new-name
git branch -m old-name new-name
~~~

### 12，stash 暂存当前工作

stash 命令可以将当前未提交到本地和服务器的代码推入到git 的栈中，此时工作空间workspace 和上一次提交的内容是完全一致的，然后可以进行 checkout 等其他操作。过后可以使用pop 将之前暂存的工作应用回来。即git stash 会把所有未提交的修改（包括暂存和非暂存的）都保存起来，用于后续回复当前的工作目录。

~~~shell
# 建议加上一个消息
git stash save "test-cmd-stash"
git stash pop
~~~

### 13，git 免密登录

~~~shell
# 记录密码在当前的仓库中
git config credential.helper store
# 全局的保存
git config --global credential.helper store
~~~

### 14，rm cached 删除分支文件

**使用情景**：误上传了数据文件或是其他无用的文件夹，可以使用以下的命令，将远程分支上的文件删除，而保留本地文件。

 --cached 的作用：将文件仅仅从索引中移除，就是之前已经提交，现在将此目录释放掉，恢复到工作区，未被追踪的状态，即 git add . 命令之前的状态

完成 rm 后仅仅是在本地的仓库中进行了记录，需要使用push 同步到远程的仓库。

~~~shell
git rm --cached -r useless #文件路径，删除文件夹需要加上-r
git commit -m "remove directory from remote repository"
git push
~~~

将不需要上传的文件夹加入到 .gitignore 中

### 15，amend 修改上次提交信息

commit 提交代码后，发现提交的描述信息有误，或是不想保留上一次的提交信息，可以使用 commit 中的amend 参数进行修改

~~~shell
git commit --amend 
~~~

### 16，commit 提交信息规范

message 信息的格式

~~~shell
<type>(<scope>):<subject> 
~~~

* type （必选）用于说明 commit 的类别，只允许使用下面的标识
  * feat：新功能
  * fix/to ：修复bug。to 使用与多次提交，最终修复时候使用fix
  * docs：文档。对文档的修改和添加
  * style：格式。不影响代码运行的改动
  * refactor：重构。既不是新增的功能，也不是代码修改bug。
  * perf：优化相关，比如性能提升、体验提升。
  * test：增加测试
  * chore：构建过程或是辅助工具的变动
  * merge：代码合并。
  * sync：同步主线或是分支的bug。

* scope （可选）说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
  * 根据具体的模块进行定义
  * 比如 location，browser，compile，compile，rootScope
* subject （必选）是commit目的的简短描述，不超过50个字符
  * 结尾不加句号或其他标点符号

总结例子：

~~~shell
fix (docoder):修改解码类逻辑，产生内存溢出
feat(Controller):用户查询接口开发
~~~

### 17，git fetch

git fetch命令用于从远程获取代码库。之后再实用git merge 进行合并操作

~~~shell
git fetch <远程主机名> //这个命令将某个远程主机的更新全部取回本地
~~~

如果只想取回特定分支的更新，可以指定分支名

~~~shell
git fetch <远程主机名> <分支名> //注意之间有空格
~~~

取回更新后，会返回一个`FETCH_HEAD` ，指的是某个branch在服务器上的最新状态，我们可以在本地通过它查看刚取回的更新信息：

~~~shell
git log -p FETCH_HEAD 
~~~

之后合并到本地的代码仓，需要实用merge 命令

~~~shell
git merge FETCH_HEAD
~~~

详细的过程理解：

使用git fetch更新代码，本地的库中master的commitID不变，还是等于1。但是与git上面关联的那个orign/master的commit ID变成了2。这时候我们本地相当于存储了两个代码的版本号，我们还要通过merge去合并这两个不同的代码版本，如果这两个版本都修改了同一处的代码，这时候merge就会出现冲突，然后我们解决冲突之后就生成了一个新的代码版本。这时候本地的代码版本可能就变成了commit ID=3，即生成了一个新的代码版本。

**问题：fetch 后不能显示所有的分支**

这个配置表示将远程仓库 origin 的所有分支的代码拉取到本地，并将其存储在本地的 origin/ 前缀的远程分支上

~~~shell
git config remote.origin.fetch +refs/heads/*:refs/remotes/origin/*		  // 所有的分支
git config remote.origin.fetch +refs/heads/*:refs/remotes/origin/develop  // 只拉取develop 分支
~~~

这个配置可以在 .git/config 文件中找到，也可以使用 git config 命令来查看和修改

参考：

~~~ 
https://devconnected.com/how-to-set-upstream-branch-on-git/#:~:text=You%20can%20check%20tracking%20branches%20by%20running%20the,yet%20%28and%20no%20upstream%20branches%20as%20a%20consequence%29
~~~

### 18，biset

git bisect命令使用二分搜索算法来查找提交历史中的哪一次提交引入了错误

使用git bisect二分法定位问题的基本步骤：

1. git bisect start [最近的出错的commitid] [较远的正确的commitid]
2. 测试相应的功能
3. git bisect good 标记正确
4. 直到出现问题则 标记错误 git bisect bad
5. 提示的commitid就是导致问题的那次提交

### 19，git reflog 参考日志

reflog 是 Git 操作的一道安全保障，它能够记录几乎所有本地仓库的改变。包括所有分支 commit 提交，已经删除（其实并未被实际删除）commit 都会被记录。总结而言，只要 HEAD 发生变化，就可以通过 reflog 查看到。

### 21，git tag

它允许用户在特定的提交上添加标签，以便于标识和引用这些提交

* 创建标签

使用 git tag 命令可以创建一个新的标签。例如，git tag v1.0 会在当前所在的提交上创建一个名为 “v1.0” 的标签。你也可以在创建标签时指定提交，如 

~~~shell
git tag v1.0 <commit-id>
~~~

* 查看标签

使用 git tag 命令可以查看所有已创建的标签。如果你想要查看特定标签的详细信息，可以使用 

~~~shell
git show-ref --tags
~~~

并结合 grep 命令进行筛选

* 推送标签

默认情况下，当你推送分支到远程仓库时，标签并不会被自动推送。你需要显式地推送标签。

使用 

~~~shell
git push origin <tagname>
~~~

命令可以推送单个标签到远程仓库，如 git push origin v1.0。如果你想要一次性推送所有本地标签到远程仓库，可以使用

~~~shell
git push origin --tags
~~~

* 删除标签 

如果你需要删除本地或远程仓库中的标签，可以使用

~~~shell
 git tag -d \<tagname> 
~~~

命令删除本地标签，如 git tag -d v1.0。要删除远程仓库中的标签，你需要先删除本地标签，然后使用

~~~shell
 git push origin :refs/tags/<tagname>
 git push origin :refs/tags/v1.0
~~~

* 检出标签 

如果你想要检出某个标签对应的提交，可以使用 

~~~shell
git checkout <tagname> 
git checkout v1.0
~~~

这会将你的工作目录切换到该标签对应的提交状态。

需要注意的是，标签在 Git 中是不可变的，一旦创建就无法修改

### 22，git rebase

主要用于重新应用在一个分支上所做的提交于另一个分支。这个操作的目标通常是让提交历史更加整洁，或者将特性分支的更改整合到主分支中。

git rebase 实际上会取出系列的提交记录，“复制”它们，然后在另外一个地方逐个地放下。
与 git merge 不同，git rebase 不会创建一个新的合并提交，而是将提交线性化地添加到目标分支上。

步骤：

* 切换到要进行 rebase 的分支

* 运行 git rebase <base_branch>，其中 <base_branch> 是你想要基于的分支、

* 如果在 rebase 过程中出现冲突，Git 会停止并允许你解决冲突。解决冲突后，使用 git add 更新解决的内容，然后运行 git rebase --continue 继续 rebase 过程。如果在 rebase 过程中你决定放弃，可以运行 git rebase --abort 来恢复到 rebase 开始前的状态。

注意：

git rebase 会改变提交历史，因此只在你确信没有其他人正在使用你即将 rebase 的分支时才应该使用

### 23， git diff 

* git diff

当工作区有改动，暂存区为空时，`git diff` 对比的是工作区与最后一次 commit 提交的仓库的共同文件。

当工作区有改动，暂存区不为空时，`git diff` 对比的是工作区与暂存区的共同文件。这个命令会显示尚未暂存（即还未通过 `git add` 命令添加到暂存区）的改动。

* git diff –cached 或 git diff –staged：

这两个命令是等价的，用于显示暂存区（已 add 但未 commit 文件）和最后一次 commit（HEAD）之间的所有不相同文件的增删改。换句话说，它们显示了你已经暂存、但还未提交的改动。

* git diff HEAD 

这个命令显示工作目录（已 track 但未 add 文件）和暂存区（已 add 但未 commit 文件）与最后一次 commit 之间的所有不相同文件的增删改。这实际上是一个综合比较，包括了还未暂存和已经暂存但还未提交的改动。

### 20，git 实用命令

~~~shell
git add -p hello.txt
git blame
git stash
git stash pop
git biset
.gitignore
~~~

### 24，reference

git知识资源

~~~shell
https://www.atlassian.com/git/tutorials/syncing
~~~

