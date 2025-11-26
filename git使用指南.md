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
     ```

     

   - 提交更改