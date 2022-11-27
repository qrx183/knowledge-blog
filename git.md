# Git学习

常用命令

1. 获取git仓库

   1. 将本地尚未进行版本控制的本地目录转换为Git仓库

      ```git
      // 首先切换到制定目录下
      git init
      
      ```

   2. 从其他服务器上克隆一个已存在的Git仓库

      ```
      git clone [要克隆的远程仓库的地址] 在本地存储的文件名
      ```

2. 记录每次更新到仓库

   1. 检查当前文件状态 git status
      1. unchecked files表示未被跟踪的文件
      2. changes to be commited表示这些文件处于暂存状态
      3. changes not staged for commit表示已跟踪的文件的内容发生了修改
   2. 跟踪新文件 git add 文件名
   3. 暂存已修改的文件,包括未跟踪和已跟踪的文件 git add 文件名
   4. 查看更详细的文件状态 git diff
      1. git diff默认比较的是当前文件与暂存区域之间的差异
      2. git diff --staged比较的是已暂存文件和已提交文件之间的差异
   5. 提交更新 git commit -m '提交信息'
   6. 跳过暂存区域直接提交 git commit -a -m '提交信息'
   7. 将文件从本地移除 git rm 文件名
   8. 将文件从暂存区域移除 git rm --cached 文件名
   9. 重命名文件 git mv oldfile newfile

3. 查看该仓库的提交历史 git log

   1. 查看提交信息的变化 git log -p

4. 撤销操作

   1. 提交完某个文件后发现还有些文件忘记提交或提交信息写错可以用 git commit --amend来再次提交并修改上次提交的提交信息
   2. 取消暂存 git restore --stageed 文件名
   3. 将文件恢复成最初的样子 git checkout --文件名             前提是文件在工作区而不是暂存区

5. 远程仓库的使用

   1. 查看远程仓库 git remote -v
   2. 添加远程仓库 git remote add 仓库别名 仓库地址       // 与git clone 不同,这是自己添加的远程库
   3. 从远程仓库中拉去数据 git fetch 远程仓库名
   4. 推送到远程仓库 git push 远程仓库名 本地分支名:远程仓库的分支名
   5. 查看某个远程仓库 git remote show 远程仓库地址
   6. 远程仓库的重命名 git remote rename 原来仓库名 新仓库名
   7. 远程仓库的移除 git remote remove 仓库名

6. 打标签

   1. 查询所有标签  git tag
   2. 添加附注标签 git tag -a 标签名 -m 标签信息
   3. 查看标签信息下的提交信息 git show 标签名





分支的合并的步骤



1. 合并两个本地分支

   1. 首先确定要合并的两个分支 dev1和dev2
   2. 确保两个分支的代码都是最新状态：分别在两个分支上执行git commit -a -m 提交信息,然后用git status查看状态
   3. 切换到要合并的分支上(假设要将dev2分支合并到dev1分支上,那就切换到dev1分支) git checkout dev1
   4. 在dev1分支上执行 git merge dev2进行合并
      1. 如果两个分支对于文件的修改不存在冲突,则直接合并成功
      2. 如果两个分支对于文件的修改存在冲突.则在冲突的文件中手动进行冲突的解决. 之后对冲突文件执行git add,查看git status无冲突文件后进行git commit

2. 将远程分支dev2合并到本地分支dev1

   1. 首先确认本地分支dev1已经commit或stash,防止被远程分支代码覆盖 在dev1分支 git commit -a -m 提交信息
   2. git pull 远程仓库名 dev2     // 将远程仓库中的dev2分支拉取到本地. 注意 如果本地没有为远程仓库设置别名,就需要去网页端把该仓库的http连接或ssh连接复制到terminal上
   3. 如果存在冲突在代码上手动解决冲突
   4. 对冲突文件执行git add 冲突文件名
   5. git commit提交
   6. giu push 远程仓库名 本地分支名:远程分支名   // 这一步是将本地分支的代码提交到远程分支

   补充: 删除远程分支 git push 远程仓库名  --delete 远程分支名

git branch --merged 可以查看当前有哪些分支进行了合并. 通常情况下合并的分支中只需要保存一个包含原来所有分支内容的分支即可,其他分支可以通过git branch -d 分支名删除掉

git branch --no-merged可以查看当前有哪些分支没有进行合并.这些分支通过git branch -d 分支名的操作无法删除,如果执意要删除,可以通过git branch -D 分支名的操作进行强制删除



