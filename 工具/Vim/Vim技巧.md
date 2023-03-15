### Vim中如何打开终端
使用命令`terminal powershell`(windows平台)，如果不指定powershell，会打开cmd
`terminal`可以简写为`term`   linux平台，会默认打开bash

### Vim中如何在insert模式下undo
`<c-u>`  使用该快捷键

### 命令行模式下如何使用刚刚复制的word
输入`:`命令，使用`/`或`?`搜索命令，都将进入命令行模式。

键入命令:set i之后，按下Tab或Ctrl+D键，将显示以“i”开头的set命令；
进入命令行模式之后，点击Ctrl+rCtrl+w键可以将当前光标下的`word`
粘贴到命令行中；点击Ctrl+rCtrl+a键可以将当前光标下的`word`
粘贴到命令行；点击Ctrl+r%键可以将当前文件名粘贴到命令行。


### 复制字符串后，重复替换

前文讲到数字寄存器，复制操作时，会保存内容至匿名寄存器以及”0寄存器，而在每次删除操作时，会更新”1-“9寄存器以及匿名寄存器，而p(aste)时，会使用匿名寄存器，所以如果p(aste)操作时，如果有替换内容，匿名寄存器内容会变为替换的内容，不过”0寄存器还是最近一次y(arn)的内

匿名寄存器 缓存最后一次操作内容
"_  黑洞寄存器 不缓存操作内容（干净删除）

p 往光标后复制  P往光标前复制
"replace currently selected text with default register 
"without yanking it
`xnoremap p "_dP`

```vimL
"_d  delete without yanking
```

### 切换vim当前工作目录到当前编辑的文件的目录下
```
:cd %:h 文件名的头部，即文件目录.例如../path/test.c就会为../path

%:t 文件名的尾部.例如../path/test.c就会为test.c
%:r 无扩展名的文件名.例如../path/test就会成为test
%:e 扩展名

```