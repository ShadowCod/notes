### git使用

#### 第一部分：介绍

1. git是什么
   - git是一个版本控制软件，用于控制各个版本的代码
2. 为什么要做版本控制
   - 如果不用版本控制，版本的迭代和回滚操作会很麻烦

3. 安装git
   - 去[git](https://git-scm.com/downloads)官网下载对应的版本，然后一直next就行了

#### 第二部分：单人开发

​	**git的三大区域：工作区、暂存区、版本库**

1. 进入一个文件夹让git来管理该文件

   ```
   git init//初始化该文件夹，让git对其进行管理
   ```

2. 设置用户名称和邮箱

   ```go
   //local只是该文件夹项目使用此名称和邮箱
   git config --local user.name ""
   git config --local user.mail ""
   //global是所有的git管理的文件夹项目都使用此名称和邮箱
   git config --global user.name ""
   git config --global user.mail ""
   ```

3. 提交工作目录中的修改到暂存区

   ```go
   //.表示将所有修改的文件都添加到
   git add 文件名|git add .
   ```

4. 查看文件夹中所有文件的状态信息

   ```go
   git status
   ```

5. 在本地仓库中生成新的版本

   ```go
   git commit -m '提交说明'
   ```

6. 查看提交记录

   ```go
   git log
   git reflog//能够查看到回退版本的操作
   ```

7. 回滚版本

   ```go
   git reset --hard "版本hash值"
   ```

8. 将工作区已修改的状态还原到未修改的状态

   ```go
   git checkout 文件名称
   ```

9. 将暂存区的修改修改还原到工作区已修改的状态

   ```
   git reset HEAD 文件名称|.|通配符
   ```

   

#### 第三部分：出现分支

1. 查看分支

   ```
   git branch
   ```

2. 创建分支

   ```
   git branch 分支名称
   ```

3. 切换分支

   ```
   git checkout 分支名称
   ```

4. 合并分支

   ```
   //注意：A要合并到B，需先切换到B分支,然后在执行合并命令
   git merge 分支名称
   //可能产生冲突，需要手动解决
   ```

5. 删除分支

   ```
   git branch -d 分支名称
   ```


#### 第四部分：远程仓库

1. 在远程仓库地址上创建一个项目仓库并获得一个地址

2. 在git上给项目地址取个别名

   ```
   git remote add 别名 项目地址	eg:git remote add origin https://github.com/TFPanzer
   ```

3. 将本地项目推送到远程仓库中

   ```
   git push -u 别名 分支名	eg:git push -u origin master
   ```

4. 将远程仓库中的项目克隆到本地

   ```
   git clone 项目地址	eg:git clone https://github.com/TFPanzer
   注意：clone下来的项目使用git branch查看分支时只会显示master但是实际上可以直接切换到其他分支
   ```

5. 将远程仓库中项目的改动拉取到本地

   ```
   git pull 别名 分支名称
   ```

   

