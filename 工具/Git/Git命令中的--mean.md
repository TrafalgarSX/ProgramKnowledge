The double dash `--` in git means different things to different commands, **but in general it separates options from parameters.**

将选项和参数分开

另一个好理解的解释
By convention, -— **terminates processing of command line options** so everything after it is **treated as positional arguments** even if it looks like an option

**Useful if you have a file name that looks like an option.**

#### 取决于不同的子命令
In **git** specifically, the meaning of `--` **depends on which subcommand you are using it with.**

It usually separates subcommand arguments (like the branch name in `git checkout`) from revisions or filenames.

Sometimes it is completely **optional** and used only to prevent an unusual filename from being interpreted as program options.
#### 例子
[In Git, what does \`--\` (dash dash) mean? - Stack Overflow](https://stackoverflow.com/questions/22750028/in-git-what-does-dash-dash-mean)

#### 总结
大部分时候就是因为文件名而区分， 不要把文件名误认为是其他 什么有意义的参数。