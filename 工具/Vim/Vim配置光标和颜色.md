Windows Terminal里的Vim不知道为什么在Normal模式下默认竖线，Insert模式下为下划线，所以想修改光标的颜色和样式。

* Windows Terminal下不是Gvim，不能用guicursor配置

```vim
" Set cursor shape and color
if &term =~ "xterm"
    " INSERT mode
    let &t_SI = "\<Esc>[6 q" . "\<Esc>]12;blue\x7"
    " REPLACE mode
    let &t_SR = "\<Esc>[3 q" . "\<Esc>]12;black\x7"
    " NORMAL mode
    let &t_EI = "\<Esc>[2 q" . "\<Esc>]12;green\x7"
endif
" 1 -> blinking block  闪烁的方块
" 2 -> solid block  不闪烁的方块
" 3 -> blinking underscore  闪烁的下划线
" 4 -> solid underscore  不闪烁的下划线
" 5 -> blinking vertical bar  闪烁的竖线
" 6 -> solid vertical bar  不闪烁的竖线
```

可通过 `:h t_SI 等查看具体变量的意义`

`"\<Esc>[6 q"` 字符串用来配置光标的形状
`"\<Esc>]12;blue\x7"` 字符串 用来配置光标颜色
* 设置光标颜色时，也可以使用RGB颜色，格式为`rgb:RR/GG/BB`。比如纯白色的光标即为 `"\<Esc>]12;rgb:FF/FF/FF\x7"`

#### 参考方法
[How to change the cursor between Normal and Insert modes in Vim? - Stack Overflow](https://stackoverflow.com/questions/6488683/how-to-change-the-cursor-between-normal-and-insert-modes-in-vim)