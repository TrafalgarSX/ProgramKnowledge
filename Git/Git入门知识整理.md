## Git是什么？
Git直接记录快照，而不是比较差异
Git把数据看作是对小型文件系统的一系列快照。每当提交更新或保存项目状态时，**Git会对当时的全部文件创建一个快照,并保存这个快照的索引**。如果文件没有修改，Git不再重新存储该文件，而是保留一个链接只想之前存储的文件。

### 三种状态
- **已修改**表示修改了文件，但还没保存到数据库中。
- **已暂存**表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
- **已提交**表示数据已经安全地保存在本地数据库中。

![[Pasted image 20230302210246.png]]
## 配置Git
* git config命令
`git config` 的配置文件有系统文件级别、用户级别和当前仓库级别。一般情况只需要使用针对当前用户的配置文件，其文件路径在 ~/.gitconfig(git config --global配置)

### 用户信息
安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改：
```git
git config --global user.name "Trafalgar"
git config --global user.email "guoyawen98@gmail.com"
```
* 配置文本编辑器
```git
git config --global core.editor code
```
* 检查配置信息
```git
git config --list
```
* 获得帮助信息
```git
git help <verb>
git <verb> --help
```

## Git基础
本章会学习配置并初始化一个仓库、开始或停止跟踪文件、暂存或提交更改。
本章也将向你演示了如何配置 Git 来**忽略指定的文件和文件模式**、如何**迅速而简单地撤销错误操作**、如何**浏览你的项目的历史版本以及不同提交（commits）之间的差异**、如何向你的远程仓库推送（push）以及如何从你的远程仓库拉取（pull）文件。

### 记录更新到到仓库

#### 获取Git仓库
1. git init 命令将当前文件夹初始化为Git仓库
2. git clone url 命令从其他服务器克隆一个已存在的Git仓库

在克隆远程仓库的时候，可以通过额外的参数指定新的目录名，自定义本地仓库的名字
```git
git clone <url> myfilename
```

#### 检查当前文件状态
可以用 `git status` 命令查看哪些文件处于什么状态。

#### 跟踪新文件
使用命令`git add`开始跟踪一个文件。
```git
git add README
```
此时，README文件是已暂存状态

#### 暂存已修改的文件
`git add` 命令是个多功能命令：
* 可以用它开始跟踪新文件
* 或者把已跟踪的文件放到暂存区
* 还能用于合并时把有冲突的文件标记为已解决状态等。
将这个命令理解为“**精确地将内容添加到下一次提交中**”而不是“将一个文件添加到项目中”要更加合适。

#### 状态简介
`git status` 命令的输出十分详细，但其用语有些繁琐。如果你使用 `git status -s` 命令或 `git status --short` 命令，你将得到一种格式更为紧凑的输出。
```git
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```
输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。
例如： `Rakefile` 文件已修改，暂存后又作了修改，因此该文件的修改中既有已暂存的部分，又有未暂存的部分。

#### 查看已暂存和未暂存的修改
如果想知道具体修改了什么地方，可以用`git diff`命令。
`git diff` 能通过文件补丁的格式更加具体地显示哪些行发生了改变。
此命令比较的是**工作目录中当前文件和暂存区域快照之间的差异**。 也就是修改之后还没有暂存起来的变化内容。

要查看**已暂存的将要添加到下次提交里的内容**，可以用 `git diff --staged`命令。 这条命令将比对已暂存文件与最后一次提交的文件差异(--stage 和 --cached是同义词)

#### 提交更新
`git commit`命令用来提交暂存区中的快照。
```git
git commit -m "提交信息"   #一行内提交
或者
git commit
进入编辑器编写提交信息

#可以跳过暂存，直接提交
git commit -a -m "提交信息"
```

#### 移除文件
要从 Git 中移除某个文件，**就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交**。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。
```git
git rm file_you_want_to_remove.ext  #将文件从工作区和暂存区删除
```
下一次提交时，该文件就不再纳入版本管理了。 

**如果要删除之前修改过或已经放到暂存区的文件**，则必须使用强制删除选项 `-f`（译注：即 force 的首字母）。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这样的数据不能被 Git 恢复。
```git
# 删除之前修改过或者修改过并放到了暂存区的文件
git rm -f <file>
```

如果想把文件**从暂存区域移除，但仍然希望保留在当前工作目录中**，换句话说，仅是从跟踪清单中删除，使用 --cached 选项即可：
```git
git rm --cached <file>
```

#### 移动文件
要在Git中对文件改名，可以这么做：
```git
git mv file_from file_to # Git可以检测到这是一个重命名操l作

#实际上，运行git mv 就相当于运行了下面三条命令。
mv file_from file_to
git rm file_from
git add file_to

如果手动或者使用其他工具重命名文件时，记得在提交前git rm删除旧文件名，再git add添加新文件名
```

### 查看提交历史
不传入任何参数的默认情况下，`git log` 会按时间先后顺序列出所有的提交，最近的更新排在最上面。
`git log` 有许多选项可以帮助你搜寻你所要找的提交：
1.比较有用的选项是 `-p` 或 `--patch` ，它会显示每次提交所引入的差异（按 **补丁** 的格式输出）。也可以限制显示的日志条目数量，例如使用 -2 选项来只显示最近的两次提交:
```git
git log -p -2
```
2.如果想看到每次提交的简略统计信息
```git
git log --stat
something like will be output
commit hashcode\n Author: Name<mail>\n   Date:<time>
comment infomation
 README           |  6 ++++++
 Rakefile         | 23 +++++++++++++++++++++++
 lib/simplegit.rb | 25 +++++++++++++++++++++++++
 3 files changed, 54 insertions(+)
```
3.`--pretty`选项使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项供你使用。 比如 oneline 会将每个提交放在一行显示，在浏览大量的提交时非常有用。 另外还有 short，full 和 fuller 选项，它们展示信息的格式基本一致，但是详尽程度不一：(可以和`--graph`结合使用)
```git
git log --pretty=oneline # only show hashcode and comment infomation
# 可以定制记录的显示格式
git log --pretty=format:"%h - %an, %ar : %s"
hashcode - author, change time: comment infomation
```

`作者`和`提交者`不是相同的概念

#### 限制输出长度
类似 `--since` 和 `--until` 这种按照时间作限制的选项很有用。 例如，下面的命令会列出最近两周的所有提交：
```git
git log --since=2.weeks
```
另一个非常有用的过滤器是 `-S`， 它接受一个字符串参数，并且只会显示那些添加或删除了该字符串的提交。 假设你想找出添加或删除了对某一个特定函数的引用的提交，可以调用：
```git
git log -S function_name
```

### 撤销操作（重要）
有时候我们**提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了**。 此时，可以运行带有 `--amend` 选项的提交命令来重新提交：
```git
git commit --amend

这个命令会将暂存区中的文件提交。如果上次提交以来还没有做任何修改（例如，上次提交后，马上执行了此命令），快照会保持不变，只修改提交信息（覆盖上次的提交信息）。

git commit -m "first commit"
git add forgotten_file
git commit --amend

最终只会有一个提交，第二个提交会代替第一次提交的结果
```

#### 取消暂存的文件
`git status`命令和一些修改文件状态的命令，会提示如何撤销操作。
```git
# 此命令可以取消已暂存的file
gti reset HEAD <file>
```

* git reset 确实是个危险的命令，如果加上了 --hard 选项则更是如此。 然而在上述场景中，**工作目录中的文件尚未修改，因此相对安全一些。**

#### 撤销对文件的修改
如果你并不想保留对 CONTRIBUTING.md 文件的修改怎么办？ 你该如何方便地撤消修改——将它还原成上次提交时的样子
```git
# this commond will discard changes in working directory
git checkout -- <file>

这个命令是危险的命令，<file> 文件在本地的任何修改都会消失，Git会用最近提交的版本覆盖它
```

### 远程仓库的使用

#### 查看远程仓库
如果想查看你已经配置的远程仓库服务器，可以运行 `git remote` 命令。 它会列出你指定的每一个远程服务器的简写。
也可以指定选项 `-v`，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。
```git
git remote -v
output like this:
bakkdoor  https://github.com/bakkdoor/grit (fetch)
bakkdoor  https://github.com/bakkdoor/grit (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
defunkt   https://github.com/defunkt/grit (fetch)
defunkt   https://github.com/defunkt/grit (push)
```

#### 添加远程仓库
```git
git remote add <shortname> <url>

shortname 用于后续在命令行中代替整个URL
例如：
git fetch shortname
```

#### 从远程仓库抓取与拉取
```git
git fetch <remote>
```
* 注意 `git fetch` 命令只会将数据下载到你的本地仓库——**它并不会自动合并或修改你当前的工作。** 当准备好时你必须手动将其合并入你的工作。
如果想自动抓取后合并远程分支到当前分支，应当使用`git pull`命令（前提：当前分支设置了跟踪远程分支 **不明白什么是跟踪远程分支**）

`git clone`命令会自动设置本地master分支跟踪克隆的远程仓库的master分支（当前默认分支）

#### 推送到远程仓库
```git
git push <remote> <branch>
例如：
git push origin master
```
只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 **你必须先抓取他们的工作并将其合并进你的工作后才能推送。(一般先pull 再push)**

#### 查看远程仓库
如果想查看一个远程仓库的更多信息：
```git
git remote show <remote>
```

#### 远程仓库的重命名与移除
重命名仓库：
```git
git remote rename origin guoyawen
git clone命令下来的 <remote>默认为origin,
```
移除远程仓库
```git
git remote remove <remote>
```


### 打标签

#### 列出标签
```git
git tag

git tag -l

git tag -l "tag_name"  通配模式下，-l或者--list 强制要求使用
```

#### 创建标签
* 轻量标签(lightweight)
* 附注标签(annotated)
**轻量标签**很像一个不会改变的分支——它只是某个特定提交的引用。
**附注标签**是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard （GPG）签名并验证。 **通常会建议创建附注标签**

#### 附注标签
在运行tag命令时指定-a选项：
```git
git tag -a v1.4 -m "my version 1.4"  # -m 指定一条存储在标签中的信息

git show  v1.4
此命令可以看到标签信息与之对应的提交信息
```

#### 轻量标签
 轻量标签本质上是将提交校验和存储到一个文件中——没有保存任何其他信息。
 ```git
 # 不需要使用 -a -S 或者 -m选项
 git tag v1.4-lw
```

#### 后期打标签
```git
git tag -a v1.2  9fceb92      # 最后为 部分校验和(校验和也可)
```

#### 共享标签
默认情况下，git push 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须**显式地推送标签到共享服务器上**。 这个过程就像共享远程分支一样——你可以运行 `git push origin <tagname>`。

#### 删除标签
```git
git tag -d v1.4-lw
```
注意上述命令并**不会从任何远程仓库中移除这个标签**，你必须用 `git push <remote> :refs/tags/<tagname>` 来更新你的远程仓库
上面这种操作的含义是，将冒号前面的空值推送到远程标签名，从而高效地删除它。

第二种方法:
```git
git push origin --delete <tagname>
```

#### 检出标签（感觉常用）
切换到某个标签指向的文件版本。
```git
git checkout 2.0.0
```
`git checkout`会使你的仓库处于“分离头指针（detached HEAD）”的状态——这个状态有些不好的副作用

在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。 因此，如果你需要进行更改，比如你要修复旧版本中的错误，那么通常需要创建一个新分支:
```git
git checkout -b version2 v2.0.0
```

### Git别名
设置完整命令的别名
```git
git config --global alias.<alias_name>   commond

例如
git config --global alias.co checkout
```