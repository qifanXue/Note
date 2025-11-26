Git使用指南

# 安装git

1. 官网下载对应git版本：[Git - Install for Windows](https://git-scm.com/install/windows)

2. 安装过程中将默认branch改为`main`

3. 安装好后，打开`git bash`

   1. 输入`git --version`测试是否安装成功

   2. 配置用户信息

      ```
      # 设置你的用户名，提交时会记录
      git config --global user.name "你的姓名" 
      # 设置你的邮箱地址
      git config --global user.email "你的邮箱@example.com"
      ```

   3. 检查配置是否成功

      ```
      git config --list
      ```



# 创建仓库和获取仓库

- 初始化新仓库

  ```
  git init
  ```

  

- 克隆现有仓库

  ```
  git clone <远程仓库URL>
  ```

# 修改暂存提交

- 检查状态

  ```
  git status
  ```

- 暂存更改

  ```
  git add <文件名>    # 暂存特定文件
  git add .          # 暂存所有当前目录下的更改（常用）
  git add -p         # 交互式地选择要暂存的文件片段（高级功能）
  ```

- 提交更改

  ```
  git commit -m "清晰的提交说明信息"
  ```

  

# 分支与同步

- 分支基础：

  ```
  git branch <分支名>           # 创建新分支
  git checkout <分支名>         # 切换到指定分支
  git checkout -b <新分支名>    # 创建并立即切换到新分支（常用）[2,5](@ref)
  ```

- 连接远程仓库

  ```
  git remote add origin <远程仓库URL>
  ```

- 推送与拉取

  ```
  git push -u origin main      # 首次推送本地`main`分支到远程`origin`并建立跟踪[1](@ref)
  git push                     # 之后推送可简化为此命令[2](@ref)
  git pull                     # 从远程仓库拉取最新更改并合并到本地[2,5](@ref)
  ```

  