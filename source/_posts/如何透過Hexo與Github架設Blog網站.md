------
title: 如何透過Hexo與Github架設Blog網站
date: 2022-09-04
categories: Hexo
tags: 
 - Hexo
 - Github
 - Blog
------

趁著剛架好網站，印象深刻的時候把這篇寫一寫 XD

當我想要架設這個Blog的時候，
比較過許多Blog網站，
例如：痞客邦、Blogger、WordPress、Medium等攥寫Blog的網站，
其中WordPress太過商業化，
想要有~~炫炮~~好看的頁面、功能外掛都需要付費...；
原本想要使用Medium(介面乾淨、整齊是主因)，
但因不支援Markdown、沒有掌握權等原因作罷。

為什麼最後選擇透過Hexo與Github架設Blog網站，<!--more-->
最主要原因當然是**免費**，
並且在免費的情況下，能夠做到：
***內容樣式自訂***、***方便管理***、***Markdown格式***...等

## Github
### 建立Github帳號
這個網站主要是透過將Hexo的框架Deploy到Github Page下，
因此需要先建立一個屬於自己的[Github帳號](https://github.com/)，
(Github是code版本控制工具，許多專案、教學都會存放於此)
並且建立一個專門給這個網站Deploy使用的Repository。
### 建立Repository
帳號註冊完成後，需要先建立一個Repository。
這裡要注意的是：輸入的Repository名稱一定要是`<user_name>.github.io`，
並且要選擇公開(Public)。
<img src="/images/hexo_github/create_repo.jpg" width="50%">
創建完Repository後，使用git clone把Repository下到本地端。
(看個人習慣使用[指令](https://git-scm.com/downloads)或[Github Desktop](https://desktop.github.com/))
```
$ git clone <repo_url.git>
```

## Hexo
[Hexo](https://hexo.io/zh-tw/)是一個Blog框架，
它能夠在使用者不會寫js、css、html等語言的情況下，
快速建立一個靜態檔案Blog，
同時，具有許多套件能夠提供給使用者彈性調整網頁功能，
並且在文章中能夠支援Markdown格式。
### Hexo安裝
因為安裝Hexo需要使用[npm(node package manager)](https://github.com/nodejs-tw/nodejs-wiki-book/blob/master/zh-tw/node_npm.rst)，
因此安裝前，首先需要先安裝[Node.js](https://nodejs.org/zh-tw/download/)，
安裝完成後，打開Terminal輸入：
```
$ cd <想要安裝的路徑>
$ npm install hexo-cli -g
$ hexo init <要創建的資料夾名稱>
```
完成後將會看到創好的資料夾以及內含的Hexo基本配置檔案，
注意：接下來需要將創好的資料夾中的所有檔案copy到clone下來的資料夾中(原資料夾就可以刪了)，
(如果熟悉操作也可直接安裝在clone下來的資料夾)
並在Terminal輸入：
```
$ cd <clone下來的資料夾路徑>
$ npm install
```
這樣就可以完成Hexo安裝。
### Hexo資料夾架構
完成Hexo安裝後，可以看到Hexo資料夾架構大概長這樣。
```
- _config.yml
- package.json
- package-lock.json
- node_modules
- themes
- source
  - _posts
- scaffolds
  - page.md
  - post.md
  - draft.md
```
**node_modules/**
`node_modules/`這個資料夾底下為專案所需要的Package。
(剛才透過npm install安裝的東西)
**scaffolds/**
`scaffolds/`這個資料夾主要存放預設模板，
當使用`$ hexo new <type> <name>`來創建一個新頁面或貼文時，
Hexo將會使用這些模板建立檔案。
**source/**
`source/`這個資料夾主要存放網站中所有的資料，
除了存放文章的資料夾`_posts/`以外(存放文章的資料夾)，
前面帶有底線的資料夾都會被Hexo忽略掉，
而存在資料夾中的文章(Markdown、HTML等靜態檔案)會被放到`public/`資料夾，
其他檔案則會被複製過去。
**themes/**
`themes/`這個資料夾是用來存放網站佈景主題的，
[Hexo官網](https://hexo.io/themes/)上有許多不同的主題供大家下載，
下載後把整個主題資料夾放到`themes/`裡面，
再到`_config.yml`去改設定就可以了。
**_config.yml**
`_config.yml`是最重要的設定檔，
在這個檔案中可以針對網站的呈現方式做設定。
**package.json**
`package.json`記錄目前有安裝的套件及其版本。
### 根目錄_config.yml設定重點介紹
大致說明一下根目錄`_config.yml`設定，
比較會用到的設定以及要注意的地方。
#### 全站設定
```
# Site
title: # Blog名稱
subtitle: # Blog副標題
description: # Blog基本描述
keywords: # 關鍵字
author: # 作者
language: # 網站語言 e.g. zh-TW
timezone: # 網站時區 e.g. 'Asia/Taipei'
```
#### 網站url設定
```
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: # 網站網址
root: # 網站根目錄
permalink: # 文章永久連結的格式
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```
注意：網站網址，必須使用 `http://` 或 `https://` 開頭。
#### 存放檔案的資料夾設定
```
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```
#### 文章寫作設定
```
# Writing
new_post_name: :title.md # File name of new posts
default_layout: post # 預設佈局
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
filename_case: 0 # 將檔案名稱換成: 1小寫 或 2大寫
render_drafts: false # 是否顯示草稿
post_asset_folder: false # 啟動Asset資料夾
relative_link: false # 把連結改為與根目錄的"相對"位址
future: true # 是否顯示未來的文章
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```
#### 首頁設定
```
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10 # 首頁顯示文章數
  order_by: -date # 文章排序(預設依日期越新越上面)
```
這部分基本上可以安裝`hexo-generator-index2`套件取代，
後續提到會再介紹。
#### 分頁設定
```
# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page
```
`per_page`可以設定一頁顯示的文章量，
若設定0表示關閉分頁功能。
#### 包含 / 不包含
```
# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include: # Hexo預設忽略的隱藏檔、資料夾，若將其列在此Hexo將會使用
exclude: # 列在此的檔案、資料夾Hexo將會忽略
ignore:
```
在此需注意二點：
1. 列在這些項目下的檔案、資料夾需要加雙引號或單引號，如：`"<檔案名稱>"`或`'<檔案名稱>'`。
2. 若要忽略`themes/`資料夾中的內容，需加`_`在前面，如：`_檔案名稱`。
#### 主題
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: <Blog主題>
```
[Hexo官方網站](https://hexo.io/themes/)有許多主題可供下載，
只要將主題下載下來後，
放入`themes/`資料夾中(通常會存成`themes/主題名稱/主題檔案`)，
並將此處Blog主題設定為所下載的主題名稱即可。
#### 部署
```
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://github.com/<user_name>/<repo_name>.git
  branch:  master
```
因為是使用Github來進行Deploy，
需要先安裝`hexo-deployer-git`[套件](https://hexo.io/zh-tw/docs/one-command-deployment.html)，
```
$ npm install hexo-deployer-git --save
```
如果沒安裝將會出現ERROR Deployer not found:git。
### 將Hexo部署到Github Page
將`_config.yml`設定檔調整完成後，
即可開始將Hexo部署到Github Page，
一樣先打開Terminal，並切換到clone下來的資料夾，並輸入：
```
$ cd <clone下來的資料夾路徑>
$ hexo clean # 清除快取檔案db.json、已產生的靜態檔案public
$ hexo generate # 產生靜態檔案
$ hexo server # 啟動伺服器，預設為http://localhost:4000/
$ hexo deploy # 部署網站
```
其中，`$ hexo server`不一定需要使用，
它主要是用來在本地端導覽網站內容(像是測試使用)，
使用時，如果預設的port埠已使用，
可以透過`$ hexo server -p <port埠>`使用另一個port埠來啟動伺服器；
當使用`$ hexo deploy`完成Deploy後，
等待一下再到你的Blog頁面`https://<user_name>.github.io/`，
就可以看到你的Blog網站了。

注意：Deploy後，可能會有一種情況是多了一個`branch`，
(這取決於你怎麼設定`# Deployment`的部分)
如果發生Deploy後等很久網站沒有更新的狀況，
需檢查Github Page設定的地方，
目前所設定的`branch`是否為Deploy的`branch`。
<img src="/images/hexo_github/github_page.jpg" width="50%">
另外，經過Deploy後的Hexo檔案是轉換過的，並非原始檔，
因此，建議如有需要可透過`branch`的方式能夠將原始檔保存於Github中進行版控。




完成以上步驟後，基本的Blog網站就設定好了~