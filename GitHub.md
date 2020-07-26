# GitHub

### 1. git原理简介

仓库： **本地仓库**和**远程仓库**（托管在网络端的仓库）

本地仓库：**工作区、版本区**，其中版本区包含**暂存区、仓库区**

从本地仓库将文件git到远程仓库流程：**工作区---暂存区---仓库区---远程仓库**



![image-20200720125811163](C:\Users\Renhb\AppData\Roaming\Typora\typora-user-images\image-20200720125811163.png)

****

https://blog.csdn.net/qq_41782425/article/details/85183250

### 2. 仓库创建

打开本地项目文件夹，除了代码等必要文件外，一个良好的习惯式添加下面几个文件。

* **README.md：**项目的说明文档
* **LICENSE：**许可，从随便一个别任的库里下载，将copyright行修改为自己的时间和名字即可。
* **gitignore：**指明无需上传的文件和子文件夹。

关于.gitignore：首先新建一个.txt文档，在其中写上要忽略的文件和文件夹。然后打开cmd，cd到项目文件下，利用**ren**命令重命名。输入

```c
ren 新建文本文档.txt.gitignore
```

```c
echo "# milk_infrared_spectrum" >> README.md
git init
git add README.md
git commit -m "first commit"

git remote add origin https://github.com/renhb1996/milk_infrared_spectrum.git
# 如果github的readme.md不在本地文件中，则以下命令
git pull --rebase origin master
# 然后再上传    
git push -u origin master

```

### 3. 上传

项目文件右键，选Git Bash Here

```c++
git init# 初始化	
git add *# 将所有文件上传至暂存区
git commit -m 'first comment'# 将暂存区文件上传至仓库区，注释

git remote add origin https://github.com/renhb1996/milk_infrared_spectrum.git# 连接远程仓库

git push -u origin master# 将文件从仓库区传至远程仓库
```

