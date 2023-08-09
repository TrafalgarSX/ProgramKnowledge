### BLOB TREE COMMIT的组成与关系
![[Git分支#分支操作的对象]]

这就是git快照系统的本质：文本有修改 --> 生成新的BLOB对象（与之前的BLOB对象有不同的hash值）--> 生成新的TREE对象，目录中引用新的BLOB对象（hash改变），和内容不变的BLOB对象（相同的对象，hash值不变） --> 生成新的COMMIT对象  -->   新的提交
![[Pasted image 20230327001341.png]]

由于这次 **提交（COMMIT）** 不是第一次 **提交（COMMIT）**，所以它有一个父节点——**commit A1337**。

### Git中的分支

💡分支只是提交对象(COMMIT)的命名引用。 

我们可以一直用 SHA-1 哈希值引用一个 **提交**，但是人们通常喜欢以其他形式命名对象。**分支** 恰好是引用 **提交** 的一种方式，实际上也只是这样。

`git branch` 命令创建一个新分支， 实际上我们创建的是另一个指针(pointer)
![[Pasted image 20230327012923.png]]
`HEAD`是一个特殊指针，它指向当前所在的分支，分支指向一个提交。

`git`内部将**分支**成为**heads**，所以.git文件夹中会创建一个`./.git/ref/heads`的目录。

### 如何在Git中记录变化
通常我们在工作目录中编写源代码。 工作目录关联着一个仓库(repo)

在做了一些改动之后，我们会把这些改动记录到**仓库**中。一个**仓库**就是一系列**提交（COMMIT）** 的集合。每个 **提交** 都是工程 **工作树** 的归档。

**仓库**也包含除代码文件以外的其他东西，如`HEAD`指针、分支等等。

创建 blob 这个过程通常发生在我们将一些东西添加到 暂存区 的时候——也就是我们使用 git add 的时候。

记住：git 是为 整个 暂存的文件创建 blob。即使文件中只有修改或添加了一个字符（如同我们在之前的例子红添加 ! 一样），该文件也会有一个新的 blob，这个 blob 有着新的哈希值。

#### .git组成
一个 **仓库** 可能还包含一些其它的东西，比如 git 钩子（hooks）。不过，仓库至少必须要有对象和引用。

让我们分别为对象和引用（简称：**refs**）各创建一个目录，Windows 下的两个目录分别为 `.git\objects` 和 `.git\refs`（UNIX 下的两个目录分别为 `.git/objects` 和 `.git/refs`）。

**分支** 是引用的一种，`git` 内部将 **分支** 称为 **heads**，所以我们会为它们创建一个目录 `git\refs\heads`。

`HEAD`，它是一个位于 `.git\HEAD` 的文件。我们可以这么

- `.git/objects` git的对象数据库，里面存储着BLOB,TREE,COMMIT等对象
```
所以，`git` 实际上是使用 SHA-1 哈希值的前两个字符作为目录的名字，剩余字符用作 **blob** 所在文件的文件名

为什么要这样呢？考虑一个非常大的仓库，仓库的数据库内存有三十万个对象（blob 对象、树对象和提交对象）。从这三十万个哈希值中找出一个值会花些时间，因此，`git` 将这个问题划分成了 256 份。
```
- `.git/index`文件，这就是著名的**索引（或暂存区）**， 它基本上就是一个位于`.git/index`的文件。

### Git for Computer Scientists
---
![[图#DAG图]]
#### Storage
##### blob
In simplified form, **git object storage is "just" a DAG of objects**, with a handful of different types of objects. They are all stored compressed and identified by an SHA-1 hash (that, incidentally, _isn't_ the SHA-1 of the contents of the file they represent, but of their representation in git).

![[blob.png]]

`blob`: The simplest object, **just a bunch of bytes**. This is often a file, but can be a symlink or pretty much anything else. The object that points to the `blob` determines the semantics.

##### tree

![[tree.png]]

`tree`: Directories are represented by `tree` object. **They refer to `blob`s** that have the contents of files (filename, access mode, etc is all stored in the `tree`), and to **other `tree`s for subdirectories**.

When a node points to another node in the DAG, it _depends_ on the other node: it cannot exist without it. **Nodes that nothing points to can be garbage collected with `git gc`**, or rescued much like filesystem inodes with no filenames pointing to them with `git fsck --lost-found`.

##### commit
![[commit.png]]

`commit`: **A `commit` refers to a `tree` that represents the state of the files at the time of the commit**. It **also refers to 0..`n` other `commit`s that are its parents**. 

More than one parent means the commit is a merge, no parents means it is an initial commit, and interestingly there can be more than one initial commit; this usually means two separate projects merged. **The body of the `commit` object is the commit message.**

##### HEAD and branch
![[HEAD_COMMIT.png]]

`refs`: **References, or heads or branches**, are like post-it notes slapped on a node in the DAG. Where as the DAG only gets added to and existing nodes cannot be mutated, **the post-its can be moved around freely**. They don't get stored in the history, and they aren't directly transferred between repositories. They act as sort of bookmarks, "I'm working here".

- mutated 突变， 使变化
- post-it 便利贴

💡`git commit` adds a node to the DAG and moves the post-it note for current branch to this new node.

The `HEAD` ref is special in that it actually points to another ref. It is a pointer to the currently active branch. 

Normal refs are actually in a namespace `heads/XXX`, but you can often skip the `heads/` part.

##### remote refs
![[remote-refs.png]]

`remote refs`: Remote references are post-it notes of a different color. The difference to normal `refs` is the different namespace, and **the fact that remote refs are essentially controlled by the remote server**. `git fetch` updates them.

##### tag
![[tag.png]]

`tag`: **A `tag` is both a node in the DAG and a post-it note (of yet another color)**. 

💡A `tag` points to a `commit`, and includes an optional message and a GPG signature.

The post-it is just a fast way to access the tag, and if lost can be recovered from just the DAG with `git fsck --lost-found`.

The nodes in the DAG can be moved from repository to repository, can be stored in more effective form (packs), and unused nodes can be garbage collected. 

But in the end, a `git` repository is always just a DAG and post-its.


### 参考
---
- [Git 内部原理图解——对象、分支以及如何从零开始建仓库](https://www.freecodecamp.org/chinese/news/git-internals-objects-branches-create-repo/)

- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/)