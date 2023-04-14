### 回退
在Git中，用HEAD表示当前版本，用HEAD^表示上一版本，用HEAD^^表示上上个版本，用HEAD~N表示上100个版本
```git
git reset --hard HEAD^

77a852  最新提交
88b9f2   上一次提交
#回到上一次提交,77a852成了未来版本，回到此版本需要commit id
git reset --hard 88b9f2 
git resest --hard 77a852  回到最新提交

如果回到了过去版本，git log 看不到未来版本的commit id怎么办？  
如果找不到commit id，Git提供了一个命令`git reflog`用来记录每一次命令
```

### 撤销修改
`git checkout -- file`可以丢弃工作区的修改。此操作会有两种情况：
*   `readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态
*   `readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态

**总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态**。

如果想把已经添加到 暂存区的修改回退到工作区(取消暂存)：
`(use "git restore --staged <file>..." to unstage)`
`git reset HEAD <file>`
如果想把工作区的修改撤销掉（discard):
`(use "git restore <file>..." to discard changes in working directory)`
`git checkout -- <file>`


### 删除文件
命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失**最近一次提交后你修改的内容**。

![[Git入门知识整理#移除文件]]
