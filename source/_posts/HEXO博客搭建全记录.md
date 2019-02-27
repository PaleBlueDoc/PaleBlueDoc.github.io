---
title: HEXO博客搭建全记录
date: 2019-02-27 15:47:25
categories: 技术杂记
tags:
	- hexo
	- github
---


## 搭建个人博客的好处

* 培养个人性情

在当前这个生活节奏越来越快的时代中，每个人都希望速成。 像读书、练字、写博客这种需要长期坚持的事对个人的性情培养就显得弥足珍贵。

* 锻炼自己的文采

每个人都有自己的想法，想要说清楚已经不易，想要写出来让人看明白更难，想要让人喜欢看，愿意看...嗯，这是门艺术。

* 帮你记录生活和知识

我对《活着》中的一段话有着很深的印象

> 可是我再也没遇到一个像福贵这样令我难忘的人了，对自己的经历如此清楚，又能如此精彩地讲述自己。他是那种能够看到自己过去模样的人，他可以准确地看到自己年轻时走路的姿态，甚至可以看到自己是如何衰老的。
>
>这样的老人在乡间实在难以遇上，也许是困苦的生活损坏了他们的记忆，面对往事他们通常显得木讷，常常以不知所措的微笑搪塞过去。他们对自己的经历缺乏热情，仿佛是道听途说般地只记得零星几点，即便是这零星几点也都是自身之外的记忆，用一、两句话表达了他们所认为的一切。在这里，我常常听到后辈们这样骂他们： “一大把年纪全活到狗身上去了。” 
>
>———余华《活着》

怀着对“一大把年纪全活到狗身上去了”这句话的恐惧，我决定还是要写个博客。

<!--more-->

接下来进入正题


## HEXO + github pages博客搭建环境及优势

* github pages服务，完全免费的静态文件服务器、响应速度快、可以绑定域名、可以用git进行版本管理
 
* 2.hexo博客框架，支持markdown编辑、安装部署方便、有丰富的插件、支持git一键部署


## 搭建过程

搭建的过程及准备工作我基本都是根据小茗同学的博客 [使用hexo+github搭建免费个人博客详细教程](http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa) 来搭建完成的。我在此只记录遇到的问题及参考的文献

## 遇到的问题

* Nodejs版本我下载的是v10.15.1，自带的NPM版本是v6.4.1在执行npm install hexo 时会爆出一些警告但并不影响使用




    $ npm install -g hexo
    
    npm WARN deprecated titlecase@1.1.2: no longer maintainedD:\nodejs\node_global\hexo -> D:\nodejs\node_global\node_modules\hexo\bin\hexo
    
    npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.7 (node_modules\hexo\node_modules\fsevents):
    
    npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.7: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
    
    + hexo@3.8.0
    
    added 2 packages from 2 contributors, removed 205 packages, updated 11 packages and moved 4 packages in 41.752s
    


    

* npm 默认的全局地址是在c盘，所以修改npm的默认全局地址及修改资源仓库 [npm 更改默认全局路径以及国内镜像](https://blog.csdn.net/chenhaifeng2016/article/details/64128095)

* 由于hexo发布到github pages上的代码只是public编译后的代码，所以如果想要在多台电脑上编辑的话还需要将hexo的代码也放到git上进行管理。这里参考文章 [多台电脑使用Hexo](https://www.jianshu.com/p/4bcf2848b3fc) 进行处理

* 写博客时我选择的是markdownpad2，在win10中会提示awesomium无法正常显示。需要下载awesomium_v1.6.6_sdk_win安装后可以正常展示。在这里附上百度云连接，不想麻烦的朋友可以直接下载 [awesomium_v1.6.6_sdk_win.zip](https://pan.baidu.com/s/1-fFxN4bWQlKnjYR9rVm1dg) 

## 在自己的服务器上搭建HEXO

在搭建过程中发现HEXO插件库中有后台管理插件 [hexo-admin](https://github.com/jaredly/hexo-admin) 或者 [hexo-admin-ehc](https://github.com/lwz7512/hexo-admin-ehc)
后者貌似比前者在编辑文章时多了上传图片的功能，而且都支持markdown在线编辑。

使用插件可以在网页上方便的添加和编写文章、选择分类和设置tags等功能。这样的话可以使用nginx代理hexo编译好的public目录，在网页上编写文章，省去了提交的步骤，想想也是很不错的。当然这需要有一个自己的服务器

## 绑定数据统计

hexo的数据统计配置方式根据使用的theme主题配置方式可能不同，但应该都是配置一下ID就可以。
我使用的是next主题，官方文档上有所有的统计分析配置方式 [数据统计与分析](http://theme-next.iissnan.com/third-party-services.html#analytics-system)