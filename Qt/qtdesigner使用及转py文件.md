### 如何使用qtdesigner
有几种使用qtdesigner的方式：
1. 下载安装Qt平台， 里面自带Qt Designer Studio。
2. python安装PySide6或者PyQt6-tools，里面也自带designer

### 如何将qtdesigner画出的ui文件转py文件
在命令行执行：
```
python.exe -m PyQt6.uic.pyuic $FileName$  -o $FileNameWithoutExtension$.py

pyuic6  xxx.ui -o xxx.py
```