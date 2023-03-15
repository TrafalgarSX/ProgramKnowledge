### git pull相关
* fatal:refusing to merge unrelated histories # 拒绝合并无关历史 
	```git
	git pull origin master --allow-unrelated-histories
	```
^4f6a22

```
# 偏离的分支，如何调和
% git pull
hint: Pulling without specifying how to reconcile divergent branches is
hint: discouraged. You can squelch this message by running one of the following
hint: commands sometime before your next pull:
hint:
hint: git config pull.rebase false # merge (the default strategy)
hint: git config pull.rebase true # rebase
hint: git config pull.ff only # fast-forward only
hint:
hint: You can replace “git config” with “git config --global” to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
Already up to date.

```
* `git config pull.rebase false`   #默认策略