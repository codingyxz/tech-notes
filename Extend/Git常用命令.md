## git checkout

- 切换分支

  ```shell
  git checkout branch_name
  ```

  

- 检出并切换分支

  ```shell
  // new_branch 为新分支名；
  // start_point 为远程分支名
  git checkout -b <new_branch> [<start_point>]
  
  等效于
  
  git branch new_branch origin/new_branch
  git checkout new_branch
  ```

- 新建并切换分支

```shell
// new_branch 新分支名
git checkout -b new_branch
```



-  推送本地分支到远端

```shell
// 推送本地分支local_branch到远程分支 remote_branch并建立关联关系
// 首先本地已经切换到local_branch

// a、远程已有remote_branch分支并且已经关联本地分支local_branch
git push

// b、远程已有remote_branch分支但未关联本地分支local_branch
git push -u origin/remote_branch

// c、远程没有remote_branch分支
git push -u origin local_branch:remote_branch
```



- 放弃修改

  ```shell
  git checkout .               // 放弃工作区中全部修改
  git checkout -- filename     // 放弃工作区中指定文件修改
  
  // 上面两个为简化形式，完整形式如下：
  // 该命令只要用于检出一个文件，如果不填写commit id，则默认从赞从去检出文件，如果暂存区为空，
  // 该文件会回滚到最近一次提交状态，相当于撤销修改
  // 该 commit id 可以是 commit hash，也可以分支名或者tag，其本质上都是commit hash
  git checkout [<commit id>] [--] <filename>
  
  ```



- 本地关联远程分支

```shell
git branch --set-upstream-to=origin/branchName branchName
```



- git合并多次提交为一次提交

  



开发联调过程中会在feature分支多次提交，待提测时需向release合并，为保证release分支清洁，同时方便按照功能进行codereview，可将多次提交合并为一次，然后重新增加commit注释。

- 切换分支到release

  ```shell
  git switch release-4.0.0
  git pull
  ```

- 合并feature-branch分支代码到release

  ```SHE
  git merge --squash <feature-branch>
  // 但此时只是进行了合并，并没有实际commit
  ```

- 提交合并后的代码，并push到远端

  ```SHELL
  git commit -m "message"
  git push
  ```

  



## git worktree



- 新增worktree

```shell
// 新增一个worktree，并指定了其关联的目录是path，关联的分支是<branch>，后者是一个可选项，默认为HEAD分支，
// 如果<branch>已经被关联到了一个worktree，则这次的add会被拒绝执行，可通过 -f 强制执行。
// 
git worktree add <path> [<branch>]
```

- 查看worktree

```shell
// 列出当前仓库已经存在的所有worktree的详细情况，包含每个worktree的关联目录，当前提交点的哈希吗和分支名
git worktree list
// 增加参数改变显示风格
git worktree list --porcelain
```

- 删除worktree

```shell
// 1、首先删除worktree的关联目录
// 2、清除worktree的信息
git worktree prune
```



##  git log

```shell
// 查看各branch之间的关系图
git log --graph --decorate --oneline --simplify-by-decoration --all

说明：
--decorate：标记会让git log显示每个commit的引用(如:分支、tag等)
--oneline：一行显示
--simplify-by-decoration：只显示被branch或tag引用的commit
--all：表示显示所有的branch，这里也可以选择，比如我指向显示分支ABC的关系，则将--all替换为branchA branchB branchC

```





## git tag

```shell
// 新建tag
git tag -a tagname -m "desc"
// 推送至远端
git push origin tagname
// 查看tag详细信息
git show tagname
// 删除本地tag
git tag -d tagname
// 删除远程tag
git push origin :/refs/tags/tagname
```



## git delete

```shell
// 删除本地分支
git branch -d branch_name

// 删除远程分支
git push origin --delete branch_name

// 清楚本地无效分支（远程已删除本地没删除的分支）
git fetch -p
```



## Git切源

```
// 方式一
// 直接切换源
git remote set-url origin [remote_url]

// 方式二
// 删除源
git remote rm origin
// 查看远程仓库地址
git remote -v
// 添加新的源
git remote add origin [remote_url]

=================================================

// 同步远程分支
git pull
// 本地分支关联远程分支
git branch --set-upstream-to=origin/[branch_name]
// 删除所有本地分支
 git branch | xargs git branch -D



```

