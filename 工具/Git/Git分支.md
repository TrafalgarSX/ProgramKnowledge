### 分支简介
在进行提交操作时，Git 会保存一个提交对象（commit object）。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象， 而由多个分支合并产生的提交对象有多个父对象。

#### 分支操作的对象
当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和， 然后在 Git 仓库中这些校验和保存为**树对象**。随后，Git 便会创建一个**提交对象**， 它除了包含作者的姓名，邮箱，comment info以及指向提交对象父对象的指针，还包含指向这个树对象（项目根目录）的指针。 如此一来，Git 就可以在需要的时候重现此次保存的快照。

```txt
type blog = array<byte>
type tree = map<string, tree | blob>
type commit = struct{
	parents: array<commit>
	author:  stirng
	message: string
	snapshot: tree
}

type object = blob | tree | commit

objects = map<string, object>
def store(o)  # o is represent object
	id = sha1(o)
	objects[id] = o
def load(id)
	reutrn object[id]
```

下面这条命令可以查看树对象、提交对象中包含什么东西
```git
git cat-file -p  hashcode
```
**树对象**：记录着目录结构和blob对象索引。
**提交对象**：指向树对象的指针和所有提交信息。
下图是包含三个文件暂存的首次提交对象及其树结构
![[Pasted image 20230212215233.png]]
下图是提交对象及其父对象
![[Pasted image 20230212215303.png]]

* Git的分支，本质上就是指向提交对象的可变指针
下图可以看出master分支和v1.0的tag都是指向提交对象的指针
![[Pasted image 20230212215446.png]]

#### 创建分支
创建分支就是创建了一个可以移动的新的指针。
```git
git branch testing  # 创建了一个testing分支
```

HEAD指针，指向当前所在本地分支。
可以使用`git log`命令查看各个分支当前所指的对象(`--decorate`)
```git
git log --oneline --decorate
```

#### 切换分支
```git
git checkout testing # 切换到 testing分支
```

![[Pasted image 20230213010434.png]]

![[Pasted image 20230213010749.png]]

* 创建新分支的同时切换过去
```git
git checkout -b <newbrancename>
```

#### 合并分支
```git
git merge <branchname>
```

当你试图合并两个分支时， 如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候， 只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 **“快进（fast-forward）”。**

下面的命令可以删除不需要的分支
```git
git branch -d <branchname>
```

分支的合并： 检出到你想合并入的分支，然后运行 `git merge` 命令：
```
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```
和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。
![[Pasted image 20230213013113.png]]

任何因包含合并冲突而有待解决的文件，都会以未合并状态标识出来。(`git status`查看)
```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======     
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

```txt
`=======`分成了上下两部分，为了解决冲突，你必须选择使用由 `=======` 分割的两部分中的一个，或者你也可以自行合并这些内容。
```

可以使用图形化工具来解决冲突， `git mergetool`

#### 合并分支提交信息
在使用`git merge`命令时，**创建的新合并提交的提交信息（commit message）默认是自动生成的**，包含一些合并相关的信息。如果你想手动编写合并提交的提交信息，可以使用`--edit`或`-e`选项来打开文本编辑器编辑提交信息。

例如，使用以下命令进行手动编写合并提交的提交信息：

```
git merge --no-ff --edit <branch>
```

这将打开你配置的默认文本编辑器，让你编辑合并提交的提交信息。你可以在编辑器中输入你想要的提交信息，保存并关闭编辑器后，Git将使用你提供的提交信息创建新的合并提交。

注意，如果你只想简单地修改自动生成的提交信息，而不是从头开始编写，你可以使用以下命令：

```
git merge --no-ff --message "Your custom message" <branch>
```

这将使用你提供的自定义提交信息创建新的合并提交。

### 分支管理
`git branch` 命令不只是可以创建与删除分支。 如果不加任何参数运行它，会得到当前所有分支的一个列表：
```git
$ git branch
  iss53
* master   # 代表现在 checkout 在这个分支，HEAD指针当前指向的分支
  testing
```

`--merged` 与 `--no-merged` 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到**当前分支**的分支。 如果要查看哪些分支已经合并到当前分支，可以运行 `git branch --merged`

已经合并的分支就可以删除了，没有合并的分支用`git branch -d`命令删除时会失败。

tips:
你总是可以提供一个附加的参数来查看其它分支的合并状态而不必检出它们。 例如，尚未合并到 `master` 分支的有哪些？
```git
$ git checkout testing
$ git branch --no-merged master
  topicA
  featureB
```

### 远程分支
远程引用是对远程仓库的引用（指针），包括分支、标签等等。
可以通过 `git ls-remote <remote>` 来显式地获得远程引用的完整列表， 或者通过 `git remote show <remote>` 获得远程分支的更多信息。

**远程跟踪分支**是远程分支状态的引用。它们是你无法移动的本地引用。一旦你进行了网络通信， Git 就会为你移动它们以精确反映远程仓库的状态。

它们以 `<remote>/<branch> `的形式命名。 例如，如果你想要看你最后一次与远程仓库 origin 通信时 master 分支的状态，你可以查看 origin/master 分支。
![[Pasted image 20230213022820.png]]

如果你在本地的 `master` 分支做了一些工作，在同一段时间内有其他人推送提交到 `git.ourcompany.com` 并且更新了它的 `master` 分支，这就是说你们的提交历史已走向不同的方向。 即便这样，只要你保持不与 `origin` 服务器连接（并拉取数据），你的 `origin/master` 指针就不会移动。
![[Pasted image 20230213022933.png]]

如果要与给定的远程仓库同步数据，运行 `git fetch <remote>` 命令（在本例中为 `git fetch origin`）。 这个命令查找 “origin” 是哪一个服务器（在本例中，它是 `git.ourcompany.com`）， 从中抓取本地没有的数据，并且更新本地数据库，移动 `origin/master` 指针到更新之后的位置。
![[Pasted image 20230213022958.png]]

多个remote的情况：
![[Pasted image 20230213023057.png]]

#### 推送
如果希望和别人一起在名为 `serverfix` 的分支上工作，你可以像推送第一个分支那样推送它。 运行 `git push <remote> <branch>`

你也可以运行 `git push origin serverfix:serverfix`， 它会做同样的事——也就是说“推送本地的 serverfix 分支，将其作为远程仓库的 serverfix 分支” 可以通过这种格式来推送本地分支到一个命名不相同的远程分支。 如果并不想让远程仓库上的分支叫做 serverfix，可以运行 `git push origin serverfix:awesomebranch` 来将本地的 serverfix 分支推送到远程仓库上的 awesomebranch 分支。

TODO 如果想推送的远程分支不存在怎么处理？
![[Git pull,push技巧#git push出现的问题。]]
* TODO 需要理解下面这段话
要特别注意的一点是当抓取到新的远程跟踪分支时，本地不会自动生成一份可编辑的副本（拷贝）。 换一句话说，这种情况下，不会有一个新的 `serverfix` 分支——只有一个不可以修改的 `origin/serverfix` 指针。  ^15e0dd

#### 跟踪分支
**从一个远程跟踪分支检出(checkout)一个本地分支会自动创建所谓的“跟踪分支”**（它跟踪的分支叫做“上游分支”）。 跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

那如何设置其他跟踪分支呢？ 或者跟踪其他远程仓库上的分支？ 或者不跟踪master分支
```
git checkout -b <branch> <remote>/<branch>

这是一个十分常用的操作，所以Git提供了 --track 的快捷方式：
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```
由于这个操作太常用了，**该捷径本身还有一个捷径。** 如果你尝试检出的分支 (a) 不存在且 (b) 刚好只有一个名字与之匹配的远程分支，那么 Git 就会为你创建一个跟踪分支：
```git
快捷方式的快捷方式
$ git checkout serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```
**以上是本地没有的分支 设置跟踪远程分支，并切换到新分支**

**设置已有的本地分支**跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支， 你可以在任意时间使用 `-u` 或 `--set-upstream-to` 选项运行 `git branch` 来显式地设置。
```
$ git branch -u origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
```

note:
```
当设置好跟踪分支后，可以通过简写 `@{upstream}` 或 `@{u}` 来引用它的上游分支。 所以在 `master` 分支时并且它正在跟踪 `origin/master` 时，如果愿意的话可以使用 `git merge @{u}` 来取代 `git merge origin/master`。
```

##### 查看跟踪远程分支的信息
如果想要查看设置的所有跟踪分支，可以使用 git branch 的 -vv 选项。 这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支与本地分支是否是领先、落后或是都有。
```git
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```

#### 删除远程分支
如果你不需要一个远程分支了（在这个远程分支上的工作已经合并到了远程仓库的master分支上或者其他稳定分支上）

可以运行带有 --delete 选项的 git push 命令来删除一个远程分支。 如果想要从服务器上删除 serverfix 分支，运行下面的命令：
```git
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
 - [deleted]         serverfix
```

### 变基
Git 中整合来自不同分支的修改主要有两种方法：`merge` 以及 `rebase`。

merge命令合并结果：
![[Pasted image 20230224030113.png]]

rebase命令合并结果：
![[Pasted image 20230224030135.png]]

极其重要：
**提取在 C4 中引入的补丁和修改，然后在 C3 的基础上应用一次**。 在 Git 中，这种操作就叫做**变基（rebase）**。 你可以使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“**重新播放**”一样。

在这个例子中，你可以checkout到 `experiment` 分支，然后将它变基到 `master` 分支上：
```console
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

原理： 找**共同祖先**，把当前分支基于祖先的提交都存为临时文件，然后将当前分支指向变基操作的目标分支，最后将之前另存为临时文件的修改依序应用。

现在回到 `master` 分支，进行一次快进合并。

```console
$ git checkout master
$ git merge experiment
```

![[rebase后fast-forward-merge.png]]

此时，`C4'` 指向的快照就和 [the merge example](https://git-scm.com/book/zh/v2/ch00/ebasing-merging-example) 中 `C5` 指向的快照一模一样了。 这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的， 但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。

#### rebase example

![[Pasted image 20230224031034.png]]
如何将client分支中的c8 c9合并到主分支，和不合并server中的修改(本例中是c3)?
```
--onto 选项 选中在client分支里但是不在server分支里的修改
git rebase --onto master server client

这条命令的意思是： 取出client分支，找出它从server分支分歧之后的补丁，然后把这些补丁在master分支上重放一遍

最后要以 fast-forward合并分支
git checkout master
git merge client   快速合并
```

``git rebase <basebranch> <topicbranch>`

```

```git
这个操作会把 server分支中的修改也整合进来。
git rebase master server   
git checkout master
git merge server
```

#### 变基的风险
**如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。**

💡如何理解这句话：如果你是和别人合作开发， 你的代码是从中央服务器获得的， 你现在基于中央服务器上的一个分支进行开发，然后开发的过程中， 某人在中央服务器上在你开发依赖的一个分支上，做了rebase, 然后你开发依赖的分支在中央服务器上没有， 并且中央服务器多出来一个提交，这时候你Pull代码，再merge, 就很尴尬， 某人原本不想要的分支，现在还在你的本地， 你如果这时候 push 分支，就会把别人 rebase 掉的分支再搞回来。🤣

详细案例请看
[Git - 变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)

变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。
#### 用变基解决变基
如果你习惯使用 `git pull` ，同时又希望默认使用选项 `--rebase`，你可以执行这条语句 `git config --global pull.rebase true` 来更改 `pull.rebase` 的默认配置。

#### 使用变基注意事项
如果你**只对不会离开你电脑的提交执行变基**，那就不会有事。 如果你对已经推送过的提交执行变基，但别人没有基于它的提交，那么也不会有事。 如果你对已经推送至共用仓库的提交上执行变基命令，并因此丢失了一些别人的开发所基于的提交， 那你就有大麻烦了
