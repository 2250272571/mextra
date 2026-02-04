---
title: hexo命令
date: 2025-05-20 22:18:09
sticky: true
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/hexo.jpg"
tags:
  - git
---

## 发布新文章
```C
hexo new "文章名"
hexo clean
hexo g
hexo s   // 生成本地静态看一下效果
hexo d   // 发布到github public 仓库

tcb hosting deploy ./public / -e cloud1-6gbla9jv5f0ef83e  //部署到腾讯云  如果提示未登录就使用tcb login
```
`输入完上面的命令后，在vscode推送源码仓库到github上`

## 连接远程仓库

```C
git init
git remote add origin [dizhi]
git checkout -b main                  
```

## 推送代码

```c
git add .
git commit -m ""
git push --force origin main 	#后续可以直接用git push
```

## 如何建立一个多人写作的git仓库？

### 1. **新建Git仓库**

- **在本地创建仓库**：

  ```bash
  mkdir my_project
  cd my_project
  git init
  ```

- **或者在GitHub/GitLab等平台创建仓库**：如果你使用的是GitHub、GitLab等托管服务，可以直接在网页端创建仓库，然后克隆到本地：

  ```bash
  git clone https://github.com/your-username/my_project.git
  cd my_project
  ```

### 2. **他人参与协作**

- **邀请他人访问仓库**：

  - 如果是GitHub，可以在仓库设置中添加协作者（Collaborator）。
  - 如果是GitLab，可以在项目成员管理中添加成员。

- **他人克隆仓库**：

  ```bash
  git clone https://github.com/your-username/my_project.git
  cd my_project
  ```

### 3. **他人创建分支并上传代码**

- **他人创建分支**：

  ```bash
  git checkout -b fenzhiname
  ```

  `fenzhinamme指的是自定义分支名`

- **他人修改代码并提交**：

  ```bash
  git add .
  git commit -m "Add new feature"
  ```

- **他人将分支推送到远程仓库**：

  ```bash
  git push origin fenzhiname
  ```

### 4. **你合并分支**

- **同步并重置 main**：

  ```bash
  git fetch origin
  git checkout main
  git reset --hard origin/main
  ```

- **合并**：

  ```bash
  git merge origin/wpy_app --allow-unrelated-histories
  ```

- **解决冲突（如果有）并推送**：

  ```bash
  git push origin main
  ```

  
