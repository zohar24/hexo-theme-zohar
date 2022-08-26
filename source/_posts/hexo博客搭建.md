---
title: Hexo博客搭建
author: zohar
top: true
cover: false
toc: true
mathjax: true
date: 2022-08-14 15:27:31
tags:
- 博客
categories:
- 博客
---


# 概述

一直想搭建一个自己的博客网站，自己在网上找了一些框架，最终准备使用hexo搭建一个。

Hexo是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。并可通过修改主题，让你的博客丰富多彩。



# 搭建Hexo

详细可参考 [Hexo官方文档](https://hexo.io/zh-cn/docs)

**目录结构**

- 概述
- 搭建Hexo
  - 安装Git
  - 安装Node.js
  - 添加国内镜像源
  - 安装Hexo
  - 模板设置
  - Front-matter 选项详解
  - 最全示例
  - 配置修改
  - 连接Github与本地
  - 发布文章
- 个性化主题
  - 更换主题
  - 新建页面
  - 配置修改
  - 代码高亮
  - 搜索
  - 修改打赏的二维码图片
  - 修改页脚

## 安装Git

下载 [Git](https://git-scm.com/download/win)。

安装选项还是全部默认

安装完成后在命令提示符中输入`git --version`验证是否安装成功。

## 安装Node.js

首先下载稳定版[Node.js](https://nodejs.org/dist/v9.11.1/node-v9.11.1-x64.msi)，最新版的Node.js可前往[Node官网](https://nodejs.org/en/download/)

安装选项全部默认。

最后安装好之后，按`Win+R`打开命令提示符，输入`node -v`和`npm -v`，出现版本号，那么就安装成功了。

### 设置全局和缓存路径(选做)

默认情况下，我们在执行`npm install -g XXXX`下载全局包时，这个包的默认存放路径位`C:\Users\username\AppData\Roaming\npm\node_modules下`，可以通过`CMD`指令`npm root -g`查看

```sh
C:\Users\liaijie\AppData\Roaming\npm\node_modules
```

但是有时候我们不想让全局包放在这里，我们可以自定义存放目录,在`CMD`窗口执行以下两条命令修改默认路径（推荐）：

```sh
npm config set prefix "D:\node\node_global"
npm config set cache "D:\node\node_cache"
```

或者打开`c:\node\node_modules\npm\.npmrc`文件，修改如下：

```sh
prefix =D:\node\node_global
cache = D:\node\node_cache
```

​	以上操作表示，修改全局包下载目录为`C:\node\node_global`,缓存目录为`C:\node\node_cache`,并会自动创建`node_global`目录，而`node_cache`目录是缓存目录，会在你下载全局包时自动创建。

## 添加国内镜像源

没有梯子的话，可以使用阿里的国内镜像进行加速。

```bash
npm config set registry https://registry.npm.taobao.org
```

## 安装Hexo

在合适的地方新建一个文件夹，用来存放自己的博客文件，比如`D:\blog`目录下。

使用定位到该目录下，输入下面的指令安装Hexo。

```bash
npm install -g hexo-cli
```

可能会有几个报错，无视它。

安装完后输入`hexo -v`验证是否安装成功。

然后就要初始化我们的网站，执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件

```bash
$ hexo init <folder>   # 初始化文件夹
$ cd <folder>  
$ npm install # 安装必备的组件
```

之后就可以输入`hexo s`启动服务，效果如下：

![](01.png)

按`ctrl+c`关闭本地服务器。

Hexo常用命令

```bash
hexo g  # 生成博客网页文件
hexo s  # 本地预览博客
hexo d  # 上传网页文件到github
```

## 模板设置

为新建文章方便，建议将`/scaffolds/post.md`修改为如下代码：

```json
---
title: {{ title }}
date: {{ date }}
top: false
cover: false
toc: true
summary:
tags:
categories:
---
```

建议将`/scaffolds/post.md`修改为如下代码：

```json
---
title: {{ title }}
date: {{ date }}
type: {{ title }}
layout: {{ title }}
---
```

这样新建文章后不用你自己补充了，修改信息就行。

### Front-matter 选项详解

`Front-matter` 选项中的所有内容均为**非必填**的。建议至少填写 `title` 和 `date` 的值。

| 配置选项   | 默认值                         | 描述                                                         |
| :--------- | :----------------------------- | :----------------------------------------------------------- |
| title      | `Markdown` 的文件标题          | 文章标题，强烈建议填写此选项                                 |
| date       | 文件创建时的日期时间           | 发布时间，强烈建议填写此选项，且最好保证全局唯一             |
| author     | 根 `_config.yml` 中的 `author` | 文章作者                                                     |
| img        | `featureImages` 中的某个值     | 文章特征图，推荐使用图床(腾讯云、七牛云、又拍云等)来做图片的路径.如: `http://xxx.com/xxx.jpg` |
| top        | `true`                         | 推荐文章（文章是否置顶），如果 `top` 值为 `true`，则会作为首页推荐文章 |
| cover      | `false`                        | `v1.0.2`版本新增，表示该文章是否需要加入到首页轮播封面中     |
| coverImg   | 无                             | `v1.0.2`版本新增，表示该文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片 |
| password   | 无                             | 文章阅读密码，如果要对文章设置阅读验证密码的话，就可以设置 `password` 的值，该值必须是用 `SHA256` 加密后的密码，防止被他人识破。前提是在主题的 `config.yml` 中激活了 `verifyPassword` 选项 |
| toc        | `true`                         | 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能。前提是在主题的 `config.yml` 中激活了 `toc` 选项 |
| mathjax    | `false`                        | 是否开启数学公式支持 ，本文章是否开启 `mathjax`，且需要在主题的 `_config.yml` 文件中也需要开启才行 |
| summary    | 无                             | 文章摘要，自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要 |
| categories | 无                             | 文章分类，本主题的分类表示宏观上大的分类，只建议一篇文章一个分类 |
| tags       | 无                             | 文章标签，一篇文章可以多个标签                               |

### 最全示例

```yml
---
title: typora-vue-theme主题介绍
date: 2018-09-07 09:25:00
author: 赵奇
img: /source/images/xxx.jpg
top: true
cover: true
coverImg: /images/1.jpg
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 这是你自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要
categories: Markdown
tags:
  - Typora
  - Markdown
  
---
```

## 资源文件夹

资源（Asset）代表 `source` 文件夹中除了文章以外的所有文件，例如图片、CSS、JS 文件等。比方说，如果你的Hexo项目中只有少量图片，那最简单的方法就是将它们放在 `source/images` 文件夹中。然后通过类似于 `![](/images/image.jpg)` 的方法访问它们。

对于那些想要更有规律地提供图片和其他资源以及想要将他们的资源分布在各个文章上的人来说，Hexo也提供了更组织化的方式来管理资源。这个稍微有些复杂但是管理资源非常方便的功能可以通过将 `config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。

```yml
_config.yml
post_asset_folder: true
```

当资源文件管理功能打开后，Hexo将会在你每一次通过 `hexo new [layout] <title>` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个文章文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，你可以通过相对路径来引用它们，这样你就得到了一个更简单而且方便得多的工作流。

详细可参考 [资源文件夹](https://hexo.io/zh-cn/docs/asset-folders) 。    



## 配置修改

修改配置文件`_config.yml`

```yml
title: # title
subtitle: # 二级标题
description: # 描述
keywords: # 关键字
author: # 作者
language: zh-CN
timezone: ''

url: # 你的网站地址
```



## 连接Github与本地

首先右键打开git bash，然后输入下面命令：

```bash
git config --global user.name "myname"
git config --global user.email "email@qq.com"
```

用户名和邮箱根据你注册**github的信息**自行修改。

然后生成密钥SSH key：

```bash
ssh-keygen -t rsa -C "email@qq.com" # 会把公钥和私钥生成到~/.ssh文件夹下
```

打开[github](http://github.com/)，在头像下面点击`settings`，再点击`SSH and GPG keys`，新建一个SSH，名字随意。

将公钥id_rsa.pub复制到框中，点击确定保存。

输入`ssh -T git@github.com`，如果如下图所示，出现你的用户名，那就成功了。

![](02.png)

新建一个username.github.io的项目，然后修改根目录下的`_config.yml`文件。

修改最后一行的配置：

```bash
deploy:
  type: git
  repository: git@github.com:username/username.github.io.git
  branch: master
```

**repository修改为你自己的github项目地址**。



## 发布文章

首先在博客根目录下执行命令`npm i hexo-deployer-git` 安装一个deploy。

输入`hexo new post "article title"`，新建一篇文章。

打开`D:\blog\source\_posts`的目录，可以发现下面多了一个`.md`文件。

编写完markdown文件后，根目录下输入`hexo g`生成静态网页，然后输入`hexo s`可以本地预览效果，最后输入`hexo d`上传到github上，这时在github.io仓库就可以看到该文章了。

当执行 `hexo deploy` 时，Hexo 会将 `public` 目录中的文件和目录推送至 `_config.yml` 中指定的远端仓库和分支中，并且**完全覆盖**该分支下的已有内容。



# 个性化主题

本次个性化设置主要针对的是`matery`主题。

## 更换主题

详细介绍参考[hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)

修改 Hexo 根目录下的 `_config.yml` 的 `theme` 的值：`theme: hexo-theme-matery`（看文件夹名）

## 新建页面

### 新建分类 categories 页

`categories` 页是用来展示所有分类的页面，如果在你的博客 `source` 目录下还没有 `categories/index.md` 文件，那么你就需要新建一个，命令如下：

```bash
hexo new page "categories"
```

编辑你刚刚新建的页面文件 `/source/categories/index.md`，至少需要以下内容：

```yaml
---
title: categories
date: 2018-09-30 17:25:30
type: "categories"
layout: "categories"
---
```

### 新建标签 tags 页

`tags` 页是用来展示所有标签的页面，如果在你的博客 `source` 目录下还没有 `tags/index.md` 文件，那么你就需要新建一个，命令如下：

```bash
hexo new page "tags"
```

编辑你刚刚新建的页面文件 `/source/tags/index.md`，至少需要以下内容：

```yaml
---
title: tags
date: 2018-09-30 18:23:38
type: "tags"
layout: "tags"
---
```

### 添加404页面

在`/source/`目录下新建一个`404.md`，内容如下：

```json
---
title: 404
date: 2022-07-10 15:00:00
type: "404"
layout: "404"
description: "你来到了没有知识的荒原 :("
---
```

### 新建关于我 about 页

`about` 页是用来展示**关于我和我的博客**信息的页面，如果在你的博客 `source` 目录下还没有 `about/index.md` 文件，那么你就需要新建一个，命令如下：

```bash
hexo new page "about"
```

编辑你刚刚新建的页面文件 `/source/about/index.md`，至少需要以下内容：

```yaml
---
title: about
date: 2018-09-30 17:25:30
type: "about"
layout: "about"
---
```

#### “关于”页面增加简历（可选）

修改`/themes/matery/layout/about.ejs`，找到`<div class="card">`标签，然后找到它对应的`</div>`标签，接在后面新增一个card，语句如下：

```html
<div class="card">
    <div class="card-content">
        <div class="card-content article-card-content">
                <div class="title center-align" data-aos="zoom-in-up">
                    <i class="fa fa-address-book"></i>&nbsp;&nbsp;<%- __('myCV') %>
                </div>
                <div id="articleContent" data-aos="fade-up">
                    <%- page.content %>
                </div>
        </div>
    </div>
</div>
```

这样就会多出一张card，然后可以在`/source/about/index.md`下面写上你的简历了，当然这里的位置随你自己设置，你也可以把简历作为第一个card。

### 新建友情连接 friends 页（可选的）

`friends` 页是用来展示**友情连接**信息的页面，如果在你的博客 `source` 目录下还没有 `friends/index.md` 文件，那么你就需要新建一个，命令如下：

```bash
hexo new page "friends"
```

编辑你刚刚新建的页面文件 `/source/friends/index.md`，至少需要以下内容：

```yaml
---
title: friends
date: 2018-12-12 21:25:30
type: "friends"
layout: "friends"
---
```

同时，在你的博客 `source` 目录下新建 `_data` 目录，在 `_data` 目录中新建 `friends.json` 文件，文件内容如下所示：

```json
[{
    "avatar": "http://image.luokangyuan.com/1_qq_27922023.jpg",
    "name": "码酱",
    "introduction": "我不是大佬，只是在追寻大佬的脚步",
    "url": "http://luokangyuan.com/",
    "title": "前去学习"
}, {
    "avatar": "http://image.luokangyuan.com/4027734.jpeg",
    "name": "闪烁之狐",
    "introduction": "编程界大佬，技术牛，人还特别好，不懂的都可以请教大佬",
    "url": "https://blinkfox.github.io/",
    "title": "前去学习"
}, {
    "avatar": "http://image.luokangyuan.com/avatar.jpg",
    "name": "ja_rome",
    "introduction": "平凡的脚步也可以走出伟大的行程",
    "url": "ttps://me.csdn.net/jlh912008548",
    "title": "前去学习"
}]
```

## 配置修改

修改配置文件 `/themes/matery/_config.yml` 。 

```yml
# 首页 banner 中的第二个按钮的配置，包括按钮的显示名称、font awesome图标和按钮的超链接.
indexbtn:
  url: https://github.com/xx
# 首页 banner 中的第二行个人信息配置，留空即不启用
socialLink:
  github:  https://github.com/xx
  email: xx@qq.com
  facebook: # https://www.facebook.com/xxx
  twitter: # https://twitter.com/xxx
  qq: 12345678
  weibo: # https://weibo.com/xxx
  zhihu: # https://www.zhihu.com/xxx
  rss: false # true、false
githubLink:
  url: https://github.com/xx
toc: # 目录
  enable: true 
# 其他参考官方文档 
```

## 代码高亮

### 方式一

如果你的博客中曾经安装过 `hexo-prism-plugin` 的插件，那么你须要执行 `npm uninstall hexo-prism-plugin` 来卸载掉它，否则生成的代码中会有 `{` 和 `}` 的转义字符。

然后，修改 Hexo 根目录下 `_config.yml` 文件中 `highlight.enable` 的值为 `false`，并将 `prismjs.enable` 的值设置为 `true`，主要配置如下：

```yaml
highlight:
  enable: false
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false
prismjs:
  enable: true
  preprocess: true
  line_number: true
  tab_replace: ''
```

如果配置不生效，执行下`hexo clean`  后再启动hexo。

### 方式二

由于 Hexo 自带的代码高亮主题显示不好看，所以主题中使用到了 [hexo-prism-plugin](https://github.com/ele828/hexo-prism-plugin) 的 Hexo 插件来做代码高亮，安装命令如下：

```bash
npm i -S hexo-prism-plugin
```

然后，修改 Hexo 根目录下 `_config.yml` 文件中 `highlight.enable` 的值为 `false`，并新增 `prism` 插件相关的配置，主要配置如下：

```yaml
highlight:
  enable: false

prism_plugin:
  mode: 'preprocess'    # realtime/preprocess
  theme: 'tomorrow'
  line_number: false    # default false
  custom_css:
```

## 搜索

本主题中还使用到了 [hexo-generator-search](https://github.com/wzpan/hexo-generator-search) 的 Hexo 插件来做内容搜索，安装命令如下：

```bash
npm install hexo-generator-search --save
```

在 Hexo 根目录下的 `_config.yml` 文件中，新增以下的配置项：

```yaml
search:
  path: search.xml
  field: post
```

## 修改打赏的二维码图片

在主题文件的 `source/medias/reward` 文件中，你可以替换成你的的微信和支付宝的打赏二维码图片

## 修改页脚

页脚信息可能需要做定制化修改，而且它不便于做成配置信息，所以可能需要你自己去再修改和加工。

修改的地方在主题文件的 `/layout/_partial/footer.ejs` 文件中，包括站点、使用的主题、访问量等。

### 增加建站时间

修改`/themes/matery/layout/_partial/footer.ejs`文件，在最后加上

```js
<script language=javascript>
    function siteTime() {
        window.setTimeout("siteTime()", 1000);
        var seconds = 1000;
        var minutes = seconds * 60;
        var hours = minutes * 60;
        var days = hours * 24;
        var years = days * 365;
        var today = new Date();
        var todayYear = today.getFullYear();
        var todayMonth = today.getMonth() + 1;
        var todayDate = today.getDate();
        var todayHour = today.getHours();
        var todayMinute = today.getMinutes();
        var todaySecond = today.getSeconds();
        /* Date.UTC() -- 返回date对象距世界标准时间(UTC)1970年1月1日午夜之间的毫秒数(时间戳)
        year - 作为date对象的年份，为4位年份值
        month - 0-11之间的整数，做为date对象的月份
        day - 1-31之间的整数，做为date对象的天数
        hours - 0(午夜24点)-23之间的整数，做为date对象的小时数
        minutes - 0-59之间的整数，做为date对象的分钟数
        seconds - 0-59之间的整数，做为date对象的秒数
        microseconds - 0-999之间的整数，做为date对象的毫秒数 */
        var t1 = Date.UTC(2017, 09, 11, 00, 00, 00); //北京时间2018-2-13 00:00:00
        var t2 = Date.UTC(todayYear, todayMonth, todayDate, todayHour, todayMinute, todaySecond);
        var diff = t2 - t1;
        var diffYears = Math.floor(diff / years);
        var diffDays = Math.floor((diff / days) - diffYears * 365);
        var diffHours = Math.floor((diff - (diffYears * 365 + diffDays) * days) / hours);
        var diffMinutes = Math.floor((diff - (diffYears * 365 + diffDays) * days - diffHours * hours) / minutes);
        var diffSeconds = Math.floor((diff - (diffYears * 365 + diffDays) * days - diffHours * hours - diffMinutes * minutes) / seconds);
        document.getElementById("sitetime").innerHTML = "本站已运行 " +diffYears+" 年 "+diffDays + " 天 " + diffHours + " 小时 " + diffMinutes + " 分钟 " + diffSeconds + " 秒";
    }/*因为建站时间还没有一年，就将之注释掉了。需要的可以取消*/
    siteTime();
</script>
```

然后在合适的地方（比如copyright声明后面）加上下面的代码就行了：

```html
<span id="sitetime"></span>
```



### 修改不蒜子初始化计数

在`/themes/matery/layout/_partial/footer.ejs`文件最后加上：

```js
<script>
    $(document).ready(function () {

        var int = setInterval(fixCount, 50);  // 50ms周期检测函数
        var pvcountOffset = 80000;  // 初始化首次数据
        var uvcountOffset = 20000;

        function fixCount() {
            if (document.getElementById("busuanzi_container_site_pv").style.display != "none") {
                $("#busuanzi_value_site_pv").html(parseInt($("#busuanzi_value_site_pv").html()) + pvcountOffset);
                clearInterval(int);
            }
            if ($("#busuanzi_container_site_pv").css("display") != "none") {
                $("#busuanzi_value_site_uv").html(parseInt($("#busuanzi_value_site_uv").html()) + uvcountOffset); // 加上初始数据 
                clearInterval(int); // 停止检测
            }
        }
    });
</script>
```

然后把上面几行有段代码：

```html
<% if (theme.busuanziStatistics && theme.busuanziStatistics.totalTraffic) { %>
    <span id="busuanzi_container_site_pv">
        <i class="fa fa-heart-o"></i>
        本站总访问量 <span id="busuanzi_value_site_pv" class="white-color"></span>
    </span>
<% } %>
<% if (theme.busuanziStatistics && theme.busuanziStatistics.totalNumberOfvisitors) { %>
    <span id="busuanzi_container_site_uv">
        人次,&nbsp;访客数 <span id="busuanzi_value_site_uv" class="white-color"></span> 人.
    </span>
<% } %>
```

修改为：

```html
<% if (theme.busuanziStatistics && theme.busuanziStatistics.totalTraffic) { %>
    <span id="busuanzi_container_site_pv" style='display:none'>
        <i class="fa fa-heart-o"></i>
        本站总访问量 <span id="busuanzi_value_site_pv" class="white-color"></span>
    </span>
<% } %>
<% if (theme.busuanziStatistics && theme.busuanziStatistics.totalNumberOfvisitors) { %>
    <span id="busuanzi_container_site_uv" style='display:none'>
        人次,&nbsp;访客数 <span id="busuanzi_value_site_uv" class="white-color"></span> 人.
    </span>
<% } %>
```

其实就是增加了两个`style='display:none'`而已。

## 添加评论插件

主题自带了gitalk插件，只需要去github官网配置即可。

首先打开[github](https://github.com/settings/applications/new)申请一个应用，要填四个东西：

```
Application name //应用名称，随便填
Homepage URL //填自己的博客地址
Application description //应用描述，随便填
Authorization callback URL //填自己的博客地址
```

然后点击注册，会出现两个字符串`Client ID`和`Client Secret`，这个要复制出来。

然后去主题的配置文件`_config.yml`下修改`gitalk`处：

```yml
gitalk:
  enable: true
  owner: 你的github用户名
  repo: 你的github用户名.github.io
  oauth:
    clientId: 粘贴刚刚注册完显示的字符串
    clientSecret: 粘贴刚刚注册完显示的字符串
  admin: 你的github用户名
```

以后写文章的时候，只要在文章页面登陆过github，就会自动创建评论框，**记得每次写完文章后打开博客文章页面一下**。

gitalk总有跨域的问题，后期看情况解决。