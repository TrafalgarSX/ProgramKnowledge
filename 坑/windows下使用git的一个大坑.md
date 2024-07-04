吐血，今天被这个坑的厉害。

一般 Windows 下使用 git 生成 ssh-key, 然后放到 ~/.ssh 文件夹下面， 然后把公钥添加到 github 上就可以了， 但是最近死活无法正常的 git clone , 但是偏偏 `ssh -t git@github.com` 又是正常的， 会返回：

```
Hi TrafalgarSX! You've successfully authenticated, but GitHub does not provide shell access.
```

到了 git clone 的时候却又说  permission denied。 

原来是 windows 10 以后系统自带 openssh, 当在命令行中使用 ssh 时，使用的是 系统自带的， 但是 git 默认使用的是自己的 ssh, 然后在系统默认的 ssh 中添加的密钥不能在 git 自己的 ssh 中使用，然后 git clone 的时候就无法正常运行。

解决方法是设置环境变量：
```powershell
命令行中添加
$env:GIT_SSH='C:\Users\guoya\scoop\shims\ssh.exe'

一般去 windows 的 系统环境变量设置那里设置就可以了。
```

我删除了系统自带的 openssh， 使用 scoop 又安装了一个。