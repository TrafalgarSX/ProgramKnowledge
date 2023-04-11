### vim常用插件及功能

- vim-orgmode 
```
vim-orgmode is a file type plugin for keeping notes, maintaining TODO lists, and doing project planning with a fast and effective plain-text system, It is also an authoring and publishing system.

(http://orgmode.org/) It contains all basic features and commands, along with important hints for customization.

To start create a new file with the extension ".org".

How to use it in vim?
[vim-orgmode/orgguide.txt at master · jceb/vim-orgmode · GitHub](https://github.com/jceb/vim-orgmode/blob/master/doc/orgguide.txt)
```
- calendar.vim
```
Press E key to view the event list, and T key to view the task list. Also, press ? key to view a quick help.

:Calendar

:Calendar 2000 1 1

:Calendar -view=year

:Calendar -view=year -split=vertical -width=27

:Calendar -view=year -split=horizontal -position=below -height=12

:Calendar -first_day=monday

:Calendar -view=clock

https://github.com/itchyny/calendar.vim
```
- vim-speeddatin
```
增加或减少日期、时间、数字和叙述
<C-A> <C-X>
Several date, time, and datetime formats are included. Additional formats can be defined in a strftime-like syntax with the :SpeedDatingFormat command.

Existing Vim semantics are preserved. <C-A> and <C-X> accept a count, and plain number incrementing is used if no date format is matched.

Use of <C-A>/<C-X> in visual mode enables incrementing several lines at once. Blank spots are filled by incrementing the match from the previous line, allowing for creation of sequences (1, 2, 3; 2000-10-30, 2000-10-31, 2000-11-01).
```
- vim-table-mode
```
To start using the plugin in the on-the-fly mode use `:TableModeToggle` mapped to <Leader>tm by default (which means \ t m if you didn't override the by `:let mapleader = ','` to have , t m).

Tip: You can use the following to quickly enable / disable table mode in insert mode by using `||` or `__`:
Enter the first line, delimiting columns by the `|` symbol. The plugin reacts by inserting spaces between the text and the separator if you omit them:
In the second line (without leaving Insert mode), enter `|` twice. The plugin will write a properly formatted horizontal line:
```

```viml
function! s:isAtStartOfLine(mapping)
  let text_before_cursor = getline('.')[0 : col('.')-1]
  let mapping_pattern = '\V' . escape(a:mapping, '\')
  let comment_pattern = '\V' . escape(substitute(&l:commentstring, '%s.*$', '', ''), '\')
  return (text_before_cursor =~? '^' . ('\v(' . comment_pattern . '\v)?') . '\s*\v' . mapping_pattern . '\v$')
endfunction

inoreabbrev <expr> <bar><bar>
          \ <SID>isAtStartOfLine('\|\|') ?
          \ '<c-o>:TableModeEnable<cr><bar><space><bar><left><left>' : '<bar><bar>'
inoreabbrev <expr> __
          \ <SID>isAtStartOfLine('__') ?
          \ '<c-o>:silent! TableModeDisable<cr>' : '__'
```

```
let g:table_mode_corner='|'


|-----------------|--------------------------|------------|
| name            | address                  | phone      |
|-----------------|--------------------------|------------|
| John Adams      | 1600 Pennsylvania Avenue | 0123456789 |
|-----------------|--------------------------|------------|
| Sherlock Holmes | 221B Baker Street        | 0987654321 |
|-----------------|--------------------------|------------|
```
- vimspector 和 neodebug
```
Vimspector 和 Neovim 的 NeoDebug 都是 Vim/neovim 下的调试插件，它们的目标都是为了让 Vim/neovim 成为一个更好的编辑器调试器。

Vimspector 是一个功能更加全面的插件，它支持多种语言和调试后端，包括 GDB、LLDB、Python PDB、Node.js 调试器等。它还支持在 Vim 中使用调试命令，如断点、单步执行、监视变量等，并提供了丰富的文档和配置选项。

NeoDebug 则是 NeoVim 下的调试插件，它通过使用 Neovim 的特性，如异步执行和通信，实现了一个轻量级的调试器。它仅支持 Python 和 Ruby，但它的优点在于易于安装和使用，它还提供了很多定制选项来满足不同的需求。此外，NeoDebug 还有一个可视化界面，可以方便地查看变量、堆栈和调用栈信息。

总的来说，Vimspector 是一个功能更加全面和强大的调试插件，适用于需要使用多种调试后端和语言的场景。NeoDebug 则是一个轻量级的调试插件，适用于简单的 Python 或 Ruby 项目调试。
```
- deoplete和asyncomplete
```
Deoplete和Asyncomplete都是Vim的自动补全插件，但它们有一些不同之处。

Deoplete是一个由Python编写的异步自动补全框架，具有高度的可定制性和扩展性，可以通过多个来源提供自动补全建议。它可以与多个语言的补全引擎集成，包括Python、JavaScript、CSS、HTML、PHP等。另外，Deoplete的补全建议可以实时刷新，用户也可以自定义补全源、补全触发器等。

Asyncomplete也是一个由Python编写的异步自动补全框架，它的主要特点是高度可配置和可扩展性。与Deoplete类似，它可以支持多种语言和多个自动补全引擎。不同之处在于，Asyncomplete可以与任何支持Vim 8+的自动补全引擎集成，并且提供了很多自定义配置项，用户可以自由地定义补全引擎、补全触发器等。

总之，Deoplete和Asyncomplete都是强大的自动补全插件，可以根据用户的需求进行定制。它们的异同主要在于其架构、可扩展性和定制性。
```
- asyncomplete是什么，如何使用？
```
asyncomplete.vim 是一个异步自动完成框架，支持多个后端源，包括 Vim 自带的 omni、用户自定义的语言服务器等。要配置 asyncomplete.vim，可以按照以下步骤进行：

安装 asyncomplete.vim 插件，可以使用插件管理器（如 vim-plug、Vundle 等）进行安装。

确保已安装必要的软件包，如 Python、Node.js 等。

在 .vimrc 文件中设置 asyncomplete.vim 的配置参数，例如

let g:asyncomplete_timeout = 1000
let g:asyncomplete_auto_popup = 1
let g:asyncomplete_pop_selection = 1

其中，g:asyncomplete_timeout 参数设置自动完成菜单显示的超时时间（单位为毫秒），g:asyncomplete_auto_popup 参数设置是否自动弹出自动完成菜单，g:asyncomplete_pop_selection 参数设置是否在选中菜单项时自动补全。

配置后端源： 以前是使用 omni(vim自带后端源) ycm等，有了lsp后使用 asyncomplete-lsp

在 Vim 中使用 asyncomplete.vim。例如，在插入模式下，输入 Ctrl-x Ctrl-o 可以强制弹出自动完成菜单。当自动完成菜单弹出时，可以使用 Tab 键进行选中，也可以使用 Ctrl-n 或 Ctrl-p 键进行上下选择。
```
- vim-lsp vim-lsp-settings asyncomplete-lsp.vim asyncomplete.vim 如何一起工作
1.  安装 `asyncomplete.vim`, `vim-lsp`, `vim-lsp-settings`, `asyncomplete-lsp.vim` 插件，可以使用 `vim-plug` 进行安装。
    
2.  配置 `vim-lsp`：
    在 `vimrc` 文件中加入以下代码：
```
" 启用LSP
set completeopt=menuone,longest,preview
call lsp#register_server({
      \ 'name': 'clangd',
      \ 'cmd': {server_info->[&shell, &shellcmdflag, 'clangd', '--background-index']},
      \ 'workspace_config': {
        \ 'clangd': {
          \ 'arguments': [
            \ "--compile-commands-dir=build",
            \ "--header-insertion=never",
          \ ],
        \ },
      \ },
      \ 'whitelist': ['c', 'cpp'],
      \ })
```
3. 配置 `vim-lsp-settings`：
在 `vimrc` 文件中加入以下代码：
```
let g:lsp_settings = {}
let g:lsp_settings['clangd'] = {
      \ 'cmd': ['clangd'],
      \ 'root_uri': 'file:///path/to/root',
      \ 'initialization_options': {
        \ 'clangd': {
          \ 'completion': {
            \ 'includeBlacklist': ['::'],
          \ },
        \ },
      \ },
      \ 'settings': {
        \ 'clangd': {
          \ 'command': ['clangd', '--background-index'],
        \ },
      \ },
      \ }
```
其中 `'root_uri'` 参数指定项目的根目录，`'initialization_options'` 和 `'settings'` 参数可以根据需要进行设置。
4. 配置 asyncomplete-lsp.vim：
在 vimrc 文件中加入以下代码：
```
call asyncomplete#sources#lsp#register()
```
这会注册 `asyncomplete-lsp` 插件作为 `asyncomplete` 的补全源。

完成上述配置后，当使用 `asyncomplete.vim` 进行补全时，会自动调用 `vim-lsp` 的服务进行补全。如果需要配置 `asyncomplete.vim` 的其它参数，可以参考它的文档进行设置。

- `let g:lsp_use_lua = has('nvim-0.4.0') || (has('lua') && has('patch-8.2.0775')): 这个配置指定了是否使用 Lua 作为 LSP 的后端，当 Vim 版本大于等于 0.4.0 或者存在 Lua 并且有 8.2.0775 及以上的 patch 时，则使用 Lua 作为 LSP 的后端。
- `let g:lsp_completion_documentation_enabled = 0: 这个配置指定了是否开启自动补全时的文档提示功能。
- `let g:lsp_diagnostics_enabled = 0: 这个配置指定了是否开启语法检查和错误提示功能。
- `let g:lsp_diagnostics_signs_enabled = 0: 这个配置指定了是否开启语法检查和错误提示功能时的标记符号。
- `let g:lsp_diagnostics_highlights_enabled = 0: 这个配置指定了是否开启语法检查和错误提示功能时的高亮提示。
- `let g:lsp_document_code_action_signs_enabled = 0: 这个配置指定了是否开启代码自动修复提示功能。
- `let g:lsp_signature_help_enabled = 0: 这个配置指定了是否开启函数参数提示功能。
- `let g:lsp_document_highlight_enabled = 1: 这个配置指定了是否开启文档高亮功能。
- `let g:lsp_preview_fixup_conceal = 1: 这个配置指定了是否开启 LSP 修复功能的代码隐藏。
- `let g:lsp_hover_conceal = 1: 这个配置指定了是否开启 LSP 悬浮提示功能的代码隐藏。

```
💡使用Lua作为LSP的后端，意味着使用了neovim的lua插件开发接口来实现LSP服务器。通过使用lua插件，可以提供更高效和可定制化的LSP支持。相比其他的LSP后端，使用Lua作为后端可以在性能上得到更好的表现。此外，由于lua是一种高度可定制化的语言，这意味着使用lua作为LSP后端可以更容易地编写自定义插件和扩展。
```

- `g:asyncomplete_min_chars`: 控制自动完成的最小字符数。默认为1，表示输入至少一个字符后才会触发自动完成。将其设置为0，则输入任何字符都会触发自动完成。
- `g:asyncomplete_auto_completeopt`: 用于控制 `<C-n>` 和 `<C-p>` 是否用于补全菜单项。默认为1，表示  `<C-n>` 和 `<C-p>` 不会用于补全菜单项。将其设置为0，则`<C-n>` 和 `<C-p>` 也可以用于补全菜单项。
------
- vim-notes  暂时不需要用
The vim-notes plug-in for the [Vim text editor] vim makes it easy to manage your notes in Vim

- vim-ingo-library
This library contains common autoload functions that are used by almost all of my plugins.

- ctrlsf.vim 具体怎么用  待测试 现在好像并没有映射快捷键
[GitHub - dyng/ctrlsf.vim: A text searching plugin mimics Ctrl-Shift-F on Sublime Text 2](https://github.com/dyng/ctrlsf.vim)
- gutentags_plus.vim  查看怎么用  
- asyncomplete.vim asyncomplete-buffer/tags/user等插件如何搭配使用
```

asyncomplete.vim是一个异步自动补全框架，它本身并没有提供任何自动补全功能，需要通过其他插件来提供具体的自动补全功能。而asyncomplete-lsp则是asyncomplete.vim提供的一个后端，用于集成LSP服务器，提供基于LSP协议的自动补全功能。asyncomplete-buffer和asyncomplete-tags则是asyncomplete.vim提供的两个后端，分别用于基于当前缓冲区和tags文件提供自动补全功能。asyncomplete-user则是asyncomplete.vim提供的一个后端，用于用户自定义自动补全功能。

asyncomplete.vim插件支持同时注册多个后端，这样可以让它们协同工作，提供更全面的自动完成功能。

要注册多个后端，可以在vim的配置文件中为每个后端添加配置

" LSP
let g:asyncomplete_sources = []
let g:asyncomplete_sources += ['asyncomplete#sources#neosnippet']
let g:asyncomplete_sources += ['asyncomplete#sources#buffer']
let g:asyncomplete_sources += ['asyncomplete#sources#tags']
let g:asyncomplete_sources += ['asyncomplete#sources#omni']
let g:asyncomplete_sources += ['asyncomplete#sources#lsp']
let g:asyncomplete#lsp#register('clangd', ['c', 'cpp'])

```
- vim-lsp-settings
```
let g:lsp_c_lang_settings = {
  \ 'clangd': {
  \   'cmd': ['clangd', '--background-index', '--clang-tidy'],
  \   'settings': {
  \     'clangd': {
  \       'arguments': [
  \         '-isystem', '/usr/include/c++/8',
  \         '-isystem', '/usr/include/x86_64-linux-gnu/c++/8',
  \         '-isystem', '/usr/include/c++/8/backward',
  \         '-isystem', '/usr/include/clang/7/include',
  \         '-isystem', '/usr/include/x86_64-linux-gnu/clang/7/include',
  \         '-I', '/path/to/include',
  \         '-std=c++17',
  \         '-Wall',
  \         '-Wextra',
  \         '-Werror',
  \       ],
  \     },
  \   },
  \ },
  \}

```