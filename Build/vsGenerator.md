### visual studio 各个版本如何生成对应的32位和64位工程
---
![[vsGenerator.png]]

分界线在vs2017, vs2017以前：
生成32位工程
```bash
cmake .. -G"Visual Studio 15 2017"  # 默认生成32
```

生成64位工程
```bash
cmake .. -G"Visual Studio 15 2017 Win64" 
```

vs2019及以后
生成32位工程
```bash
cmake .. -G"Visual Studio 17 2022"  # 默认生成64
```

生成32位工程
```bash
cmake .. -G"Visual Studio 17 2022" -A Win32
```