# git用法学习

[toc]



### 创建版本库，添加、提交文件

#### 创建版本库 

- 创建文件夹，文件夹中执行：

  ```bash
  git init
  ```

#### 添加文件

- 仓库内新建文件（可多个），再执行（一次执行）：

  ```bash
  git add file1 file2 ... filen
  ```

#### 提交文件

- 多次添加，一次提交，执行：

  ```bash
  git commit -m "描述"
  ```

### 版本间穿梭

#### 查看状态

- 查看工作区（当前仓库）的状态;

- 查看文件有无被修改过

  ```bash
  git status
  git diff
  ```

#### 版本回退

![Imgur](imgs/版本回退.png)



#### 工作区与暂存区

![Imgur](imgs/工作区与暂存区.jpg)



#### 撤销修改

<img src="imgs/撤销修改.png" alt="image-20210403215346894" style="zoom:150%;" />



#### 删除文件

```bash
git rm file #此为从git版本库里将文件删除，删除后还需要提交
```

### 远程仓库

#### 添加远程仓库

<img src="imgs/添加远程仓库.jpg" alt="Imgur"  />

### 分支管理

#### 创建与合并分支

![Imgur](imgs/创建与合并分支.png)

#### 解决冲突

![](imgs/冲突时分支合并.png)

当git无法自动合并分支时，必须首先解决冲突，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用***git log --graph***命令可以看到分支合并图

#### 分支管理策略

![](imgs/分支管理策略.png)

#### Bug分支

![](imgs/Bug分支.png)

#### Feature分支

开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并的分支，可以通过***git branch -D branch_name***来强行删除；

但是如果提前获知待丢弃分支的commit id，强行删除后仍然可以通过***git reset***来移动到该提交。

#### 多人协作

![](imgs/多人协作.png)

#### Rebase

同步远程最新代码，**git push**产生冲突，说明有人先你一步同步了他的本地代码到远程;

此时使用**git pull**拉取远程的提交并于本地的提交进行合并，并自动提交；

如果**git pull**时有冲突，则需要手动解决并手动提交；

合并之后会产生merge的记录，使用**git log**查看如下所示:

![](imgs/rebase1.png)

产生了历史分叉，使用**git rebase**可以将原本分叉的提交合并为一条直线，如下图所示；

此时本地的修改不再基于d1be385，而是基于f005ed4，即所谓**变基**。

![](imgs/rebase2.png)

然后可以**git push**同步本地提交到远程，此时远程的提交历史也会是一条直线。

- rebase操作可以把本地未push的分叉提交历史整理成直线
- rebase的目的是使我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比

### 标签管理

#### 创建标签

![](imgs/创建标签.png)

#### 操作标签

![](imgs/操作标签.png)

### 自定义git

#### 忽略特殊文件

```bash
#排除所有.开头的隐藏文件
.*
# 排除所有.class文件:
*.class

# 不排除.gitignore和App.class:
!.gitignore
!App.class
```

- 忽略某些文件时， 需要编写`.gitignore`
- `.gitignore`文件本身需要放到版本库里，并且可以对其进行版本管理

#### 配置别名

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.last 'log -1'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

- 配置Git的时候，加上`--gobal`是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。
- 每个仓库自己的配置文件位于当前仓库的.git/config中
- 当前用户的Git配置文件放在用户主目录下的一个隐藏文件`.gitconfig`中

***OK, finished***:smiley_cat:

