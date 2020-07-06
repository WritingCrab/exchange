# git

## 1

服务器端建仓库：

```bash
$ git init --bare xxxx.git
```

服务器仓库需要`--bare`这个参数

然后另一面clone的时候，需要注意

```bash
$ git clone jiangdongchao@192.165.53.15:/home/jiangdongchao/work_dir/xxxx.git
```

要注意home前面的`/`还有home后面是用户名。

添加文件

```bash
$ git add *
```

提交（似乎是到本地的什么地方）

```bash
$ git commit -m ""
```

推到服务器，我目前还只知道这种用法，别的以后再说

```bash
$ git push origin master
```

同步代码的时候，别的东西以后再说，现在心情很差！！！！

```bash
$ git pull origin master
```

---

今天发现TortoiseGit真的很好用，commit是在本地，然后push才是到远端，如果在分支上面改了东西的话，首先切换到主分支（然后需不需要拉取来着），然后合并自己的分支，解决冲突，再然后如果自己的提交过多很乱的话，就需要在日志里面合并分支，在bash的话，应该是git rebase -i ，然后编辑吧

---

关于**分支**，比如说需要合并分支的修改，那么需要做的是，首先提交自己的修改到本地，就是无论你做什么，自己的本地修改一定要是提交了的，然后进行拉取，拉取后在本地进行合并，再然后就没有了

关于**变基**，假如多个分支一块工作，然后分别合并之类的，合并之后会有一些分叉，但是如果我们按时间顺序去看的话，会发现每一时刻一定是有一个点点，变基就是将多个分支并在一起，使主线具备各分支的变更，也就是将版本库变得干净了。

---

关于稀疏检出，原因是我在用的时候，经常会出现我需要主动进行Makefile的修改，然后需要把这个修改提交到本地，但是推送之前又需要改回来，于是就会多出几个比较难看的提交，此时就要用到稀疏检出，相当于是将这个文件给忽略掉，但与.gitignore还不一样（这块我还没具体求证），具体操作就是在.git/config中添加

```
sparsecheckout = true
```

然后在.git/info/sparse-checkout中添加**要检出**的条目，所以注意，在这种情况下，一般要检出的是多数，仅有几个文件不需要进行检出，那么首先写个`*`，然后利用反向`!`来标记需要进行稀疏检出的文件即可，这样在你进行关于这个文件的修改的时候，其内容不会发生改动，也就不需要进行提交什么的了。

## 更换推送地址

比如服务器ip换了：

```bash
git remote -v
git remote rm origin
git remote add origin [URL]
```

## git仓库迁移

当前裸仓库位置`/home/ippfcox/repo/aaa.git`

clone的仓库位置`/home/ippfcox/test/aaa`，这里可以是正常仓库也可以是裸版本库（所以是不是直接在裸版本库的目录进行就可以了）

目标是将仓库迁移到已经创建了空仓库的位置`/home/git/repo/AAA.git`

```bash
ippfcox@ubuntu:~/test/aaa$ git push --mirror /home/git/repo/AAA.git
```

## 删除远端分支

git push origin --delete xxx

## 以树状图查看log

git log --graph

## 每条log占一行

git log --pretty=oneline

还有很多可以配置

## patch

