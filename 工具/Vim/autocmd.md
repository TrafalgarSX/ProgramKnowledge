### 自动命令(autocmd)
---
自动命令，是在指定事件发生时自动执行的命令。利用自动命令可以将重复的手工操作自动化，以提高编辑效率并减少人为操作的差错。

#### 定义自动命令
格式：
```
:autocmd [group] events pattern [nested] command
```
- group, 组名是可选项，用于分组管理多条自动命令
- events，事件参数， 用于指明触发命令的一个或多个事件
- pattern， 限定针对符合匹配模式的文件执行命令
- nested, 嵌套标记是可选项，用于允许嵌套自动命令
- command, 指明需要执行的命令、函数或脚本

![[vim事件.png]]

从打开文本到输入文本，然后保存退出， 这些操作将以以下顺序触发一系列事件
![[触发事件顺序.png]]

```
:h autocommand-events
```

#### pattern参数
匹配模式用来指定**应用自动命令的文件（对哪些文件生效）**。在匹配模式中，可以使用以下特殊字符：

`*` 匹配任意长度的任意字符  
`?` 匹配单个字符  
`\?`匹配字符'?'  
`.` 匹配字符'.'  
`,` 用于分割多个pattern  
`\,`匹配字符','

可以使用多个逗号来分割多个模式，以匹配多种类型的文件。
```vim
将对于.c和.h文件设置'textwidth'选项：
:autocmd BufRead,BufNewFile *.c,*.h set tw=0
```

#### nested参数
默认情况下，自动命令并不会嵌套执行。例如在自动命令中执行:e或:w命令，将不会再次触发BufRead和BufWrite事件。而使用nested参数，则可以激活嵌套的事件。
```vim
:autocmd FileChangedShell *.c nested e!
```

#### 查看自动命令
```
:autocmd

会列出所有的自动命令

:autocmd filetypedetect * *.htm
会查看相匹配的自动命令
```

#### 删除自动命令
```
:autocmd!
```

删除指定组的自动命令：
```
:autocmd! group

指定组、事件和匹配模式，可以删除特定的自动命令
:autocmd! Unfocussed FocusLost *.txt*
```

在命令中使用特殊字符`“*”`来指代所有事件或文件。例如以下命令，将删除Unfocussed组中所有针对txt文件的自动命令：
```
autocmd! Unfocussed * *.txt
```

#### 自动命令组
通过`:augroup`命令，可以将多个相关联的自动命令分组管理，以便于按组来查看或删除自动命令。
```
:augroup cprograms
:    autocmd!
:    autocmd FileReadPost *.c :set cindent
:    autocmd FileReadPost *.cpp :set cindent
:augroup END
```

#### 示例
离开Vim编辑器时，自动保存文件
```
autocmd FocusLost * :wa
```

根据文件类型执行自动命令
```vim
autocmd BufEnter *.php :%s/[ \t\r]\+$//e
```

根据文件类型设置键盘映射
```vim
autocmd bufenter *.tex map <F1> :!latex %<CR>
```

根据文件类型，设置不同的选项：
```vim
autocmd FileType ruby setlocal ts=2 sts=2 sw=2 expandtab
```

自动应用配置文件
```
augroup Reload_Vimrc        " Group name.  Always use a unique name!
    autocmd!                " Clear any preexisting autocommands from this group
    autocmd! BufWritePost $MYVIMRC source % | echom "Reloaded " . $MYVIMRC | redraw
    autocmd! BufWritePost $MYGVIMRC if has('gui_running') | so % | echom "Reloaded " . $MYGVIMRC | endif | redraw
augroup END
```