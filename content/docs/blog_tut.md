---
date: '2024-11-10T13:55:21+08:00'
draft: false
title: '使用Hugo部署你的博客'
---

## 简介

Hugo是一个静态网站生成器，它可以帮助你快速搭建一个博客。本文将介绍如何在Windows11系统下，使用Hugo部署你的博客，并将博客部署到GitHub Pages上。

## 准备工作

- 确保在Windows11中安装PowerShell，如果没有安装，可以使用winget进行安装：

    ```bash
    winget install --id Microsoft.PowerShell --source winget
    ```

    【注意】PowerShell与Windows PowerShell不同，Windows PowerShell是Windows系统自带的命令行工具，而PowerShell是一个开源的跨平台命令行工具，可以在Windows、Linux、macOS上运行。
- 确保你的电脑上已经安装了Git，如果没有安装，点击[这里](https://git-scm.com/downloads/win)进行安装
- 确保你的电脑上已经安装了Hugo，如果没有安装，可以参考[Hugo官网](https://gohugo.io/)进行安装
- 安装完成后，在任意地址中，使用下面的命令创建一个新的Hugo网站：

    ```bash
    cd /path/to/your/folder
    hugo new site MyFreshWebsite --format yaml
    # replace MyFreshWebsite with name of your website
    ```

## 安装Hugo主题

Hugo有很多主题可以选择，你可以在[Hugo官网](https://themes.gohugo.io/)上找到你喜欢的主题。这里我们以[hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod)为例，介绍如何安装主题。

1. 推荐使用Git Submodule的方式安装主题，这样可以方便的更新主题。在终端中执行以下命令，将`PaperMod`主题添加为Git Submodule：

    ```bash
    cd /path/to/your/folder/MyFreshWebsite
    git init
    git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
    git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
    git submodule update --remote --merge # update the theme
    ```

2. 修改`config.yaml`文件，将主题设置为`PaperMod`：

    ```yaml
    theme: "PaperMod"
    ```

3. 创建一个新的博客文章：

    ```bash
    hugo new posts/my-first-post.md
    ```

    【注意】`my-first-post.md`是你的文章文件名，可以根据自己的需要修改。生成后，将文章中的`draft`字段设置为`false`，这样文章才会被发布。

4. 运行Hugo服务器，在电脑本地预览网站效果：

    ```bash
    hugo server
    ```

    打开浏览器，访问`http://localhost:1313`，即可看到网站效果。

至此，你已经成功安装了Hugo主题。

## 部署到GitHub Pages

1. 在GitHub上创建一个新的仓库，仓库名为`username.github.io`，其中`username`为你的GitHub用户名。

2. 修改`config.yaml`文件，将`baseURL`设置为你的GitHub Pages地址：

    ```yaml
    baseURL: "https://username.github.io/"
    ```

3. 将本地项目与GitHub仓库关联

    在你的项目根目录下，执行以下命令：

    ```bash
    echo "# README" >> README.md
    git add README.md
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/username/username.github.io.git
    git push -u origin main
    ```

    上述命令将创建一个README文件，并将项目推送到GitHub仓库。

4. 在GitHub仓库中，创建一个新的分支`gh-pages`，否则后续的操作会报错。

5. 在GitHub仓库设置中，找到`Pages`选项下的`Build and deployment`，将`Source`设置为`Deploy from a branch`，并在`Branch`中选择`gh-pages`分支，点击`Save`保存。

6. 在GitHub仓库设置中，找到`Actions`选项下的`General-Workflow permissions`，选择`Read and write permissions`，点击`Save`保存。

7. 在项目根目录下，创建一个新的文件`.github/workflows/deploy.yml`，并将以下内容复制到文件中：

    ```yaml
    name: Publish to GH Pages
    on:
      push:
        branches:
          - main
      pull_request:

    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout source
            uses: actions/checkout@v3
            with:
              submodules: true

          - name: Checkout destination
            uses: actions/checkout@v3
            if: github.ref == 'refs/heads/main'
            with:
              ref: gh-pages
              path: built-site

          - name: Setup Hugo
            run: |
              curl -L -o /tmp/hugo.tar.gz   'https://github.com/gohugoio/hugo/releases/download/v0.138.0/hugo_extended_0.138.0_linux-amd64.tar.gz'
              tar -C ${RUNNER_TEMP} -zxvf /tmp/hugo.tar.gz hugo          
          - name: Build
            run: ${RUNNER_TEMP}/hugo

          - name: Create .nojekyll file
            run: touch built-site/.nojekyll

          - name: Deploy
            if: github.ref == 'refs/heads/main'
            run: |
              cp -R public/* ${GITHUB_WORKSPACE}/built-site/
              cd ${GITHUB_WORKSPACE}/built-site
              git add .
              git config user.name 'your username'
              git config user.email 'your email'
              git commit -m 'Updated site'
              git push          
    ```

    【注意】将`username`替换为你的GitHub用户名。

8. 提交代码到GitHub仓库，GitHub Actions会自动部署你的网站。

    ```bash
    git add .
    git commit -m "update"
    git push
    ```

9. 打开浏览器，访问`https://username.github.io/`，即可看到你的博客。

## 参考链接

1. Youtube视频教程: [Deploy Hugo Blog to Github Pages via Github Actions w/ a Custom Domain](https://www.youtube.com/watch?v=_QSr2_pxIJs)
2. 文本教程: [Deploying a Blog Powered by Hugo to Github Pages w/ Custom Domain via Github Actions](https://theplaybook.dev/docs/deploy-hugo-to-github-pages/)