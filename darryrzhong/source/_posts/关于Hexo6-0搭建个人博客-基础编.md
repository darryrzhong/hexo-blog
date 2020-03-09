---
title: 关于Hexo6.0搭建个人博客(基础编)
date: 2019-03-27 10:56:07
tags: hexo
categories: hexo
---


>初识Hexo

 #  `什么是Hexo?`
`Hexo`是一个快速，简单而强大的博客框架。您可以使用[Markdown](http://daringfireball.net/projects/markdown/)（或其他语言）编写文章，Hexo可以在几秒钟内生成具有美丽主题的静态文件。
对的,就是这么简单明了,其实就是一个搭建博客的工具而已,也不是什么高深莫测的东西,接下来我将带着大家一步步徒手搭建属于自己的博客.`说明:(这里说的搭建属于自己的博客,并不是指搭建属于自己的独一无二的博客,那是前端的本事了,我们这里只说最基本的)`

# 1.安装Hexo
建立Hexo只需要几分钟,安装Hexo非常简单。但是，您首先需要安装其他一些东西.
*   [Node.js](http://nodejs.org/)
*   [Git](http://git-scm.com/)

如果你的电脑已经有这些，祝贺！只需使用npm安装Hexo：
> $ npm install -g hexo-cli

如果没有,那就一起来看下面吧(`这里只说windows的安装,因为本人太穷,买不起mac啊o(╥﹏╥)o`)

### 安装Git
![安装git.png](https://upload-images.jianshu.io/upload_images/5549640-6357f7847468f282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   Windows：下载并安装[git](https://git-scm.com/download/win)。
一切按照默认走就行了,没什么特殊的地方.

### 安装Node.js
![Nodejs安装.png](https://upload-images.jianshu.io/upload_images/5549640-affa652a4cf53b61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<!--more-->

我们这里简单点,直接下载并运行[安装程序](http://nodejs.org/)就完了.
还是走默认就行.

### 安装Hexo
一旦安装了所有需求，就可以用npm安装Hexo：
> $ npm install -g hexo-cli

随便找个地方, `Git Bash ->$ npm install -g hexo-cli` 就行了
我这里直接在桌面上GitBash
![GitBash.png](https://upload-images.jianshu.io/upload_images/5549640-b16e6205d228e21e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果出现以下就说明安装成功了
![success.png](https://upload-images.jianshu.io/upload_images/5549640-6114f3db7039eb70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 好了,到这里我们就算是已经成功完成了第一步:`安装Hexo`

# 2.利用Hexo初始化我们的站点跟目录(文件)
> $ hexo init <文件夹> 
$ cd <文件夹> 
$ npm install

选择你想要的盘符来建立我们的博客站点文件,我这里选择D:/
![init.png](https://upload-images.jianshu.io/upload_images/5549640-1564927dc14691b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里的`blog` 是你的文件夹名字,一般默认blog就要行了
看到下面的提示就说明站点目录建立成功了
![successHexo.png](https://upload-images.jianshu.io/upload_images/5549640-f431e9ac66e2e7a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后 初始化`$ npm install`

![install.png](https://upload-images.jianshu.io/upload_images/5549640-ad7551d9a4bdd023.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ok了,看看你的blog文件是不是这样的,是的话就对了
![blog.png](https://upload-images.jianshu.io/upload_images/5549640-ff6f52f6837e6ff4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在介绍两个命令 ,以后经常要用到的
> hexo generate（可简写为hexo g）
hexo sever（可简写为hexo s）
hexo clean

`hexo g`: 编译,生成静态文件,也就是public文件夹的东西
`hexo s `: 开启本地服务(以上两步的操作可以合并成hexo s -g)
`hexo' clean`:顾名思义就是清除缓存的意思了啦,这招一般在你改动之后网站没有变化时候用.

接下来看看 你博客的初步成果吧.
进入blog文件根目录:
`Git Bash ->$ hexo g`
 `$ hexo s`

![hexo g.png](https://upload-images.jianshu.io/upload_images/5549640-9e983a6bed9a4bbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![localhost.png](https://upload-images.jianshu.io/upload_images/5549640-b3c2dbbc1c767dee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在你的浏览器输入`http://localhost:4000`,查看你的博客.
![Myblog.png](https://upload-images.jianshu.io/upload_images/5549640-5918ee50f1c270cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 到此为止,你的个人博客就已经搭建完成了,是不是很简单

好了,关于怎么用Hexo搭建个人博客站点就介绍到这里了,后面我还会继续介绍[关于Hexo 
的主题优化](),[搭建漂亮的博客]、`利用Markdown写出第一遍博文`、`SEO优化等等一系列的教程`,希望能够帮到更多的小伙伴.
1.[关于Hexo6.0搭建个人博客(进阶篇)](https://www.jianshu.com/p/043a4ae7684d)
2.[关于Hexo6.0搭建个人博客(高级篇)](https://www.jianshu.com/p/52753aafd478)
### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)


