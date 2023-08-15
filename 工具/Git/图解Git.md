这一篇更像是图形化的速记表。
### Git的基本用法
---
![[add_commit_contrary.png]]

上面的四条命令在工作目录、暂存目录(也叫做索引)和仓库之间复制文件。

- `git add _files_` 把当前文件放入暂存区域。
- `git commit` 给暂存区域生成快照并提交。
- `git reset -- _files_` 用来撤销最后一次`git add _files_`，你也可以用`git reset` 撤销所有暂存区域文件。
- `git checkout -- _files_` 把文件从暂存区域复制到工作目录，用来丢弃本地修改。

你可以用 `git reset -p`, `git checkout -p`, or `git add -p`进入交互模式。

![[directcommit_contrary.png]]

- `git commit -a` 相当于运行 git add 把所有当前目录下的文件加入暂存区域再运行。git commit.
- `git commit _files_` 进行一次包含最后一次提交加上工作目录中文件快照的提交。并且文件被添加到暂存区域。
- `git checkout HEAD -- _files_` 回滚到复制最后一次提交。
[[Git重置#带路径]]
### 约定
---
![[article_convention.png]]
绿色的5位字符表示提交的ID，分别指向父节点。分支用橘色显示，分别指向特定的提交。当前分支由附在其上的_HEAD_标识。 这张图片里显示最后5次提交，_ed489_ 是最新提交。 _main_ 分支指向此次提交，另一个 _stable_ 分支指向祖父提交节点。

### 命令详解
---
#### diff
![[diff.png]]

💡我发现只要加了 --cached 就和 工作目录没有什么关系

其他的基本都是和工作目录的比较， 除了两个分支之间的比较和 索引肯HEAD的比较

#### commit
没什么好看的， 很常见

#### checkout
checkout命令
- 用于从历史提交（或者暂存区域）中拷贝文件到工作目录，
- 也可用于切换分支。

当给定某个文件名（或者打开-p选项，或者文件名和-p选项同时打开）时，git会**从指定的提交中拷贝文件到暂存区域和工作目录**。比如，`git checkout HEAD~ foo.c`会将提交节点 _HEAD~_ (即当前提交节点的父节点)中的`foo.c`复制到工作目录并且加到暂存区域中。

如果命令中**没有指定提交节点**，则会从暂存区域中拷贝内容。注意当前分支不会发生变化。

```bash
git checkout <file>   # 从暂存区获取
```

⚠️ checkout 命令会将文件复制到**工作目录**和**暂存区**， 所以小心使用， 可能会覆盖你的改动。

##### 不指定文件名, 指定分支
当**不指定文件名，而是给出一个（本地）分支时**，那么 _HEAD_ 标识会移动到那个分支（也就是说，我们“切换”到那个分支了），然后暂存区域和工作目录中的内容会和 _HEAD_ 对应的提交节点一致。新提交节点（下图中的a47c3）中的**所有文件都会被复制（到暂存区域和工作目录中）**；只存在于老的提交节点（ed489）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

![[checkout.png]]

##### 既不指定文件也不指定分支( 分离 HEAD )
果既没有指定文件名，也没有指定分支名，而是一个
- 标签
- 远程分支
- SHA-1值
- 或者是像 _main~3_ 类似的东西

就得到一个**匿名分支**，称作 _detached HEAD_ （被分离的 _HEAD_ 标识）。这样可以很方便地在历史版本之间互相切换。比如说你想要编译1.6.6.1版本的git，你可以运行`git checkout v1.6.6.1`（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说`git checkout main`。然而，当提交操作涉及到“分离的HEAD”时，其行为会略有不同，详情见在[下面](https://marklodato.github.io/visual-git-guide/index-zh-cn.html#detached)。

💡 最大的优点是：  方便的在历史版本中切换

![[checkout_nofile_nobranch.png]]

#### HEAD标识处于分离状态时的提交操作
当_HEAD_处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。(你可以认为这是在**更新一个匿名分支**。)
[[分离指针操作]]

![[detach_HEAD.png]]

一旦此后你切换到别的分支，比如说 _main_ ，那么这个提交节点（可能）再也不会被引用到，然后就会被丢弃掉了。注意这个命令之后就不会有东西引用 _2eecb_。

![[detach_checkout_main.png]]

但是，如果你想保存这个状态，可以用命令`git checkout -b _name_`来创建一个新的分支。

![[detach_HEAD_get_name.png]]

#### Reset
reset命令把
- **当前分支指向另一个位置**，并且有选择的变动工作目录和索引。
- 也用来在**从历史仓库中复制文件到索引，而不动工作目录**。(与 checkout 不同)

如果不给选项，那么当前分支指向到那个提交。如果用`--hard`选项，那么工作目录也更新，如果用`--soft`选项，那么都不变。

![[git_reset.png]]

如果没有给出提交点的版本号，那么默认用_HEAD_。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用`--hard`选项，工作目录也同样。

![[git_reset_hard.png]]

如果给了文件名(或者 `-p`选项), 那么工作效果和带文件名的[checkout](https://marklodato.github.io/visual-git-guide/index-zh-cn.html#checkout)差不多，除了索引被更新。

![[git_reset_file.png]]
[[Git重置]]

#### Merge
merge 命令把不同分支合并起来。合并前，索引必须和当前提交相同。
- 如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。 
- 另一种情况是如果当前提交是另一个分支的祖父节点，就导致 _fast-forward_ 合并。指向只是简单的移动，并生成一个新的提交。

图略， 很简单。


否则就是一次**真正的合并**。默认把当前提交(_ed489_ 如下所示)和另一个提交(_33104_)以及他们的共同祖父节点(_b325c_)进行一次[三方合并](http://en.wikipedia.org/wiki/Three-way_merge)。结果是先保存当前目录和索引，然后和父节点 _33104_ 一起做一次新提交。

![[git_merge.png]]

#### Cherry Pick
cherry-pick命令"复制"一个提交节点并在当前分支做一次完全一样的新提交。

![[git_cherry_pick.png]]

#### Rebase
**衍合(rebase, 变基)** 是合并命令的另一种选择。

合并把两个父分支合并进行一次提交，提交历史不是线性的。

**衍合(rebase)** 在当前分支上重演另一个分支的历史，提交历史是线性的。 

rebase 本质上，这是线性化的自动的 cherry-pick

上面的命令都在 _topic_ 分支中进行，而不是 _main_ 分支，在 _main_ 分支上**重演**，并且把分支指向新的节点。注意旧提交没有被引用，将被回收。

要限制回滚范围，使用`--onto`选项。下面的命令在 _main_ 分支上重演当前分支从 _169a6_ 以来的最近几个提交，即 _2c33a_。

![[git_rebase.png]]

同样有`git rebase --interactive`让你更方便的完成一些复杂操作，比如丢弃、重排、修改、合并提交。没有图片体现这些，细节看这里:[git-rebase(1)](http://www.kernel.org/pub/software/scm/git/docs/git-rebase.html#_interactive_mode)




