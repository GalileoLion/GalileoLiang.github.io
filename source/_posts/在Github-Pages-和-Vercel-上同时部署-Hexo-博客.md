---
title: 在Github Pages 和 Vercel 上同时部署 Hexo 博客
---
# 在Github Pages 和 Vercel 上同时部署 Hexo 博客



## Vercel

[Vercel](https://vercel.com/) 是一个云平台，使开发人员能够托管 Jamstack 网站和网络服务，这些网站和服务可即时部署，自动扩展，无需监督，零配置。 他们提供全球边缘网络、SSL 加密、资源压缩、缓存失效等服务。

步骤 1: 在你的 `package.json` 文件中添加一个构建脚本：

```
{
  "scripts": {
    "build": "hexo generate"
  }
}
```

步骤 2: 将你的 Hexo 网站部署到 Vercel 上

要通过 [Git整合Vercel](https://vercel.com/docs/git-integrations) 部署 Hexo 应用程序，请确保它已被推送到 Git 仓库。

使用 [导入流](https://vercel.com/import/git) 将该项目导入 Vercel。 在导入过程中，你会发现所有相关的选项都是预先配置好的；但是，你可以选择改变其中的任何选项，这些选项的清单可以在 [这里](https://vercel.com/docs/build-step#build-&-development-settings) 找到。

在你的项目被导入后，所有后续推送到分支的内容都会产生 [预览部署](https://vercel.com/docs/platform/deployments#preview) ，而对 [生产分支](https://vercel.com/docs/git-integrations#production-branch)（通常是“主分支”）所做的所有更改都会导致 [生产部署](https://vercel.com/docs/platform/deployments#production)。

或者，您可以单击下面的部署按钮创建新项目：

[![部署Vercel](Hexobushu.assets/button.svg+xml)](https://vercel.com/new/hexo)

将项目连接到 Github Pages 的仓库即可。



## Github Pages

在本教程中，我们使用 [GitHub Actions](https://docs.github.com/zh/actions) 部署 GitHub Pages。 此方法适用于公开或私人储存库. 若你不希望将源文件夹上传到 GitHub，请参阅 [一键部署](https://hexo.io/zh-cn/docs/github-pages#一键部署)。

1. 建立名为 ***username\*.github.io**的储存库。 若之前已将 Hexo 上传至其他储存库，将该储存库重命名即可。
2. 将 Hexo 文件夹中的文件 push 到储存库的默认分支。 默认分支通常名为**main**，旧一点的储存库可能名为**master**。

- 将 `main` 分支 push 到 GitHub：

  ```bash
  $ git push -u origin main
  ```

- 默认情况下 `public/` 不会被上传(也不该被上传)，确保 `.gitignore` 文件中包含一行 `public/`。 整体文件夹结构应该与 [示例储存库](https://github.com/hexojs/hexo-starter) 大致相似。

1. 使用 `node --version` 指令检查你电脑上的 Node.js 版本。 记下主要版本（例如，`v20.y.z`）
2. 在储存库中前往 **Settings** > **Pages** > **Source** 。 将 source 更改为 **GitHub Actions**，然后保存。
3. 在储存库中建立 `.github/workflows/pages.yml`，并填入以下内容 (将 `20` 替换为上个步骤中记下的版本)：

```yml
.github/workflows/pages.ymlname: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

1. 部署完成后，前往 *username*.github.io 查看网页。

每次配置完或者输入了新的文章，在目录下使用`hexo cl; hexo g; hexo s`命令可以更新并预览网页。

## 参考文献

在 GitHub Pages 上部署 Hexo：https://hexo.io/zh-cn/docs/github-pages

使用 Vercel 和 Github 部署 Hexo 安装以及使用教程：https://ohevan.com/vercel-hexo-configuration.html

Hexo 博客搭建并部署到 GitHub Pages(2024最新详细版)：https://blog.csdn.net/clearloe/article/details/139879493

https://hexo.io/zh-cn/docs/

https://hexo.geekswg.top/anzhiyu-docs/