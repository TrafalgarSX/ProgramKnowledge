Gist https://gist.github.com/ 是Github的一个子服务。最简单的功能就是分享代码片段，例如把一些小型脚本放到Gist方便分享和管理。不同于大型项目使用repository进行管理，Gist就是小型代码片段的分享。类似的服务还有如Pastie等，但明显Gist更有优势（Github)。

gist提供的功能：（包括但不限于）
- gist 能无限制的免费创建私有代码片段，而不被搜索，只有通过浏览器输入其URL才能看见
- 匿名张贴，不需要有Github账号就可使用
- 版本管理功能
- gist随同github提供了api开发接口，可以在本地创建gist, 可以使用Git进行操作
- 制作任务列表
- 作为一个网页收藏夹


### 创建新Gist, 编辑修改Gist

-   跑到[https://gist.github.com/](https://gist.github.com/) 直接填写内容或者在自己的Gist 右上角上点击 `New gist`即可
-   可以一个Gist多个文件, 使用 `Add file` 添加即可.
-   可以设置indent为空格space还是tab, tab长度, 是否行缩进.
-   点 _Create secret gist_ 创建私有代码, _Create public gist_ 创建开放的gist. 前者可以不被搜索到.
-   创建Gist后,点选自己的某个Gist, 进去后右上角可进行网上的编辑/修改: Edit, 编辑; Delete, 删除; Star, 标星. 旁边还有举报 2333. 修改后下方的`Update public/secret gist`即可保存修改.
-   编辑时上方的`Make Secret`可以转为私有库.

### 浏览Gist

-   左上角可以看到列出自己最近的gist, 右上角`See all of your gists`可以查看所有自己的Gist.私有gist会显示SECRET标签.
-   搜索框可以进行代码搜索(开放gist), 可能搜出相关的代码片段
-   点`All Gists`可以到`Discover gists`模式, 查看最近发布或被fork的gists(或者别的排序方式). 参考意义不大.
-   在浏览Gist时点击右上`GithubGist`图标或者左上头像选`Your Gists`即可返回
-   在浏览Gist文件时, 点Raw可以看文字的纯代码.