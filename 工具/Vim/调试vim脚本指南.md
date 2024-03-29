### 如何调试vim脚本
使用 `-D` 参数可以开启 Debug 模式， 在 Debug 模式中可以使用 `cont`, `next`, `interrupt`, `step`, `quit` 等调试命令， 以及 `breakadd`, `breakdel` 来添加和移除断点。 

使用 `-u` 来禁止加载任何配置文件，使用 `:source` 命令逐个加载。 使用 `:set verbose` 和 `:set verbosefile` 等 [配置变量](https://harttle.land/2017/01/30/variables-in-vim.html) 可以设置日志级别和输出文件， `-V` 启动参数也可以起到同样的作用。

#### 日志级别
在介绍 Debug 之前有必要先介绍如何查看运行日志，**本文只介绍到日志级别的设置方法和日志文件的设置方法。** 日志级别是由 `verbose` [配置变量](https://harttle.land/2017/01/30/variables-in-vim.html) 控制的。这是一个默认为零的数字，级别越高输出越详细。 `verbose` 非零时 Vim 就会在当前窗口显示日志信息，例如 `:set verbose=20` 就可以开启几乎所有日志。 这些级别的描述如下：
```
>= 1	读写 viminfo 文件时
>= 2	当 ":source" 一个文件时，通常是载入一个配置文件
>= 5	所有搜索到的 tag 文件和 include 文件
>= 8	执行到 autocommands 的文件
>= 9	每次执行到 autocommand 时
>= 12	每次执行到 function 时
>= 13	产生、捕获、结束处理、忽略一个异常时
>= 14	":finally" 语句中等待的所有命令
>= 15	每一个执行到的 Ex 命令（截断到 200 字符）
```
更多 `verbose` 选项变量的信息可以参考 `:help vbs`。 日志级别也可以通过 `-V` 启动参数来设置。例如：
```
vim -V20 main.cpp
```

#### 日志文件
如果你试过上述日志级别的设置方式会发现日志直接输出在当前屏幕，这会影响在当前 Vim 中的连贯操作。 我们可以通过 `:set verbosefile` 把日志输出到文件中。例如把它输出到 `vim.log` 文件：
```
set verbosefile=vim.log
```
然后在另一个 Shell 中 `tail` 这个文件，我们就可以一边使用 Vim 一边查看它的日志了：
```
tail -f vim.log
```
更多 `verbosefile` 选项变量的信息可以参考 `:help verbosefile`。 日志文件也可以通过 `-V` 启动参数来设置。例如设置级别为 `20`，输出文件名为 `vim.log`：
```
vim -V20vim.log main.cpp
```
注意级别与文件名之间不能加空格，且文件名不得以数字开头（否则会被识别为级别的一部分）。

#### 零配置启动
有时为了定位问题，我们可以禁止 Vim 自动加载配置文件和插件，手动地逐个加载 Vim 脚本。 需要先零配置启动 Vim，通常设置 `-u` 即可：
```
vim -u NONE main.cpp
```
如果你在使用 GVim 可能需要使用 `-U`，具体情况可参考 `:help -U` 或 [StackOverflow](https://vi.stackexchange.com/questions/2003/how-do-i-debug-my-vimrc-file)。 启动后使用 `:source {filename}` 命令来逐个加载 Vim 配置即可。

#### Debug方法
在启动 Vim 时添加 `-D` 参数即可开启调试模式，Vim 会在第一行配置文件出中断并进入 Debug 模式：
```
vim -D main.cpp
```
中断后会进入 Debug 模式，你可以看到 `>` 提示符。此时你就可以输入 Debug 命令，操作方式和 gdb 非常类似：

-   cont: 继续执行直到下一个断点。
-   quit: 停止当前过程，继续执行到下一个断点。
-   step: 执行当前断点处的指令，执行完后仍然回到 Debug 模式。
-   next: 与 step 一样，但会直接执行完整个方法调用和文件引入（source）
-   interrupt: 停止当前过程，执行下一个命令前回到 Debug 模式。
-   finish: 执行完当前脚本或方法调用，回到 Debug 模式。

掌握 Debug 命令后下一个事情就是打断点，可以在函数和文件的相应行号处添加断点：
```
breakadd func [lineNumber] functionName
breakadd file [lineNumber] fileName
breakadd here
```

同样的语法可以用来移除断点，关键字从 `breakadd` 换为 `breakdel`：
```
breakdel func [lineNumber] functionName
breakdel file [lineNumber] fileName
breakdel here
```
还可以按照编号移除 `breakdel {number}` 和移除全部 `breakdel *`。 更详细的参数和命令可以参考 `:help debug`。

#### 小技巧（像chrome控制台一样Debug)
上述日志和调试命令已经足够定位 Vim 脚本的问题了。 下面介绍几个小技巧可以方便一些场景下的操作：

像 Chrome 控制台一样，可以 Debug 某个命令：
```
:debug CommandName
```

也可以直接 debug 某个函数，不需要 Vim 先进入 Debug 模式：
```
:debug call Foo()
```

类似地，日志级别也可以在调用函数时进行设置
```
:20verbose call Foo()
```