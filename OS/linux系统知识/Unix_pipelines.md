The Unix philosophy lays emphasis on building software that is simple and extensible.
software should be able to **work with other programs through a common interface – a text stream.** This is one of the core philosophies of Unix which makes it so powerful and intuitive to use.

### Example1:根据git仓库的提交数输出作者排行榜

start with a simple one:
display a list of authors/contributors of a git repo sorted based on the number of commits and sort the list in descending order
```

git log --format='%an' | sort | uniq -c | sort -nr
```

### Example2: Browse memes from /r/memes and set your wallpaper from /r/earthporn
Did you know that you can just append “`.json`” to a reddit url to get a json response instead of the usual html? This allows for a world of possibilities!


### 参考
1. [The beauty of Unix pipelines](https://prithu.dev/posts/unix-pipeline/)
2. [The Beauty of Unix Pipelines | Hacker News](https://news.ycombinator.com/item?id=23420786)