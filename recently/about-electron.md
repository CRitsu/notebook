# Electron

这篇文章是我在阅读官方文档和尝试使用Electron这个工具时记载的一些笔记，其中我摘抄了一些官方文档中我认为很关键的句子，也记载了一些使用中的细节和体会。

首先引用官方的介绍。

> Build cross platform desktop apps with JavaScript, HTML, and CSS

Electron，是一个使用JS、HTML、CSS来构建跨平台桌面应用的工具。

它将Chromium和Node.js合并到一个运行环境，并且打包成各个平台下可以运行的APP。

它是由GitHub的一个团队和一些活跃的贡献者一同维护的开源工具。

现在有很多基于Electron开发的应用，我最喜欢的轻量级编辑器——Microsoft的Visual Studio Code，以及Github的Atom Editor都是基于Electron构建的比较出名的APP。

事实证明了我们不需要质疑Electron的能力，当我们需要使用JS来创建一个应用，或者将我们现成的网站转化成桌面APP，Electron无疑是很好的选择。

[这是Electron的官方文档](https://electronjs.org/docs)

官方文档大部分都有中文翻译，比较方便。

# 需求

在阅读Electron的文档时首先确立我们的需求。

我们需要使用React构建UI，NeDB来持久化数据，这个APP运行在本地。

Electron在其中的角色是构建本地运行环境，以及制作软件包。

有几个问题需要解决。

**1. Electron的基本使用**

探索使用方法。

**2. 本地或服务端渲染**

在React组件中使用NeDB时，由于最终Build时Webpack将代码编译到浏览器环境中，此时NeDB将默认使用HTML5的IndexedDB储存数据，这对网页应用来说是合适的，但是作为桌面APP，我们希望数据可以保存在文件中方便备份或做版本控制。所以我们需要探索在Electron中如何在非浏览器环境下使用NeDB。



