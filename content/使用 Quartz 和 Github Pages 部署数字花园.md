Quartz是一个静态博客生成器，可以搭配Obsidian来搭建自己的数字花园
- [Quartz文档](https://quartz.jzhao.xyz/)
- [Quartz项目地址](https://github.com/jackyzha0/quartz)
## 准备工作
最低环境要求：[Node](https://nodejs.org/) v20 以及 `npm` v9.3.1

初始化项目：
```bash
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```

执行`npx`命令时可能会报错：
```text
npx : 无法加载文件 C:\Program Files\nodejs\npx.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```

解决方法：

使用管理员身份运行Power Shell，执行命令：
```bash
set-ExecutionPolicy RemoteSigned
```

会出现以下提示，选择Y并回车
```text
执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): Y
```

## 本地预览
初始化完成后可以在本地预览项目：
```bash
npx quartz build --serve
```

## 编写笔记
自己的笔记存放在`/content`目录中，可以将该目录在Obsidian中打开，之后就可以快乐写作了

## 设置Github仓库
在Github创建一个新的存储库，README、gitignore和license留空

在仓库页面上复制远程地址
```bash
git@github.com:berserkduck/dg.git
```

导航到项目根目录，执行：
```bash
git remote -v

git remote set-url origin REMOTE-URL

git remote add upstream https://github.com/jackyzha0/quartz.git

git branch -M main 

git push -u origin main
```
1. `git remote -v`命令用于查看当前本地仓库配置的所有远程仓库（remote）及其对应的 URL，显示每个远程仓库的fetch（拉取） 和 push（推送）地址
2. 执行`git remote set-url origin REMOTE-URL`，将`REMOTE-URL`替换为自己的仓库地址
3. upstream指向原始仓库地址，如果没有upstream地址，可以执行第三条命令进行指定，可以用于更新Quartz
4. 将当前分支强制重命名为main
5. 将本地main分支推送到远程仓库

之后更新内容时，可以使用以下命令：
```bash
npx quartz sync
```

## 部署到Github Pages
在`quartz/.github/workflows/`目录下创建一个新文件 `deploy.yml`
```yml
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      ["main"]
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: "pages"
  cancel-in-progress: false
 
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v3
        with:
          node-version: 18.14
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: public
 
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

在仓库的`Settings > Pages` 中，选择 GitHub Actions作为部署源。执行`npx quartz sync`提交更改，会自动执行部署。