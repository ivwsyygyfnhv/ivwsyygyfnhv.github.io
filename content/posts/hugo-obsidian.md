---
title: Hugo+Obsidian+Github部署博客
date: 2024-03-29T17:21:47+08:00
tags:
  - blog
categories: 
draft: false
---
## 安装 Hugo

安装 hugo：
```shell
go install github.com/gohugoio/hugo@latest
```

新建项目：
```shell
hugo new site blog
```

安装主题：
```shell
git submodule add https://github.com/nodejh/hugo-theme-mini.git themes/mini
```

配置 `hugo.toml`：
```toml
theme = 'mini'
```

迁移文章到项目中：
```shell
mv <path_to_md> content/posts
```

启动 hugo 命令部署网站：
```shell
hugo deploy -D
```

## 配置 Github Actions 部署

配置`.github/workflows/hugo.yml`文件：
```yml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.124.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

提交到 github 仓库，查看 Actions：
![截屏2024-03-29 17.33.49.png](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-03-29%2017.33.49.png)

## 配置 Obsidian

在文件与链接中，指定新建笔记的存放位置为指定的附件文件夹：`content/posts`，这样新建笔记时，便会在 hugo 的页面目录中创建页面，同时指定忽略文件为除了 `content` 外的其他文件夹。

在创建页面时，需要在页面的头部定义 Front matter，为了简化这个过程，可以在项目中新建一个 template 目录，并且在目录下定义 Front-matter 文件：
```yml
---
title: {{title}}
date: {{date}}
tags: []
categories: []
draft: false
---
```
指定模板的日期格式为：`YYYY-MM-DDTHH:mm:ssZ`，这样在插入模板时，会把格式化后的日期替换到 `{{date}}` 中。

图床配置这里我选择了 PicGo+腾讯云的组合，在 Obisidian 中安装第三方插件：`Image auto upload Plugin`，然后在 PicGo 中填入 cos 的存储桶配置，在插件选项中开启剪切板自动上传，这样便可以实现从本地剪贴板复制图片上传到腾讯云，并且把地址链接粘贴到编辑器了：
![截屏2024-03-29 17.49.12.png](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-03-29%2017.49.12.png)

最后还可以在 Obsidian 中安装 Git 插件，当完成编辑文章时，便可以在 Command+P 弹出的命令面板中使用 Git Push 命令，来同时完成博客的备份以及 Github Pages 的部署了。
