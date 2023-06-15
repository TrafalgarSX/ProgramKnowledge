### 如何使用emacs运行gdb进行debug
#### emacs进行调试的一些快捷键
[[emacs]]
在.emacs中的一些gdb配置
```
(global-set-key [M-left] 'windmove-left)
(global-set-key [M-right] 'windmove-right)
(global-set-key [M-up] 'windmove-up)
(global-set-key [M-down] 'windmove-down)

(global-set-key [f5] 'gud-run)
(global-set-key [S-f5] 'gud-cont)
(global-set-key [f6] 'gud-jump)
(global-set-key [S-f6] 'gud-print)
(global-set-key [f7] 'gud-step)
(global-set-key [f8] 'gud-next)
(global-set-key [S-f7] 'gud-stepi)
(global-set-key [S-f8] 'gud-nexti)
(global-set-key [f9] 'gud-break)
(global-set-key [S-f9] 'gud-remove)
(global-set-key [f10] 'gud-until)
(global-set-key [S-f10] 'gud-finish)

(global-set-key [f4] 'gud-up)
(global-set-key [S-f4] 'gud-down)

(setq gdb-many-windows t)


(require 'xt-mouse)
(xterm-mouse-mode)
(require 'mouse)
(xterm-mouse-mode t)
(defun track-mouse (e))

(setq mouse-wheel-follow-mouse 't)

(defvar alternating-scroll-down-next t)
(defvar alternating-scroll-up-next t)

(defun alternating-scroll-down-line ()
  (interactive "@")
    (when alternating-scroll-down-next
      (scroll-down-line))
    (setq alternating-scroll-down-next (not alternating-scroll-down-next)))

(defun alternating-scroll-up-line ()
  (interactive "@")
    (when alternating-scroll-up-next
      (scroll-up-line))
    (setq alternating-scroll-up-next (not alternating-scroll-up-next)))

(global-set-key (kbd "<mouse-4>") 'alternating-scroll-down-line)
(global-set-key (kbd "<mouse-5>") 'alternating-scroll-up-line)
```

- F5 运行，Shift + F5 继续
- F7/F8 代码级单步， 以及 Shift-F7/F8 指令级单步
- F9 设置断点，Shift-F9 删除断点
- F10 跳出循环，Shift-F10 跳出函数
- F4 移动到上一个调用栈帧，Shift-F4移动到下一个

😅👉6月1号， 用emacs进行gdb调试的时候发现时间长了操作就忘了。

现在记录一下详细操作：
```
gdbtool libEtCommon.so
现在进入到emacs中
如果没有图形界面，说明配置文件没有生效， 可以在emacs中使用命令让配置文件生效
M-x load-file <enter>
输入配置文件路径：  ~/.emacs

1.然后进入图形界面
2.先用f9在需要打断点的地方打断点

3.然后左上的gdb窗口, 回车， 可以看到前面有gdb>,   输入attach pid
4.然后等待调用动态库的程序调用动态库函数

5.然后输入continue
6.然后就可以了看到进入断点了。

如果jdb不好用， 就用Scanner 输入卡住
```


#### emacs的一些基本操作
这一步需要学习emacs的基本操作（大概需要30分钟）

1.  先记住C-x C-c（Ctrl-x Ctrl-c）（两个连续的组合键）为退出
2.  运行emacs 按C-h t进入基础教走程跟着做一遍（[中文教程](https://beefnoodles.cc/assets/doc/emacs_tutorial.txt)）
3.  补充几条教程里没有提到的命令
    -   多窗口的命令
        -   C-x 1 只保留当前窗口
        -   C-x 2 水平分屏
        -   C-x 3 垂直分屏
        -   C-x 0 关闭当前窗口
        -   C-x o 切换窗口
    -   其他常用快捷键
        -   C-x k关闭缓冲区


其中一个窗口就会被切换成反汇编窗口：
`gdb-display-disassembly-buffer`

#### 可能会遇到的情况
1.  窗口乱了怎么办？执行两次M-x gdb-ma[ny-window]
2.  在用鼠标中间键点击浏览代码时，可能会触发gdb命令窗口被代码占据的bug，这时选择被占据的窗口用C-x k关闭buffer即可
3.  我想临时把一个窗口放大，然后回到多窗口。用[winner-mode](http://www.ergoemacs.org/emacs/emacs_winner_mode.html)。运行M-x winner-mode打开winner-mode，C-x 1关闭当前窗口以外窗口，然后通过C-c ←返回。

https://www.cnblogs.com/xiaoshiwang/p/11912383.html
[如何优雅地使用终端进行调试emacs-gdb | 现代魔法](https://beefnoodles.cc/2019/05/29/emacs-gdb/)