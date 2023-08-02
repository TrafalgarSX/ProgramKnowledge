### 什么是三路合并
---
三路合并是一种常见的合并技术，用于将两个分支的更改合并到一起。它有助于解决分支之间的冲突，并在最终结果中保留所有分支的更改。

核心就是说把两个版本开始变得不一样之前的那个版本（base）也一块拿过来作比较。

通常，我们在一个主分支上进行开发，然后在新的分支上进行额外的更改，例如修复错误或添加新功能。当我们想要将这些更改合并回主分支时，就会用到三路合并。

### 如何进行三路合并
---
```bash
git checkout master
git merge other-branch  // 将分支的更改合并到主分支上
```

### 解决冲突
---
在进行三路合并时，可能会出现冲突。冲突指的是两个分支上的更改无法自动合并，需要手动解决。

Git使用特定的标记来显示冲突的部分。例如：
```bash
<<<<<<< HEAD
This is the main branch
=======
This is the branch to be merged
>>>>>>> branch-name
```

```
冲突的部分用`=======`分隔，
`<<<HEAD`之前的是主分支的更改，
`>>>branch-name`之后的是要合并的分支的更改。

我们需要手动解决这个冲突，选择保留哪些更改或者将两个更改结合在一起。
```

```txt
For conflicting paths, the index file records up to three versions: 
stage 1 stores the version from the common ancestor   BASE
stage 2 from HEAD                      OURS
stage 3 from MERGE_HEAD (you can inspect the stages with git ls-files -u).   THEIRS

The working tree files contain the result of the "merge" program; i.e. 3-way merge results with familiar conflict markers <<< === >>>.
```

### 理解
---
Three-way merge 是因为有a b, c三个节点。如果只提供a,b， 就是two-way merge.

Three-way merge的好处是程序能理解你到底做了哪些改动，如果这些改动没有冲突，则它会自动帮你合并。 Two-way merge说白了就是一个diff, 程序永远不能帮你自动merge, 需要人工干预。

举个例子， 假设有一个文本文件，A和B同时做了改动， a,b 节点如下
![[v2-06530aa113465fb58a1cacec706cd6db_720w.webp]]

现在问题来了，merge的结果应该是怎样的？ 条件不足，无法判断。

如果现在告诉它们的共同祖先c，进行three-way merge:
![[v2-808d45c2081f197dedad108bd5d26552_720w.webp]]

通过base/src的对比，我们知道A的改动是删除了3; 通过base/dst的对比，我们知道B的改动是增加了1，所以merge结果是:

![[v2-1c914d84ddcdedd3c0fa6de8f38e264e_720w.png]]

类似的，如果原来的[c节点](https://www.zhihu.com/search?q=c%E8%8A%82%E7%82%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A866309494%7D)是这样的：

![[v2-a40d94f19c538b09ddfad8692dae4fbb_720w.webp]]

则通过base/src对比，我们知道A的操作是删除了1; 通过base/dst的对比，我们知道B的操作是增加了3。则merge的结果应该是:

![[v2-9d7c1a8cbf87c79ee0248fd32c8a0954_720w.png]]

由此可见，如果只有节点a和b，进行two-way merge，其实是无法确定到底做了什么改动，merge的结果应该是怎样的，必须由用户自己解决。如果有a/b的共同祖先c的信息，[three-way merge](https://www.zhihu.com/search?q=three-way%20merge&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A866309494%7D) 能通过对比知道用户做了哪些增删改操作，并尝试自动merge结果.

### 我的理解（重要）
---
以前一致在svn合作开发的基础上理解三路合并， 这个根基就是错的。

三路合并，首先得有三路  分支1， 分支2， 分支1，2的基础版本， 而我在工作中的merge都是在一个主干上开发， 我合并代码都是在主干上合并别人的最新代码， 这相当于别人的更改， 我的更改， 我们更改的基础版本是三路， 工作中使用的winmerge不支持三路合并， 看不到我们更改的基础版本，所以我一直难以理解， 一直以为是合并的两个分隔就是两路合并。

💡三路合并，两路合并最重要的差别不是合并界面有几个框， **两者的差别重点在于 两个最新版本的修改是否和共同祖先版本进行了比较**，能够得到两个新版本和共同祖先版本的差异，才能知道两个新版本都做了哪些修改， 才能知道我们应该**取舍两个版本做出修改的哪个部分， 而不是在两个版本的差异里做选择**。

***在修改中做选择， 而不是在差异中做选择！！！***

其实简单理解三路合并就是 基本， 改动1， 改动2三路合并

```markdown
在三路合并过程中，JetBrains编辑器**会比较当前分支的最新提交、另一个分支的最新提交以及共同祖先提交之间的差异，并生成合并结果。**

当你使用JetBrains编辑器的合并工具时，它会自动识别和比较这三个版本之间的差异，以确定文件的修改和冲突。它会根据共同祖先提交、当前分支的最新提交和另一个分支的最新提交之间的差异来生成合并结果。

**WinMerge作为另一种文件比较和合并工具，通常用于比较和合并两个文件的差异，而不是三路合并。** WinMerge只会比较最新两个提交之间的差异，并不会直接比较共同祖先提交和另外两个最新提交之间的差异。

因此，在三路合并方面，JetBrains编辑器提供了更全面的功能，它会比较共同祖先提交、当前分支的最新提交和另一个分支的最新提交之间的差异，并生成合并结果。WinMerge则更适用于比较和合并两个文件的差异，而不是处理完整的三路合并。
```

### vscode 的三路合并功能为什么比我想的少个BASE代码框
---
老的vscode 并不支持 git merge的三路合并， 合并代码的时候，只能看到自己的更改和别人传入的更改。

![[124872716-304a6f00-dff8-11eb-9843-2c6947f51222.png]]

这就导致merge的时候只能看到 两个版本间的diff, 我们只能选择当前更改或者传入的更改，这可能会导致问题， 因为可能两个版本间的diff中间**可能有冲突我们必须要选择其一的部分， 也有新增的功能必须要在最后的 result 中的代码**， 如果我们只选择其中一个，虽然解决了冲突的部分，但是我们有可能把新增的想要的新功能也给解决掉了，这样就会丢失新的代码。

现在新版本的vscode已经是 4-editor view  which also shows the base editor 了。

vscode 上的issue 有和我一样的疑惑 [Please show base by default in the 3-way-merge editor · Issue #165628 · microsoft/vscode · GitHub](https://github.com/microsoft/vscode/issues/165628) 

