---
title: git笔记
date: 2018-2-8 14:58:10
tag: 
  - git
---

## 版本回退

```
git log [--pretty=oneline]
```

先查看下日志，查看各个版本的`mode id`

```
git reset --hard (HEAD^ | modeid)
```

`HEAD`表示当前版本，`HEAD^`表示上一个版本，`HEAD~100`表示之前的100个版本，也可以通过`modeid`来回退版本

```
git reflog
```

记录你的每一次命令，在我们回退后又后悔的时候可以查看各个版本的`mode id`

<!-- more -->

## 工作区和暂存区

- 工作区：文件目录

- 暂存区：`.git`文件里的，git还为我们创建了第一个分支`master`，以及一个指向master的指针叫`HEAD`

前面讲了我们把文件往Git版本库里添加的时候，是分两步执行的：

- 第一步是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；

- 第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以，现在，git commit就是往master分支上提交更改。

你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

## 管理修改

`git add .` ：他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，***但不包括被删除的文件***。

`git add -u` ：他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u ***不会提交新文件***（untracked file）。（git add --update的缩写）

`git add -A` ：是上面两个功能的合集（git add --all的缩写）


## 撤销修改

```
git checkout -- file
```

丢弃工作区的修改，让这个文件回到最近一次git commit或git add时的状态；命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令

```
git reset HEAD file
```

丢弃暂存区的修改（unstage）重新放回工作区

当然如果已经提交到版本库或远程仓库，你就要回退版本了。

## 删除文件

```
git rm file
```

从版本库中删除文件，然后`git commit`就好了

```
git checkout -- file
```

从版本库中获取误删的文件

## 远程仓库

### 创建github远程仓库

- 创建SSH Key：

在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

```
$ ssh-keygen -t rsa -C "youremail@example.com"
```

生成两个文件：id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人

- 登陆GitHub，打开“Account settings”，“SSH Keys”页面：

然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容使用命令git push -u origin master第一次推送master分支的所有内容；此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；

## 分支

- 创建一个名为`dev`的分支：

```
git checkout -b dev
//相当于以下两句
git branch dev
git checkout dev
```

git checkout命令加上-b参数表示创建并切换，然后用`git branch`查看当前分支：

```
$ git branch

* dev
  master
```

`*`号表示当前分支。

- 合并`dev`分支到`master`：

```
$ git merge dev
Updating d17efd8..fec145a
Fast-forward
 readme.txt |    1 +
 1 file changed, 1 insertion(+)
```

git merge命令用于合并指定分支到当前分支，注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。

合并完成后，就可以放心地删除dev分支了：

```
$ git branch -d dev
Deleted branch dev (was fec145a).
```

如果有冲突，需要手动解决。可以用`git log`命令查看分支的合并情况：

```
$ git log --graph --pretty=oneline --abbrev-commit
*   59bc1cb conflict fixed
|\
| * 75a857c AND simple
* | 400b400 & simple
|/
* fec145a branch test
...
```

### 分支管理策略

使用`Fast Forward`模式下，删除分支后，分支信息也会没掉：

```
$ git merge --no-ff -m "merge with no-ff" dev
```

禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息，--no-ff参数，表示禁用Fast forward

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

### Bug分支

- `stash`功能：

可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

```
$ git stash
Saved working directory and index state WIP on dev: 6224937 add merge
HEAD is now at 6224937 add merge
```

现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 2 commits.
$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt |    2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git branch -d issue-101
Deleted branch issue-101 (was cc17032).
```

现在，是时候接着回到dev分支干活了！

```
$ git checkout dev
Switched to branch 'dev'
$ git status
# On branch dev
nothing to commit (working directory clean)
```

工作区是干净的，刚才的工作现场存到哪去了？用git stash list命令看看：

```
$ git stash list
stash@{0}: WIP on dev: 6224937 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；

另一种方式是用git stash pop，恢复的同时把stash内容也删了：

```
$ git stash pop
```

再用git stash list查看，就看不到任何stash内容了，可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令：

```
$ git stash apply stash@{0}
```

### Feature分支

删除分支：

```
git branch -d feature-vulcan
```

强制删除分支：

```
git branch -D feature-vulcan
```

### 多人协作

远程仓库的默认名称为`origin`；

抓取远程分支：

```
git checkout -b dev origin/dev
```

设置本地`dev`分支和远程`origin/dev`分支的链接：

```
git branch --set-upstream dev origin/dev
```

提交到远程分支：

```
git push origin dev
```

如果远程分支和你冲突了，要先`git pull`把最新的提交从`origin/dev`中拿下来，本地解决冲突后，再推送

因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin branch-name推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

## 标签管理

标签是版本库的一个快照。

### 创建标签

打标签：`git tag <name>`；

查看所有标签：`git tag`；

在某个提交版本打标签：`git tag <name> <commitId>`；

查看标签：`git show v0.9`；

创建带有说明的标签，用-a指定标签名，-m指定说明文字：`git tag -a v0.1 -m "version 0.1 released" 3432525`；

通过-s用私钥签名一个标签：`git tag -s v0.2 -m "signed version 0.2 released" fec145a`；

### 操作标签

命令`git push origin <tagname>`可以推送一个本地标签；

命令`git push origin --tags`可以推送全部未推送过的本地标签；

命令`git tag -d <tagname>`可以删除一个本地标签；

命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

### 移除文件的版本控制

如果你想把一个文件从版本控制中移除，并且保留本地的文件，首先需要把这个文件加入到gitignore文件中。然后执行以下命令就可以了。

```
git rm file_path --cached
```

以上命令将file_path所代表的文件从版本控制中删除，并保留本地文件，此外还要进行commit操作才能将服务器端的文件删掉。如果想把一个文件夹从版本控制中删除并保留本地的文件，只需在上述命令的基础上加上-r参数，即

```
git rm -r folder_path --cached
```

如果想把所有gitignore中的文件从版本控制中删除的话，需要执行以下两个命令，即先移除所有文件，再执行添加所有文件（这次会忽略gitignore中的文件）。

```
git rm -r . --cached
git add .
```
