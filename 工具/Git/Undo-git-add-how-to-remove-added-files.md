### 如何删除误添加到git版本管理系统中的文件
---
有两种主要的方法 undo `git add` 或者 remove added files in Git.

- `git reset`
- `git rm --cached`

#### `git reset`
![[Git重置#通过路径来重置]]

#### `git rm --cached`
On the other hand, **`git rm` is used to remove a file from the staging area and the working directory**. This means that it will permanently delete the file from your repository.

永久删除暂存区和工作目录中对应的文件

```bash
git rm <file>
```

But you only want to “unstage” your files (that is, undo the `git add` command) and not “remove” them from your working repository. This is where you use the `--cached` flag. The cached option specifies that the removal should happen only on the staging index. Working directory files will be left alone.

```bash
git rm --cached <file>
```

这只会删除暂存区中的， 不会删除工作目录中的。

