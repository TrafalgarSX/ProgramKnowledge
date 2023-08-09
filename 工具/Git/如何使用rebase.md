### git rebase 合并分支
---
没有rebase前
![[1690911382243.jpg]]

rebase后
![[rebase后log.png]]

可以看到在rebb 分支上 rebase master后， rebb分支上的两个提交 be829f1 982a931 消失了，在master 分支的最新commit上的 2eefbfc 后添加了两个新的commit。

这就是 rebase 的作用， rebb 和 master 分支原本都基于 75349e, rebb后来 rebase了 master ， 结果就是将 rebb 分支的两个提交的 base 变成了 master 的最新的 commit 2eefbfc。 这就是 **变基**， 修改分支的 base。 
#### 注意
![[rebase解决冲突.png]]

在 rebase 过程中也会有冲突， 像上面的例子中， rebb 分支有两个 commit 需要在master分支上`重放`, 这就会可能导致冲突， 实际上也真的导致了冲突， 那有了冲突怎么办， 该输入什么命令呢？

**在“重放”，把rebb分支接上去就是逐个和新基底处理冲突的过程**

git 给了4个提示：
1. 遇到冲突就解决， 解决完了 输入 `git rebase --continue`, 继续 rebase。
2. 删除冲突文件， 然后 `git rebase --continue`, 这个太鸵鸟了， 不能随便删除文件
3. 跳过当前要 rebase 的 commit, `git rebase --skip`
4. 放弃 rebase, 回到 rebase 前的状态， 输入 `git rebase --abort`

### 什么是rebase
---
![[Git分支#变基]]

### 合并提交
---
```bash
git rebase -i HEAD~4
```

在 git rebase -i HEAD~4 命令中，HEAD~4 表示最近的 4 个提交，也就是要对最近的 4 个提交进行交互式合并。你可以根据需要修改这些提交的操作，例如合并、删除、重新排序等。

```markdown
-i 参数的作用是告诉 Git 打开一个交互式界面，以便对提交进行操作。

HEAD~4 是一个相对引用，表示相对于当前提交（HEAD）的前 4 个提交。这意味着你将会操作最近的 4 个提交。

通过使用 HEAD~<number> 的方式，你可以指定相对于当前提交的前几个提交，其中 <number> 是一个整数，表示要回溯的提交数。例如，HEAD~1 表示当前提交的前一个提交，HEAD~2 表示当前提交的前两个提交，以此类推。
```

具体而言，该命令将打开一个文本编辑器，列出了最近的 4 个提交的相关信息，并且允许你对这些提交进行操作。你可以对每个提交进行以下操作之一：

- pick：保留该提交。
- reword：保留该提交，并修改提交信息。
- edit：保留该提交，并在合并过程中停下来进行修改提交内容。
- squash：将该提交合并到前一个提交。
- fixup：将该提交合并到前一个提交，但不保留该提交的提交信息。
- drop：删除该提交。
通过修改这些提交的操作，你可以重新排序提交、合并提交或删除提交。完成编辑后，保存并关闭编辑器，Git 将会按照指定的操作对提交进行合并。

需要注意的是，git rebase -i HEAD~4 命令会修改提交历史，因此在使用该命令之前，请确保你了解其影响，并且在操作前先备份你的代码。

### git pull --rebase
---
Git 在最近的某个版本起，直接运行 `git pull` 会有如下提示消息：

```bash
warning: 不建议在没有为偏离分支指定合并策略时执行 pull 操作。 您可以在执行下一次 pull 操作之前执行下面一条命令来抑制本消息：

  git config pull.rebase false  # 合并（缺省策略）
  git config pull.rebase true   # 变基
  git config pull.ff only       # 仅快进

......
```

原来 `git pull` 时也可以通过 `rebase` 来进行合并，这是因为 `git pull` 实际上等于 `git fetch` + `git merge` ，我们可以在第二步直接用 `git rebase` 替换 `git merge`来合并 `fetch` 取得的变更，作用同样是避免额外的 `merge` 提交以保持线性的提交历史。

两者的区别在上文中已进行过对比，我们可以把对比示例中的 `Matser` 分支当成远程分支，把 `Feature` 分支当成本地分支，当我们在本地执行 `git pull` 时，其实就是拉取 `Master` 的更改然后合并到 `Feature` 分支。如果两个分支都有不同的提交，默认的 `git merge` 方式会生成一个单独的 merge 提交以整合这些提交；而使用 `git rebase` 则相当于基于远程分支的最新提交重新创建本地分支，然后再重新应用本地所添加的提交。

具体的使用方式有多种：

- 每次执行 pull 命令时添加特定选项： `git pull --rebase` 。
- 为当前仓库设定配置项： `git config pull.rebase true`，在 `git config` 后添加 `--global` 选项可以使该配置项对所有仓库生效。

### 不仅仅是分支
---
虽然我们之前的示例都是在不同的两个分支之间执行 rebase 操作，**但事实上 rebase 命令传入的参数并不仅限于分支。**

**任何的提交引用**，都可以被视作有效的 `rebase` 基底对象，包括一个
- 提交 ID
- 分支名称
- 标签名称
- `HEAD~1` 这样的相对引用。

### rebase 更好的场景与理由
---
For most operations, a rebase is preferred to a merge:

- It remembers each of your commits.
    
- Your commits will always show up as the last in the list (“the cream rises to the top”)
    
- Note that the old commits (C6..C8) are no longer referenced by any refs, so they are now available for “garbage collection”.
### rebase 的缺点
---
我们上面已经提到了 rebase 有保持整洁的线性提交历史的优点，但也需要意识到它有以下潜在的弊端：

- 如果涉及到已经推送过的提交，需要**强制推送**才能将本地 rebase 后的提交推送到远程。**因此绝对不要在一个公共分支（也就是说还有其他人基于这个分支进行开发）执行 rebase**，否则其他人之后执行 git pull 会合并出一条令人困惑的本地提交历史，进一步推送回远程分支后又会将远程的提交历史打乱（详见Rebase and the golden rule explained），较严重的情况下可能会对你的人身安全带来风险。（git官方书中的例子）

-  对新手不友好，新手很有可能在交互模式中误操作「丢失」某些提交（但其实是能够找回的）。

- 假如你频繁的使用 rebase 来集成主分支的更新，一个潜在的后果是你会遇到越来越多需要合并的冲突。尽管你可以在 rebase 过程中处理这些冲突，但这并非长久之计，更推荐的做法是频繁的合入主分支然后创建新的功能分支，而不是使用一个长时间存在的功能分支。

### 参考
---
[git rebase 用法详解与工作原理 | Shall We Code?](https://waynerv.com/posts/git-rebase-intro/)