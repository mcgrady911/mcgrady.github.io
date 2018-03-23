# Git 骚操作手册





### Git Bisect 快速定位bug

```cmake
# 进行 bisect debug
$ git bisect start

# 标记一个 commit 为 ‘good’
$ git bisect good xxx

# 标记一个 commit 为 ‘bad’
$ git bisect bad xxx

# 过程中需要不断验证应用是否正常
...

# 退出 bisect
$ git bisect reset

# 输出log
git bisect log

# 修改log.txt的bisect标记可重置状态
$ git bisect replay log.txt


```

