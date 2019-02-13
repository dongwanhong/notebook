# Gitbook 初级指南
通常大家在工作和生活中总会要记录一些东西，而做好的笔记最好不会丢失又操作简单，在这样的需求的驱动下我偶然接触到 `Gitbook` 这款工具，在此我对其如何使用作一个简单的记录。

## 概念
Modern book format and toolchain using Git and Markdown.
`GitBook` 是一个基于 `Node.js` 的命令行工具，可使用 `Github/Git` 和 `Markdown` 来制作精美的电子书，事实上 `GitBook` 并不是关于 `Git` 的教程，但是结合 `Git` 我们可以对电子书进行有效的版本控制，国内外一些开发者更是通过 `GitBook` 和 `gitalk` 等一些插件创建属于自己的博客系统。

## 作用
`GitBook` 支持输出多种文档格式：
 * **静态站点**：`GitBook` 默认输出该种格式，生成的静态站点可直接托管搭载 Github Pages 服务上。
 * **PDF**：需要安装 `gitbook-pdf` 依赖。
 * **eBook**：需要安装 `ebook-convert`。
 * **单HTML网页**：支持将内容输出为单页的 `HTML`。
 * **JSON**：一般用于电子书的调试或元数据提取。

## 安装 Node.js
因为 `GitBook` 是基于 `Node.js` 的工具，所以我们首先需要安装 `Node.js`（[下载地址](https://nodejs.org/en/download/)），找到对应平台的版本安装即可。
现在安装 `Node.js` 都会默认安装 `npm`（包管理工具），所以我们不用单独安装 npm，安装完成通过以下命令检查是否安装成功：
``` bash
$ node -v
```
以上命令在 `node` 安装成功后会显示 `node` 的版本号。
``` bash
$ npm -v
```
以上命令在 `npm` 安装成功后会显示 `npm` 的版本号。
## 安装 Gitbook
当上面的工具都安装完成后打开命令行，执行以下命令安装 GitBook：
``` bash
$ npm install -g gitbook-cli
```
检测安装是否成功：
``` bash
$ gitbook-cli -v
```
以上命令在 `gitbook-cli` 安装成功后会显示 `gitbook-cli` 的版本号。

## 基础使用
创建第一本电子书
如果将第一本电子书的名字取名 mybook，那么创建该电子书的方法有两个，首先你可以创建一个文件夹名为 mybook 然后在终端打开该文件夹并执行下面的命令：
``` bash
$ gitbook init
```
或者你也可以直接在任一目录下打开终端执行以下命令：
``` bash
$ gitbook init mybook
```
创建好电子书后会在该文件下生成 README.md 和 SUMMARY.md 两个文件，其中 README.md 就是静态网站首页的内容，也就是电子书的封面，而 SUMMARY.md 是整个电子书的目录，为了一睹为快我们先执行下面的命令来本地查看以下书籍的样貌：
``` bash
$ gitbook serve
```
运行该命令后会启动本地服务器，通过 [http://localhost:4000/](http://localhost:4000/) 即可预览书籍，同时在书籍的文件夹中会生成一个 _book 文件夹, 里面的内容即为生成的 html 文件. 当然我们也可以使用下面命令来生成网页而不开启服务器：
``` bash
$ gitbook build
```

## 参考资料
* [Gitbook Github 地址](https://github.com/GitbookIO/gitbook)
* [Gitbook 工具链文档](https://toolchain.gitbook.com/)
* [官方插件](https://plugins.gitbook.com/)
* [网络插件整合](https://www.jianshu.com/p/427b8bb066e6)(收集大部分常用插件并给出了大部分插件的使用方式)
* [网络系列教程](http://gitbook.zhangjikai.com/)(完整介绍从安装到配置以及插件等的使用)