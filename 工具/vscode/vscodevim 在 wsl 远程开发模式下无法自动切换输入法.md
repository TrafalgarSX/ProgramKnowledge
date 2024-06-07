
[vscodevim 在 wsl 远程开发模式下无法自动切换输入法 | listenerri's blog](https://listenerri.com/2023/06/26/vscodevim-%E5%9C%A8-wsl-%E8%BF%9C%E7%A8%8B%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B8%8B%E6%97%A0%E6%B3%95%E8%87%AA%E5%8A%A8%E5%88%87%E6%8D%A2%E8%BE%93%E5%85%A5%E6%B3%95/)

vscode 下的 vim 扩展插件([vscodevim](https://github.com/VSCodeVim/Vim)) 在 windows 下使用 wsl/ssh 远程开发时，设置的自动切换 windows 输入法无效，分析了下源码发现跟自动切换输入法相关的几个设置项，其 scope 被设置为了 machine：

[https://github.com/VSCodeVim/Vim/blob/15dbe0b415f269a5be14dc74651a69b9334fba14/package.json#L1064](https://github.com/VSCodeVim/Vim/blob/15dbe0b415f269a5be14dc74651a69b9334fba14/package.json#L1064)

涉及到的设置项有：

- vim.autoSwitchInputMethod.defaultIM
- vim.autoSwitchInputMethod.switchIMCmd
- vim.autoSwitchInputMethod.obtainIMCmd

翻阅 vscode 插件开发文档后了解到，当某个配置项的 scope 属性被设置为 machine 时，这意味着此配置项只在 vscode 核心具体工作的那台机器上生效，本地开发时自然是读取正常，但当进行 wsl/ssh 远程开发时，vscode 真正的核心(vscode-server)是在远程主机里工作的，本地可见的 vscode 只是个客户端或者说个空外壳，所以上述几个配置应该在远程主机中设置。

同时 vim 扩展插件的插件类型(extensionKind)被设置为了 ui 类，也就是说 vim 插件只工作 vscode UI 所在的地方，即只工作在客户端/空外壳上。

综上所述，进行远程开发时，vim 插件(其他插件也是这样)中提供的配置项，scope 属性为 machine 的配置需要在远程主机中设置，vscode 提供了这种操作的界面，如下图所示：

![remote-vim-settings](https://listenerri.com/2023/06/26/vscodevim-%E5%9C%A8-wsl-%E8%BF%9C%E7%A8%8B%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B8%8B%E6%97%A0%E6%B3%95%E8%87%AA%E5%8A%A8%E5%88%87%E6%8D%A2%E8%BE%93%E5%85%A5%E6%B3%95/remote-vim-settings.png)

需要注意的是，设置项的值应该跟在 windows 本地做开发的值是一样的，因为 vim 插件只工作在本地，只是需要从远程主机上读取这些配置(这一点设计的很奇怪)。


```json
{
    "vim.autoSwitchInputMethod.defaultIM": "1033",
    "vim.easymotion": true,
    "vim.autoSwitchInputMethod.obtainIMCmd": "c:/Users/guoya/scoop/apps/im-select/current/im-select.exe",
    "vim.autoSwitchInputMethod.switchIMCmd": "c:/Users/guoya/scoop/apps/im-select/current/im-select.exe {im}"
}
```

我在 D 盘下也有 im-select.exe， 但是配置成 D:/im-select.exe 还有不生效， 这可能是因为我配置 wsl 启动的时候默认不 mount D盘，只 mount C 盘的原因（D盘太大了）， mount D 盘wsl 启动太慢了。