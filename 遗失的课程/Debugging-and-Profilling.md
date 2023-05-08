### Debugging
---
#### RGB
bash输出颜色。
[[bash#放到你的 ~/.bashrc 配置文件中，给 man 增加漂亮的色彩高亮]]

#### log
`/var/log`路径下有很多log. 例如`/var/log/systemlog`
```bash
logger "Hello Logs"

log show --last 1m | grep Hello

```

#### Debugger
```bash
python -m ipdb program.py
```

#### Static Analyzer
`pyflaks`  `mypy`

#### Profiling
- CPU profilers
- tracing profilers(function execute time)
- sampling profilers
- line profilers

```
python -m cProfile -s tottime  some.py 

kernprof -l -v some.py
```

#### Memory Profiling
```
python -m memory_profiler mem.py
```

#### Strace
this stress program is just running in the CPU, and it's just a program to just hog one CPU and like test that you can hog the CPU. 
```
perf stat stress -c 1

sudo perf report
```

#### Profilers
sampling profilers

Flame Graph

call graph: 函数调用关系和调用次数

#### Allsoft
