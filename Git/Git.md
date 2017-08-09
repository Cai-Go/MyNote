# Git 操作
## 创建版本库 
创建一个空目录
````java
$ mkdir MyNote
$ cd MyNote
$ pwd
/f/MyNote
````
**pwd**命令用于显示当前目录。
通过**git init**命令把这个目录变成Git可以管理的仓库：
````java
$ git init
````
仓库建好了。
## 把文件添加到版本库
编写一个**ReadMe.md**文件，然后放到Git仓库。
第一步，用命令**git add**告诉Git，把文件添加到仓库：
````java
 $ git add ReadMe.md
````
第二步，用命令**git commit**告诉Git，把文件提交到仓库：
````java
$ git commit -m "create ReadMe.md" //-m后面输入的是本次提交的说明，可以输入任意内容。
````
## 添加到GitHub
先在github创建新的仓库，完成后在本地的**MyNote**仓库下运行命令：
````java
$ git remote add origin git@github.com:wuzhaohui026/MyNote.git
$ git push -u origin master  //把本地库的所有内容推送到远程库上

````
从现在起，只要本地作了提交，就可以通过命令：
````java
$ git push origin master
````
## 分支
创建**dev**分支
````java
$ git checkout -b dev
````
**git checkout**命令加上**-b**参数表示创建并切换，相当于以下两条命令：
````java
$ git branch dev
$ git checkout dev
````
用**git branch**命令查看当前分支：
````java
$ git branch
````
**git branch**命令会列出所有分支，当前分支前面会标一个*号。
当**dev**分支的工作完成，我们就可以切换回**master**分支：
````java
$ git checkout master
````
把**dev**分支的工作成果合并到**master**分支上：
````java
$ git merge dev  //git merge命令用于合并指定分支到当前分支。
````
删除dev分支了
````java
$ git branch -d dev
````
## 冲突
当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。
## 相关命令
- **git status**:可以让我们时刻掌握仓库当前的状态。
- **git diff**:顾名思义就是查看difference，显示的格式正是Unix通用的diff格式。
- **git log**:显示从最近到最远的提交日志。如果嫌输出信息太多，看得眼花缭乱的，可以试试加上** - -pretty=oneline**参数
- **git reset - -hard 版本号**:版本回退
> 在Git中，用**HEAD**表示当前版本，上一个版本就是**HEAD^**，上上一个版本就是**HEAD^^**，当然往上100个版本写100个^比较容易数不过来，所以写成 **HEAD~100**。
- **git reflog**:用来记录你的每一次命令。
- **git checkout - - file**:意思就是把文件在工作区的修改全部撤销。
- **git reset HEAD file**:可以把暂存区的修改撤销掉（unstage），重新放回工作区。
- **rm file**删除文件或者 **git rm file**删掉，并且**git commit**。
- **git clone**:克隆一个本地库。
- **git branch**:查看分支。
- **git branch <name>**:创建分支。
- **git checkout <name>**:切换分支。
- **git checkout -b <name>**:创建+切换分支。
- **git merge <name>**:合并某分支到当前分支。
- **git branch -d <name>**:删除分支。
- **git log - -graph**:查看分支合并图。
- ** git merge - -no-ff -m "merge with no-ff" dev**:- -no-ff参数，表示禁用Fast forward。
- **$ git stash**:把当前工作现场“储藏”起来，等以后恢复现场后继续工作。
- **git stash list**：查看工作现场。
- **git stash apply**：恢复现场，再用**git stash drop**来删除。
- **git stash pop**：恢复的同时把stash内容也删了。
- **git branch -D <name>**：强行删除一个没有被合并过的分支。
- **git remote**：查看远程库的信息。
- **git remote -v**：显示更详细的信息。
- **git push origin dev**：推送其他分支。
- **git pull**:把最新的提交抓下来。
- **git tag <name>**：打标签。
- **git show <tagname>**：查看标签信息。
- **git tag -a <tagname> -m "blablabla..."**：可以指定标签信息。
- **git tag -s <tagname> -m "blablabla..."**：可以用PGP签名标签。
- **git push origin <tagname>**：可以推送一个本地标签。
- **git push origin --tags**:可以推送全部未推送过的本地标签。
- **git tag -d <tagname>**：可以删除一个本地标签。
- **git push origin :refs/tags/<tagname>**：可以删除一个远程标签。