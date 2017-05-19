---
title: browser-sync实时预览html/css效果
tags: browser-sync, sublime text 3, npm, nodejs, 实时预览
grammar_cjkRuby: true
---


## Windows下使用browser-sync实时预览修改的html/css

### 1、安装npm
下载npm安装包[npm][1]，安装完成之后检查nodejs和npm的安装：
```node -v```
```npm -v```
### 2、安装browser-sync
**使用Node.js的包管理（NPM）库来安装BrowserSync**。打开一个终端窗口，运行以下命令：
```npm install -g browser-sync```

_**安装完成之后**_:需要将安装的node module即browser-sync添加到环境变量，
* 将```C:\Users\username\AppData\Roaming\npm```添加到环境变量
* 设置NODE_PATH:命令行输入： ```set NODE_PATH=%AppData%\npm\node_modules```

**启动 BrowserSync**:
一个基本用途是，如果您只希望在对某个css文件进行修改后会同步到浏览器里。那么您只需要运行命令行工具，进入到该项目（目录）下，并运行相应的命令：
**静态网站**
如果您想要监听.css文件, 您需要使用服务器模式。 BrowserSync 将启动一个小型服务器，并提供一个URL来查看您的网站。
// --files 路径是相对于运行该命令的项目（目录） 
browser-sync start --server --files "css/*.css"
如果您需要监听多个类型的文件，您只需要用逗号隔开。例如我们再加入一个.html文件（**注意**：_html文件的名字必须为index.html_,否则浏览器打开文件后显示cann't get ）
```// --files 路径是相对于运行该命令的项目（目录） 
browser-sync start --server --files "css/*.css, *.html"
// 如果你的文件层级比较深，您可以考虑使用 **（表示任意目录）匹配，任意目录下任意.css 或 .html文件。
browser-sync start --server --files "**/*.css, **/*.html"
```
我们做了一个静态例子的示范，您可以下载示例包，文件您可以解压任何盘符的任何目录下，不能是中文路径。打开您的命令行工具，进入到BrowsersyncExample目录下，运行以下其中一条命令。Browsersync将创建一个本地服务器并自动打开你的浏览器后访问http://localhost:3000地址，这一切都会在命令行工具里显示。你也可以查看Browsersync静态示例视频
```
// 监听css文件 
browser-sync start --server --files "css/*.css"
// 监听css和html文件 
browser-sync start --server --files "css/*.css, *.html"
```
**动态网站**
如果您已经有其他本地服务器环境PHP或类似的，您需要使用代理模式。 BrowserSync将通过代理URL(localhost:3000)来查看您的网站。
```// 主机名可以是ip或域名
browser-sync start --proxy "主机名" "css/*.css"
```
在本地创建了一个PHP服务器环境，并通过绑定Browsersync.cn来访问本地服务器，使用以下命令方式，Browsersync将提供一个新的地址localhost:3000来访问Browsersync.cn，并监听其css目录下的所有css文件。

```browser-sync start --proxy "Browsersync.cn" "css/*.css"```
**一点建议**
我们建议您结合gulp或grunt来使用，我们这里有详细说明Gulp文档、Grunt文档。如果您还没有使用gulp或grunt，那么可以通过以上方式创建Browsersync
### 3、在sublime text 3中通过插件browser-sync实时预览html
除了上面使用nodejs安装的browser-sync来实时预览html之外，__sublime中的browser-sync插件也能实时预览，而且安装使用起来非常方便__
直接通过package control 安装就行，然后在顶部工具栏打开browser-sync就可以了
### 4、webstorm实时预览html（LiveEdit）
webstorm自带的LiveEdit模式也能实时预览html，具体使用步骤请参考百度搜索结果。[LiveEdit设置步骤][2]


  [1]: https://www.npmjs.com/get-npm?utm_source=house&utm_medium=homepage&utm_campaign=free%20orgs&utm_term=Install%20npm
  [2]: https://jingyan.baidu.com/article/454316ab68ac03f7a7c03ae3.html