### Pull Request流程
---
1. 克隆别人的代码：fork
2. 在你fork的仓库修改后的分支上，按下“New pull request”按钮
![[Pull-Request.png]]
这时，你会进入一个新页面，有Base和Head两个按钮。Base是你希望提交变更的目标，Head是你目前包含你的变更的哪个分支或仓库
![[base-head.png]]
3. 填写说明，帮助别人理解你的提交，然后按下"create pull request"按钮即可

PR创建后，管理者就要决定是否接收该PR。对于非代码变更（比如文档），使用Web界面就足够了。但是对于代码变更，可能需要命令行验证是否可以运行。

### git am
---
`git am`命令用于将一个patch文件，合并进入当前代码。
Github对每个PR会自动生成一个patch文件。我们下载该文件，合并进本地代码，就可以在本地查看效果了。
 ```bash
 
 $ curl -L http://github.com/cbeust/testng/pull/17.patch | git am
 ```

上面代码中，`curl`的`-L`参数表示，如果有302跳转，`curl`会自动跟进（跟随重定向）。后面网址里面的`/cbeust/testng`是目标仓库，`pull/17`表示该仓库收到的第17个 PR。

如果 PR 只包含一个 commit，那么也可以直接下载这个 commit 的 patch 文件。

 ```bash
 $ curl https://github.com/sclasen/jcommander/commit/bd770141029f49bcfa2e0d6e6e6282b531e69179.patch | git am
 ```

上面代码中，网址里面的`/sclasen/jcommander`是代码变更所在的那个仓库。