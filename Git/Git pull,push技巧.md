### 应该怎样从Git服务端pull代码
* Do not use `git pull`
Because you get in one go "get the changes you don't know of from the central repo, and merge them with your work", **without a chance to review the changes between the two steps.**

generally run:
1. `git fetch`
2.  `git log --graph --oneline origin/master my/branch` (e.g : inspect the state of the remote branch I'm interested in)
3. run either `git rebase origin/master` or `git merge origin/master` (we happen to have a workflow which favors `rebase`, but anyways : I already have an idea of how complicated that action is going to be)

the difference with `git pull` is that at step 3, I can do :

-   merge or rebase an _intermediate_ commit of the remote branch , or an intermediate commit of my own branch,
-   cherry-pick a specific commit to see what mess it would introduce,
-   edit my branch _before_ rebasing/merging (one common case: drop that commit which does almost the same thing as the bugfix added on master)
* ...

当在两个不相关的分支下执行`git pull`时，会出现无关历史，无法合并的情况：
![[Git Errors#^4f6a22]]
例如：
* 假设远程仓库的branch是master, 本地默认的branch是main（只要两个不同即可），用户现在本地添加并提交了一些文件， 然后执行`git pull`命令[^1]，再`git checout -b master`, 创建并切换到本地master branch以后，再执行`git merge main`时就会出现
	```git
	fatal: refusing to merge unrelated histories
    # 拒绝合并无关历史
	```

Here are some common scenarios where `fatal: refusing to merge unrelated histories can occur.`

* You have a **new Git repository with some commits**. You then try to pull from an existing remote repo. The merge becomes incompatible because the histories for branch and remote pull are different. Git sees the situation as you trying to merge two completely unrelated branches, and it doesn’t know what to do.
* There’s something wrong with the .git directory. It may have been accidentally deleted at some point or got corrupted. This can happen if you’ve cloned or cleaned a project. Here the error occurs because Git doesn’t have the necessary information about your local project’s history.
* The branches are at different HEAD positions when you try to push or pull data from a remote repo and cannot be matched due to a lack of commonality.

TODO：
git pull 会自动设置跟踪分支吗（如果默认分支是一样的话）
**解答：** 不会，克隆指令会自动设置跟踪分支，但是`git init`,再`git remote add`,最后`git pull` 这个过程中不会自动设置跟踪分支。


[^1]:注意：此时只有 `origin/master` 的指针，并没有创建对应的master分支和创建对应的文件夹，后续使用`git checkout -b master`命令的时候才会创建master分支并从.git中恢复master的分支内容（可编辑的副本）
![[Git分支#^15e0dd]]



### git push出现的问题。

* 本地初始化了一个仓库，向不相关的远程仓库push，而且默认的分支不同（一个为main,一个为master）时，会要求先设置跟踪分支才能够提交成功，并且**这个操作会让远程仓库创建一个main分支。**
	```git
	> git push origin
    fatal: The current branch main has no upstream branch.
	To push the current branch and set the remote as upstream, use

 	  git push --set-upstream origin main
	# push失败，需要设置跟踪分支	
 
    ❯ git push --set-upstream origin main
	> git push -u origin main 
	# 可以成功 push  -u 是 --set-upstream的简写
	```


