### vim重复技巧
----
当我们编辑文本的时候，执行了一些命令，我们向重复这些命令，但又不想重复做一遍一样的操作（可能命令很长）。 vim实际上会将之前的操作记录在vim寄存器中，方便进行重复操作。

| 重复类型     | 重复操作符 | 回退操作符 |
| ------------ | ---------- | ---------- |
| 文本改变重复 | .          | u          |
| 行内查找重复 | ;          | ,          |
| 全文查找重复 | n          | N          |
| 文本替换重复 | &          | u          |
| 宏重复       | @[reg]     | u          |

#### 文本改变重复
`.` 操作就是用来重复上一次操作的
> 例如：
> 使用 dd 命令删除一行，接着用 `.` 重复 dd 命令
> 使用 yy 复制一行数据，用 `p` 进行粘贴， 点击 `.` 进行重复粘贴操作

#### 行内查找重复
vim 可以使用  `f{char}{char}` 来从当前位置开始到行尾进行查找， `F{char}{char}` 从当前位置开始到行首进行查找。
```
可以通过 :h ; 来查看 vim 的帮助手册

; 是重复进行正向查找的工作， , 是重复反向查找的工作
```

#### 全文查找重复
```
:h / 来查看 vim 帮助手册

/{pattern} 正向查找  ?{pattern} 反向查找

n Repeat the latest “/” or “?” [count] times. last-pattern {Vi: no count} 查询结果里 正向查找

N Repeat the latest “/” or “?” [count] times in opposite direction. last-pattern {Vi: no count}  查询结果里反向查找
```

#### 文本重复替换
```
:h & 查看 vim 帮助手册
& Synonym for :s (repeat last substitute). Note that the flags are not remembered, thus it might actually work differently. You can use :&& to keep the flags.
```

```
:{range}s/target/replacement/g 
该命令使用 replacement 替换 {range}  范围里行内所有的  target 字符

可以使用 & 来重复替换操作
```

#### 宏录制重复
```
:h @ 来查看 vim 的帮助手册
@{0-9a-z".=*+} Execute the contents of register {0-9a-z”.=*+} [count] times. Note that register ‘%’ (name of the current file) and ‘#’ (name of the alternate file) cannot be used.
```

vim 中的宏录制时将一系列的操作保存到寄存器中， 然后直接使用 @{reg} 就可以重复之前录制的宏操作

vim 可以使用 q{reg} 开始进行录制， 时候使用 q 结束录制， {reg}是 a-z 中的任意一个。

录制结束后，比如 qa 录制操作到寄存器 a 中后， 使用 @a 就可以重复录制的操作