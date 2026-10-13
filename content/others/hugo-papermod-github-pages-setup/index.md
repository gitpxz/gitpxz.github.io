+++
date = '2026-10-11T14:38:27+08:00'
draft = false
title = '从0到搭建博客全流程：Hugo + PaperMod + GitHub Pages'
+++

# 从 0 搭建博客全流程：Hugo + PaperMod + GitHub Pages

真正的长期资产应该是**自建博客**。平台托管的数据所有权不在你手里，平台政策一变（如博客园 2023 年的商业化危机）积累就可能受损。自建方案用 GitHub Pages + Hugo，**完全免费、Markdown 原生、文章纳入 Git 版本控制**，意味着你的技术积累本身就是一份可回溯的代码仓库。起步门槛只是“花 2 小时配好主题”。

下面按实际操作顺序，把整个搭建过程梳理成一份可复用的步骤清单。每一步都标注了关键命令和容易踩的坑。

**一、环境准备**

**1. 安装 Git**
Windows 上从 [git-scm.com](https://git-scm.com/) 下载安装，安装后在终端执行 git --version 确认。

https://github.com/git-for-windows/git/releases/download/v2.56.0.windows.2/Git-2.56.0.2-64-bit.exe

![](hugo-papermod-github-pages-setup_images/image_1.png)

![](hugo-papermod-github-pages-setup_images/image_2.png)

**2. 安装 Hugo Extended**
Hugo 没有安装程序，需要手动配置：

- 从 Hugo 的 GitHub Releases 下载 hugo\_extended\_xxx\_windows-amd64.zip

地址：<https://github.com/gohugoio/hugo/releases>

![](hugo-papermod-github-pages-setup_images/image_3.png)

![](hugo-papermod-github-pages-setup_images/image_4.png)

- 解压出 hugo.exe，放到一个固定目录，比如 C:\hugo\bin

![](hugo-papermod-github-pages-setup_images/image_5.png)

- 把 C:\hugo\bin 加入系统环境变量 **Path**
- **关闭并重新打开终端**，执行 hugo version 验证

![](hugo-papermod-github-pages-setup_images/image_6.png)

如果报 'hugo' 不是内部或外部命令，说明 Path 没配好，或终端没重启。

hugo官方下载地址：https://github.com/gohugoio/hugo/releases

hugo官方文档地址：https://gohugo.io/getting-started/

文档的中文翻译版：https://hugo.opendocs.io/getting-started/

**二、创建本地站点**

hugo new site pxz

cd pxz

git init

![](hugo-papermod-github-pages-setup_images/image_7.png)

此时目录结构大致为：

pxz/

├── archetypes/

├── content/

├── themes/

├── static/

└── hugo.toml

**三、安装 PaperMod 主题**

**推荐方式：Git 子模块**

git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

![](hugo-papermod-github-pages-setup_images/image_8.png)

**如果网络报错**（Connection was reset），有两种处理：

- 给 Git 配置代理：git config --global http.proxy http://127.0.0.1:端口
- 或者直接下载主题 Zip 包，解压后手动放到 themes/PaperMod/

**验证子模块状态：**

git submodule status

![](hugo-papermod-github-pages-setup_images/image_9.png)

开头是**空格**表示正常，开头是**减号**说明没拉取完整，需要 git submodule update --init --recursive。

**四、配置 hugo.toml**

这是最容易出问题的环节。核心配置示例：

![](hugo-papermod-github-pages-setup_images/image_10.png)

baseURL = "https://你的用户名.github.io/"

locale = "en-us"

title = "毫末生木"

theme = "PaperMod"

[params]

description = "站点描述"

author = "你的名字"

[params.homeInfoParams]

Title = "欢迎"

Content = "这里记录技术积累。"

[[menu.main]]

name = "文章"

url = "/posts/"

weight = 10

[[menu.main]]

name = "分类"

url = "/categories/"

weight = 20

[[menu.main]]

name = "标签"

url = "/tags/"

weight = 30

[[menu.main]]

name = "归档"

url = "/archives/"

weight = 40

**关键坑点：**

|  |  |  |
| --- | --- | --- |
| **问题** | **原因** | **解决** |
| invalid UTF-8 character | 文件带了 BOM | 用 VS Code「Save with Encoding → UTF-8」（不带 BOM） |
| 线上样式丢失 | baseURL 末尾缺 / | 必须写成 https://用户名.github.io/ |
| 中文标题报错 | 编码不一致 | 统一用 UTF-8 无 BOM 保存 |

**五、创建归档页和第一篇文章**

**归档页** content/archives.md：

---

title: "归档"

layout: "archives"

url: "/archives/"

summary: archives

---

**创建文章：**

hugo new posts/hello.md

![](hugo-papermod-github-pages-setup_images/image_11.png)

文章 Front Matter 示例：

---

title: "Hello"

date: 2026-10-11

draft: false

categories: ["技术"]

tags: ["Hugo"]

---

正文内容...

**关于 draft：**

- draft: true → 本地能看，线上不发布
- draft: false → 正式发布

想新建文章默认就是 false，改 archetypes/default.md 模板里的 draft 值。

**六、本地预览**

hugo server

![](hugo-papermod-github-pages-setup_images/image_12.png)

访问 http://localhost:1313/。如果只看到一行站点标题，说明还没有文章或首页配置未生效，属正常现象，加文章后就会变成列表。

下面是自己增加了几篇文章后的本地效果，接下来托管到github上。

![](hugo-papermod-github-pages-setup_images/image_13.png)

**七、配置 GitHub Actions 自动部署**

**创建 .github/workflows/hugo.yaml：没有文件夹的话自己手动创建**

![](hugo-papermod-github-pages-setup_images/image_14.png)

name: Deploy Hugo site to Pages

on:

push:

branches: ["main"]

workflow\_dispatch:

permissions:

contents: read

pages: write

id-token: write

concurrency:

group: "pages"

cancel-in-progress: false

jobs:

build:

runs-on: ubuntu-latest

steps:

- name: Checkout

uses: actions/checkout@v4

with:

submodules: recursive

- name: Setup Hugo

uses: peaceiris/actions-hugo@v3

with:

hugo-version: 'latest'

extended: true

- name: Build

run: hugo --minify

- name: Upload artifact

uses: actions/upload-pages-artifact@v3

with:

path: ./public

deploy:

needs: build

runs-on: ubuntu-latest

environment:

name: github-pages

url: ${{ steps.deployment.outputs.page\_url }}

steps:

- name: Deploy to GitHub Pages

id: deployment

uses: actions/deploy-pages@v4

**关键点：**

- submodules: recursive 不能少，否则 PaperMod 拉不下来
- 分支名要和实际一致（这里用 main）

**创建 .gitignore：**

public/

.hugo\_build.lock

resources/

![](hugo-papermod-github-pages-setup_images/image_15.png)

**创建 static/.nojekyll（空文件）：**

type nul > static\.nojekyll

放在 static/ 下才会被 Hugo 复制到 public/ 并部署上线。

**八、推送到 GitHub**

这里需要github账号，没有账号需要自行注册，并创建对应的仓库

**1. 在 GitHub 网页创建仓库**

- 仓库名：你的用户名.github.io
- 可见性：**Public**
- README、.gitignore、license **全部不勾**

**2. 配置 Git 身份**

git config --global user.email "你的邮箱"

git config --global user.name "你的名字"

**3. 关联并推送**

![](hugo-papermod-github-pages-setup_images/image_16.png)

![](hugo-papermod-github-pages-setup_images/image_17.png)

git remote add origin https://github.com/你的用户名/你的用户名.github.io.git

git branch -M main

git add .

git commit -m "init: hugo blog with PaperMod"

git push -u origin main

**九、启用 GitHub Pages**

1. 进入仓库 **Settings → Pages**

![](hugo-papermod-github-pages-setup_images/image_18.png)

1. **Source** 改为 **“GitHub Actions”**（不是 “Deploy from a branch”）
2. 去 **Actions** 标签页看 workflow 是否绿色通过

![](hugo-papermod-github-pages-setup_images/image_19.png)

**如果线上 404：**

|  |  |
| --- | --- |
| **排查项** | **检查内容** |
| Pages Source | 必须是 GitHub Actions |
| baseURL | 末尾斜杠不能少 |
| .nojekyll | 必须在 static/ 下 |
| 子模块 | workflow 里要有 submodules: recursive |
| 缓存 | 用无痕模式访问，等 5-10 分钟 |

**十、日常写作流程**

hugo new posts/文章名.md

![](hugo-papermod-github-pages-setup_images/image_20.png)

# 写内容，确认 draft: false

git add .

git commit -m "新文章"

git push

![](hugo-papermod-github-pages-setup_images/image_21.png)

几十秒后线上自动更新。

下面的推送到github后的效果：

![](hugo-papermod-github-pages-setup_images/image_22.png)

**十一、进阶：分类与多级目录**

**分类（Categories）**：在 Front Matter 里加 categories: ["分类名"]，自动生成 /categories/分类名/ 页面，一篇文章可属于多个分类。

**标签（Tags）**：同理，用 tags，适合做关键词聚合。

**多级目录**：content/posts/hugo/setup.md 会生成 /posts/hugo/setup/，导航栏下拉菜单用 parent 字段实现。

**图片**：推荐 Page Bundle 方式，把图片和 index.md 放同一文件夹，用相对路径引用。

**十二、增加评论功能**

你的博客已经在用 GitHub，评论数据直接存进仓库的 Discussions，不需要额外注册账号，也没有广告。

**第一步：准备工作**

1. 登录github，确保你的博客仓库是 **Public** 的
2. 去仓库 **Settings → General → Features**，勾选 **Discussions**

![](hugo-papermod-github-pages-setup_images/image_23.png)

勾上Discussions并点击Set up discussions

![](hugo-papermod-github-pages-setup_images/image_24.png)

1. 访问 [github.com/apps/giscus](https://github.com/apps/giscus)，点击install，安装到你的博客仓库

![](hugo-papermod-github-pages-setup_images/image_25.png)

![](hugo-papermod-github-pages-setup_images/image_26.png)

1. 访问 [giscus.app](https://giscus.app/)，填写仓库名（你的仓库名称），页面会生成一段 <script> 代码，里面包含 data-repo-id、data-category-id 等参数

![](hugo-papermod-github-pages-setup_images/image_27.png)

**第二步：创建评论模板**

在项目根目录创建 layouts/partials/comments.html（**不要改主题里的文件**，否则更新主题会被覆盖）：

<div class="comments">

<script src="https://giscus.app/client.js"

data-repo="gitpxz/gitpxz.github.io"

data-repo-id="R\_kgDO..."

data-category="Announcements"

data-category-id="DIC\_kwDO..."

data-mapping="pathname"

data-strict="1"

data-reactions-enabled="1"

data-emit-metadata="0"

data-input-position="bottom"

data-theme="preferred\_color\_scheme"

data-lang="zh-CN"

crossorigin="anonymous"

async>

</script>

</div>

把 data-repo-id 和 data-category-id 换成 giscus.app 生成的实际值。

**第三步：在 hugo.toml 中开启评论**

[params]

comments = true

**第四步：推送并验证**

git add layouts/partials/comments.html hugo.toml

git commit -m "add giscus comments"

git push

等 Actions 跑完，打开任意一篇文章，拉到页面底部，应该能看到评论框。

![](hugo-papermod-github-pages-setup_images/image_28.png)
