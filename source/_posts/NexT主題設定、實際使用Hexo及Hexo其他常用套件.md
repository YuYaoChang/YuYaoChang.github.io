------
title: NexT主題設定、實際使用Hexo及Hexo其他常用套件
date: 2022-09-18
categories: Hexo
tags: 
 - Hexo
 - Github
 - Blog
------

上一篇介紹有關[**如何透過Hexo與Github架設Blog網站**](https://yuyaochang.github.io/2022/09/04/%E5%A6%82%E4%BD%95%E9%80%8F%E9%81%8EHexo%E8%88%87Github%E6%9E%B6%E8%A8%ADBlog%E7%B6%B2%E7%AB%99/#more)，
這一篇來介紹**NexT主題設定、實際使用Hexo及Hexo其他常用套件**
## Theme:NexT
主題的部分，[Hexo官網](https://hexo.io/themes/)上有許多主題可供大家下載，
建議可以找近期還有在維護的主題，
本篇以許多人選擇的[NexT主題](https://github.com/theme-next/hexo-theme-next)為例。
(因為我喜歡乾淨整潔的畫面私心推薦XD)<!--more-->
### NexT基本安裝
雖然說叫安裝，但其實也只是把它從Github上面拉下來，
首先，先切換到`themes/`資料夾中，並建立一個`NexT`資料夾，
完成後，再開啟Terminal到Repository的資料夾中，
並從[NexT Github](https://github.com/theme-next/hexo-theme-next)中將所有檔案`$git clone`下來。
```yml
$cd <repo_file> # 通常為 <user_name>.github.io
$git clone https://github.com/theme-next/hexo-theme-next themes/NexT
```
clone完成後，必須修改根目錄的`_config.yml`檔案，
將`# Extensions`中的`theme:`部分加入資料夾名稱`NexT`。
```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: <Blog主題> # NexT
```
修改完根目錄的`_config.yml`並Deploy就完成NexT基本安裝了。
### NexT _config.yml 設定
由於NexT可以加入的功能太多種，以下將挑幾個較容易使用到的來做說明：
#### NexT設置Blog的logo
如果希望更改Blog網站的logo，
只要將圖片放入`NexT/source/images/`資料夾中，
並調整以下code(使用不同裝置開啟所顯示的圖片)，
即可變更Blog的logo。
(建議將要更換的logo圖片大小，調整成與預設NexT圖片大小一致)
```yml
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```
#### NexT版型設定
NexT目前有四種版型，
想更換版型的話可在`Schemes`這邊進行調整(都是黑白為主)，
剛安裝完成預設主題將會是Muse，拿掉註解即可更換，
可以利用`$hexo server`自己trytry看想要使用哪一種。
```yml
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```
#### NexT menu
這部分主要是在調整網站其他頁面的icon，
如：首頁(home)、關於我(about)、標籤(tags)、分類(categories)等，
(這裡調整後，要記得到根目錄的`_config.yml`檔案，將`# Header`的部分對應的頁面註解拿掉，才會顯示出頁面)
`||`後面是指想要使用的icon，
icon將使用[Font Awesome](https://fontawesome.com/icons?d=gallery)(fa)網站中的icon，
基本上大部分只需要輸入`fa fa-<icon_name>`即可。
```yml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
# Enable / Disable menu icons / item badges.
menu_settings:
  icons: true
  badges: false
```
#### 網站訪問人數、文章閱讀人數
如果想在網站底下新增累積Blog訪問人數、累積文章閱讀人數，
只要將`enable: false`改為`enable: true`即可開啟該功能，
同時，也可以透過Font Awesome來改變icon。
```yml
busuanzi_count:
  enable: false # true
  total_visitors: true
  total_visitors_icon: fa fa-user
  total_views: true
  total_views_icon: fa fa-eye
  post_views: true
  post_views_icon: fa fa-eye
```
#### Google Analytics(GA)
首先要先擁有一個Google帳號，並前往[Google Analytics](https://analytics.google.com/analytics/web/provision/#/provision)開啟服務，
開啟GA服務後，要先建立一個GA的帳戶，
建立完成後，到`設定 > 帳戶 > 資料流串 > 網頁串流詳情(GA_id)`，
<img src="/images/hexo_github/GA_id.jpg" width="50%">
找到GA_id後，將這串GA_id複製貼到以下`tracking_id`這邊做設定即可。
```yml
# Google Analytics
google_analytics:
  tracking_id: # <app_id>
  # By default, NexT will load an external gtag.js script on your site.
  # If you only need the pageview feature, set the following option to true to get a better performance.
  only_pageview: false
```
#### 社群網站連結設定
將社群網站連結加入Blog中，
(可以透過Font Awesome來改變icon)
只要將連結的註解取消即可。
```yml
social:
  #GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:your@mail.com || fa fa-envelope
  #Weibo: https://weibo.com/yourname || fab fa-weibo
  #Google: https://plus.google.com/yourname || fab fa-google
  #Twitter: https://twitter.com/yourname || fab fa-twitter
  #FB Page: https://www.facebook.com/yourname || fab fa-facebook
  #StackOverflow: https://stackoverflow.com/yourname || fab fa-stack-overflow
  #YouTube: https://youtube.com/yourname || fab fa-youtube
  #Instagram: https://instagram.com/yourname || fab fa-instagram
  #Skype: skype:yourname?call|chat || fab fa-skype
```
#### 側邊大頭貼設定
若要開啟此功能，只要將下方`url`部分的註解拿掉即可，
目前預設圖像是`avatar.gif`(一個灰色人像)，
想要跟換的話，只要將大頭貼放到`next/source/images/<your_photo>`並更改`url`就可以更換完成了。
```yml
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: #/images/avatar.gif
  # If true, the avatar will be dispalyed in circle.
  rounded: false
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```

以上就是NexT主題大部分常用的設定，其他可以再自行研究看裡面的內容 ~

### Theme NexT Canvas Nest
[Theme NexT Canvas Nest](https://github.com/theme-next/theme-next-canvas-nest)是一個很有科技感~~炫炮~~的背景(本站使用之背景)，
要如何設定這個~~炫炮的~~背景，
首先，先到根目錄`source/`中創一個`_data/`資料夾，
並在`_data/`資料夾中，創一個`footer.swig`檔案，
`footer.swig`檔案中寫入並儲存以下code：
```
<script color="0,0,255" opacity="0.5" zIndex="-1" count="300" src="https://cdn.jsdelivr.net/npm/canvas-nest.js@1/dist/canvas-nest.js"></script>
```
寫完儲存後，
只要到根目錄`_config.yml`檔案中將`footer`註解拿掉即可。
```yml
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig # 側邊欄
  #postMeta: source/_data/post-meta.swig # 文章標題下
  #postBodyEnd: source/_data/post-body-end.swig # 文章結尾
  footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  #style: source/_data/styles.styl
```
## 實際使用Hexo
Blog基本上就是拿來寫文章用的，
在寫文章的過程中，
一定會有將文章分類的需求(還是只有我有強迫症XDD)，
因此，建立一個分類選項、標籤是非常重要的。
### 創建分類選項、標籤
在Hexo中想要將文章分類，需要先創建一個分類選項，
透過下面指令，創建一個分類頁面：
```yml
$hexo new page categories
```
創建後，`source/categories/`資料夾中會有一個檔案`index.md`，
內容預設是像這樣：
```yml
---
title: categories
date: # 創建的時間
---
```
之後將type加入到內容中並儲存
```yml
---
title: categories
date: # 創建的時間
type: categories
---
```
**使用：**只要在文章前面加入categories就可以將文章分類
```yml
---
title: # 文章主題
date: # 發文時間
categories: # 分類
---
```
同樣的，如果要創造標簽的話，
只需要照前面的步驟創建tags即可。
**這裡要注意的是：**
一篇文章只能有一種分類，
如果在這加入多個分類，
將會默認選擇第一個分類。

## Hexo其他常用套件
### hexo-addlink
**功能：**在每篇文章最後添加永久鏈接及文字(像簽名檔、版權聲明的概念)。
**安裝、使用：**安裝`hexo-addlink`套件後，
```yml
$npm install hexo-addlink
```
將下面的code加在根目錄`_config.yml`檔案中。
```yml
addlink:
  before_text: # 加在網址前面的文字
  after_text: # 加在網址後面的文字
```
### hexo-generator-sitemap
**功能：**Google是透過sitemap來取得網站資料，但Hexo並無內建必須額外安裝套件，才能讓你的Blog被搜尋到。
**安裝、使用：**安裝`hexo-generator-sitemap`套件後，
```yml
$npm install hexo-generator-sitemap
```
在根目錄的`_config.yml`中加入以下code即可。
```yml
# Sitemap
sitemap:
  path: sitemap.xml # sitemap path
  tags: false # include true, exclude false
  categories: false # include true, exclude false
```
完成後，以後在產生靜態網頁時，將會自動產生出`sitemap.xml`檔案，[進一步了解sitemap.xml是什麼](https://awoo.ai/zh-hant/blog/sitemap-xml/#到底，sitemapxml是什麼東西！？)
(可以在`https://<user_name>.github.io/sitemap.xml`查看`sitemap.xml`是否產生成功)
以上步驟完成後，
要到[Google Search Console](https://search.google.com/search-console/about)輸入Blog網址以及`sitemap.xml`，如下：
<img src="/images/hexo_github/sitemap.jpg" width="100%">
看到狀態成功，即完成sitemap設定，
之後Google就會定期看這個檔案是否有更新，
如果有更新將會把新的網址(新文章的分頁)加入Google搜尋引擎中。
後續如果有**文章不想加入搜尋引擎中**，也可以在標題部分加入：
```yml
---
sitemap: false
---
```
這樣那篇文章就不會加入Google搜尋引擎中。
### hexo-generator-searchdb
**功能：**在Blog中使用關鍵字搜索文章
**安裝、使用：**安裝`hexo-generator-searchdb`套件後，
```yml
$npm install hexo-generator-searchdb
```
到`themes/source/next/_config.yml`檔案，
找到以下這段code，
```yml
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```
改成`enable: true`即可。


當然還有很多[hexo套件](https://hexo.io/plugins/)可以使用，可以依照個人需求去找找~



延伸閱讀：
[如何透過Hexo與Github架設Blog網站](https://yuyaochang.github.io/2022/09/04/%E5%A6%82%E4%BD%95%E9%80%8F%E9%81%8EHexo%E8%88%87Github%E6%9E%B6%E8%A8%ADBlog%E7%B6%B2%E7%AB%99/#more)