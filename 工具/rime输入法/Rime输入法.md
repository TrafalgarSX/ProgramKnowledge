垃圾rime   windows下无法正常使用vim, 遂放弃

### Rime中数据文件分布及作用
-----
共享文件夹路径
| 系统    | 路径                 |
| ------- | -------------------- |
| linux   | /usr/share/rime-data |
| windows | 安装目录/data        |
| macos   | /Library/Input Methods/Squirrel.app/Contents/SharedSupport/                     |

用户文件夹
| 系统    | 路径 |
| ------- | ---- |
| linux   | ~/.config/ibus/rime     |
| windows |%APPDATA%/Rime      |
| macos        | ~/Library/Rime     |

共享文件夹是预设输入方案的源文件，属于发行软件的一部分，不能修改，否则更新有可能会覆盖。

用户文件夹是为用户准备的内容：
| 作用             | 文件名              |
| ---------------- | ------------------- |
| 全局设置         | default.yaml        |
| 发行版设置       | weasel.yaml         |
| 预设输入方案副本 | ${方案}.schema.yaml |
| **安装信息**         | installation.yaml   |
| **用户状态信息**     | user.yaml                    |

部署(编译)方案所生成的二进制文件
| 文件夹作用   | 文件名              |
| ------------ | ------------------- |
| Rime棱镜     | ${方案}.prism.bin   |
| Rime固态词典 | ${词典名}.table.bin |
| Rime反查词典 | ${词典名}.reverse.bin                    |

记录用户写作习惯的文件：
| 文件作用     | 文件名                                       |
| ------------ | -------------------------------------------- |
| **用户词典**     | ${词典名}.userdb/ 或者 ${词典名}.userdb.kct/ |
|  **用户词典快照** |  ${词典名}.userdb.txt                                             |

用户自己设定的：
| 文件作用                     | 文件名              |
| ---------------------------- | ------------------- |
| **用户对全局设置的定制信息**    | default.custom.yaml |
| **用户对预设输入方案的定制信息** | ${方案}.custom.yaml |

标注的文件在清理文件的时候要注意备份

### 输入方案
----- 
一套输入方案：
- 方案定义 ${方案}.schema.yaml  包含输入方案配置信息的YAML文档
- 词典

### 输入法引擎与功能组件
------
>按键消息  -》  后台服务  -》 分配给对应的会话 -》 方案 或 输入引擎处理

会话： 多窗口，多线操作，每一组会话都有一部输入引擎完成按键序列到文件的变换

输入引擎的工作流程：
>加载输入方案 -》 预备功能组件 -》 各就位后进入处理按键消息 -》 处理按键消息   循环处理

看示例代码中的组件：
```yaml
# luna_pinyin.schema.yaml
# ...
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  processors:              # 一、這批組件處理各類按鍵消息                 **重要**
    - ascii_composer       # ※ 處理西文模式及中西文切換
    - recognizer           # ※ 與 matcher 搭配，處理符合特定規則的輸入碼，如網址、反查等
    - key_binder           # ※ 在特定條件下將按鍵綁定到其他按鍵，如重定義逗號、句號爲候選翻頁鍵
    - speller              # ※ 拼寫處理器，接受字符按鍵，編輯輸入碼
    - punctuator           # ※ 句讀處理器，將單個字符按鍵直接映射爲文字符號
    - selector             # ※ 選字處理器，處理數字選字鍵、上、下候選定位、換頁鍵
    - navigator            # ※ 處理輸入欄內的光標移動鍵
    - express_editor       # ※ 編輯器，處理空格、回車上屏、回退鍵等
  segmentors:              # 二、這批組件識別不同內容類型，將輸入碼分段
    - ascii_segmentor      # ※ 標識西文段落
    - matcher              # ※ 標識符合特定規則的段落，如網址、反查等
    - abc_segmentor        # ※ 標識常規的文字段落
    - punct_segmentor      # ※ 標識句讀段落
    - fallback_segmentor   # ※ 標識其他未標識段落
  translators:             # 三、這批組件翻譯特定類型的編碼段爲一組候選文字
    - echo_translator      # ※ 沒有其他候選字時，回顯輸入碼
    - punct_translator     # ※ 轉換標點符號
    - script_translator    # ※ 腳本翻譯器，用於拼音等基於音節表的輸入方案
    - reverse_lookup_translator  # ※ 反查翻譯器，用另一種編碼方案查碼
  filters:                 # 四、這批組件過濾翻譯的結果
    - simplifier           # ※ 繁簡轉換
    - uniquifier           # ※ 過濾重複的候選字，有可能來自繁簡轉換
```

详细了解：
[RimeWithSchemata · rime/home Wiki · GitHub](https://github.com/rime/home/wiki/RimeWithSchemata#rime-with-schemata)
https://github.com/LEOYoon-Tsaw/Rime_collections/blob/master/Rime_description.md

### 码表与词典
----
词典是`translator`的参考书

往往与同名输入方案配套使用。 Rime的词典文件： `<>.dict.yaml`

### 定制指南
----
