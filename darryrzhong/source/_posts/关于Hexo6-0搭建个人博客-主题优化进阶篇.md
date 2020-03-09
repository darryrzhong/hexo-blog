---
title: 关于Hexo6.0搭建个人博客(主题优化进阶篇)
date: 2019-03-27 11:00:57
tags: hexo
categories: hexo
---

>本篇博文将带大家发现新大陆,教你打造炫酷的个人博客站点.

阅读本文前建议先行阅读本人另外一遍基础博文[关于Hexo搭建个人博客(基础篇)](https://www.jianshu.com/p/d574962baa16)

#目录
* 配置博客基本信息
* 配置主题
* 优化主题
    * 上传头像,并设置头像旋转效果
    * 设置个人社交图标链接
    * 设置RSS
   *  设置酷炫动态背景
  * 设置主题语言
  * 设置网站logo
  *  设置左上角或者右上角的fork me on github效果
  *  设置顶部滚动加载条
  * 自定义博客底部显示效果
  * 为首页文章添加阴影边框效果
  * 为首页添加自定义菜单栏标签

# 配置博客基本信息

> 在站点根目录下`_config.yml`中进行基础配置
建议下载个文本编辑器打开,这里推荐`Sublime Text`,

![set1.png](https://upload-images.jianshu.io/upload_images/5549640-482e1976e130e564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对应显示效果(显示效果因主题不同而不同,只做描述)
![show.png](https://upload-images.jianshu.io/upload_images/5549640-bd3bc18775a42712.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![show1.png](https://upload-images.jianshu.io/upload_images/5549640-e5ba02014b1ffbcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当然了,你们的主题和我的肯定是不一样的,所以下面就开始教大家挑选自己喜欢的主题
并自定义个人喜好.
# 配置主题
> 挑选主题

<!--more-->


你可以点击这里选择你喜欢的[Themes](https://hexo.io/themes/),里面有大量美观的主题

![thems.png](https://upload-images.jianshu.io/upload_images/5549640-cdd07a4736fb7e61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 主题很多,可以慢慢挑选,挑选好了,直接`clone`下来就好了.

我这里以简约著称的`Next`主题为例讲解,本人用的也是这款主题,当然喜欢炫酷一点的小伙伴们可以自行选择,只是优化会略有不同.
![next.png](https://upload-images.jianshu.io/upload_images/5549640-fe9ac26f718e6b42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里有下载主题有两种方式:
1.通过get 命令方式直接安装
2.直接把整个仓库clone到本地,再移动到你的站点目录Themes下

这里主要教大家用命令行方式
>$ cd hexo
$ git clone https://github.com/theme-next/hexo-theme-next themes/next

进入hexo根目录下,`GitBash`

![clone.png](https://upload-images.jianshu.io/upload_images/5549640-51f908629d495d42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

clone成功后,你的Themes文件夹下就会有next主题文件了
![nextfile.png](https://upload-images.jianshu.io/upload_images/5549640-c0edcb59f5481b7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先介绍下next文件夹下`_config.yml`文件,以后我们凡是修改主题,都是修改这个文件

然后下一步,我们先应用我们刚才clone下来的主题,看看什么效果

打开blog(`你的博客站点跟目录`)下的_config.yml文件进行设置:
将`theme: landscape `修改为 `theme: next`
![config_next.png](https://upload-images.jianshu.io/upload_images/5549640-ae67afe409b32a2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后`hexo g`和` hexo s `一下 ,我们来看一下效果:
![nextshow.png](https://upload-images.jianshu.io/upload_images/5549640-9b1aafc9f9b44a80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>好了 ,没错就是这个样子(的确是丑了点,所以需要你自己去优化成你自己想要的样子啦)

当然你也可以直接clone我配置好了的主题[darryrzhong-next](https://github.com/darryrzhong/hexo-theme-next).顺便star一下最好啦.


# 优化主题
到现在我们基础博客算是搭建完成了,下面手把手教大家美化主题,赶快上车吧

带大家优化主题之前,先介绍下next目录结构,了解目录结构能帮助我们更快的优化
```
.
├── .deploy
├── public
├── scaffolds
├── scripts
├── source
|   ├── _drafts
|   └── _posts
├── themes
├── _config.yml
└── package.json
```

最新的next已经把我们需要的大部分功能都已经集成了,所以我们需要修改的文件只有那么几个
* deploy：执行hexo deploy命令部署到GitHub上的内容目录
* public：执行hexo generate命令，输出的静态网页内容目录
* `source：`站点资源目录,你写的文章,需要的素材等等都是放在这个目录下,包括以后
你需要新建菜单标面目录也是放在这里.
* drafts：草稿文章
* posts：成功发布的文章目录(你发布的文章都在这里面可以看到 )
* `themes：`主题文件目录
* `_config.yml：`全局配置文件，大多数的设置都在这里
常用的需要了解的就是这些目录了,其中高亮的地方就是整个next的核心部分了


# 1. 上传头像,并设置头像旋转效果
设置头像:
 打开`themes/next/_config.yml`找到`avatar: /images/avatar.gif`;
其中`images`文件在`themes/next\source\`中,将你的头像图片放到images中,一般默认
命名为avatar,记得改下后缀就可以了.
设置旋转效果:
打开`themes\next\source\css\_common\components\sidebar\sidebar-author.styl`,
添加以下注释代码
```
.site-author-image {
  display: block;
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;

   /* 头像圆形 */
  border-radius: 80px;
  -webkit-border-radius: 80px;
  -moz-border-radius: 80px;
  box-shadow: inset 0 -1px 0 #333sf;

  /* 鼠标经过头像旋转时间 */
  -webkit-transition: -webkit-transform 1.0s ease-out;
  -moz-transition: -moz-transform 1.0s ease-out;
  transition: transform 1.0s ease-out;


}

 img:hover {
  /* 鼠标经过停止头像旋转 
  -webkit-animation-play-state:paused;
  animation-play-state:paused;*/

  /* 鼠标经过头像旋转360度 */
  -webkit-transform: rotateZ(360deg);
  -moz-transform: rotateZ(360deg);
  transform: rotateZ(360deg);
}

```
做完这一步,头像也就设置完成了.
![avatar1.png](https://upload-images.jianshu.io/upload_images/5549640-9f456601e84520fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 2.设置个人社交图标链接

![social.png](https://upload-images.jianshu.io/upload_images/5549640-45b9b5c535dbdcaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 打开`themes/next/_config.yml`,找到`social`;
![social1.png](https://upload-images.jianshu.io/upload_images/5549640-20c5861e4294c883.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中`name`为你图标的名字,你可以去[Font Awesome](http://www.bootcss.com/p/font-awesome/#)中挑选你喜欢的,然后复制名字就可以了
如 icon-user-md 只需要user-md就可以了,next里面已经集成好了.如果next找不到图标的话,图标就会被一个问号所取代.

# 3.设置RSS
![rss.png](https://upload-images.jianshu.io/upload_images/5549640-250cccfcbe0a9eb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

来到你的站点根目录(`比如我的blog/`)下;
执行 GitBash 命令 : `$ npm install --save hexo-generator-feed`安装插件,插件会放在
`node_modules`文件夹里面.

安装好插件后,打开全局配置文件`_config.yml`,在末尾加入以下代码:

```
# Extensions
## Plugins: http://hexo.io/plugins/
plugins: hexo-generate-feed
```
然后打开主题配置文件`_config.yml`,找到`rss`;
添加配置:`rss: /atom.xml `;
到这里就行了,重新运行一下 `hexo g `,`hexo s ` ,你就会看到效果了.


# 4.设置酷炫动态背景
![background.png](https://upload-images.jianshu.io/upload_images/5549640-ecff9cc69ba138dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新版本的next已经支持canvas-nest了,所以直接添加代码就可以了
打开`next/layout/_layout.swig`文件在</body>之前加上以下代码;

```{% if theme.canvas_nest %}
<script type="text/javascript" src="//cdn.bootcss.com/canvas-nest.js/1.0.0/canvas-nest.min.js"></script>
{% endif %}
```
打开主题配置文件`_config.yml`,  修改以下代码;

```
# Canvas-nest
# Dependencies: https://github.com/theme-next/theme-next-canvas-nest
canvas_nest: true
```

重新运行下就可以了,

# 5.设置主题语言
你的博客首页显示的默认是英文名，而我们想要有一个中文的名字的话,就需要设置下语言显示;
我们可以在next\language文件下的zh-CN（中文）语言包下增加相应的字段,还可以修改其他的字段;
```
title:
  archive: 归档
  category: 分类
  tag: 标签
  schedule: 日程表
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  schedule: 日程表
  sitemap: 站点地图
  commonweal: 公益 404
sidebar:
  overview: 站点概览
```
改成我们自己想要的显示效果后.在全局配置文件`_config.yml`中重新配置,
`language:  zh-CN `,重新运行下服务器记得,只要是全局效果,都需要重新clean一下,在重新运行下,否则看不到效果.

# 6.设置网站logo
效果如下:
![logo.png](https://upload-images.jianshu.io/upload_images/5549640-29eeceed36f1aa40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 这个小图标就是你的logo了.
打开主题配置文件`_config.yml`    ,找到字段`favicon:`;
```
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
```
可以看到有四种效果,一般我们只需将medium换成我们自己图标路径就行了.
对于图标大小也是有要求的,看实例就知道了,就不做过多的说明了.
# 7.设置左上角或者右上角的fork me on github效果
显示效果如下:
![fork me github.png](https://upload-images.jianshu.io/upload_images/5549640-db9ba6926349302a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
实现方法如下:
当然有很多种颜色和效果,你可以在[这里](https://blog.github.com/2008-12-19-github-ribbons/)选择你需要的效果;然后复制你选择效果右侧的代码;
```
<a href="https://github.com/you"><img style="position: absolute; top: 0; left: 0; border: 0;" src="https://s3.amazonaws.com/github/ribbons/forkme_left_red_aa0000.png" alt="Fork me on GitHub"></a>
```
打开`next\layout\_layout.swig`,将刚才复制的代码黏贴到`<div class="headband"></div>`的下面;

![github.png](https://upload-images.jianshu.io/upload_images/5549640-d3027a15a753d407.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重新运行下,看看效果吧;

# 8. 设置顶部滚动加载条
效果如下:
![bar.png](https://upload-images.jianshu.io/upload_images/5549640-c14b7b4139476942.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开`next\layout\_partials\head`文件，添加以下代码:
```
<script src="//cdn.bootcss.com/pace/1.0.2/pace.min.js"></script>
<link href="//cdn.bootcss.com/pace/1.0.2/themes/pink/pace-theme-flash.css" rel="stylesheet">
```
![barset.png](https://upload-images.jianshu.io/upload_images/5549640-125852e11f2f1929.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上加载条就可以正常显示了,默认颜色是粉色的,本人觉得粉色的也挺好看就没改了,
当然你可以自定义你想要的颜色,做法如下:
在刚才添加的代码后面再加上以下代码:
```
<style>
    .pace .pace-progress {
        background: #1E92FB; /*进度条颜色*/
        height: 3px;
    }
    .pace .pace-progress-inner {
         box-shadow: 0 0 10px #1E92FB, 0 0 5px     #1E92FB; /*阴影颜色*/
    }
    .pace .pace-activity {
        border-top-color: #1E92FB;    /*上边框颜色*/
        border-left-color: #1E92FB;    /*左边框颜色*/
    }
</style>
```
自定义加载条就大功改成了;
# 9.自定义博客底部显示效果
![bottom.png](https://upload-images.jianshu.io/upload_images/5549640-d8397b41d9235db7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体实现如下:
1.隐藏网页底部powered By Hexo / 强力驱动
打开`themes/next/layout/_partials/footer.swig`,直接隐藏以下代码即可,建议不要删除
代码如下:
```
<!--{% if theme.footer.powered.enable %}
  <div class="powered-by">{#
  #}{{ __('footer.powered', '<a class="theme-link" target="_blank"' + nofollow + ' href="https://hexo.io">Hexo</a>') }}{% if theme.footer.powered.version %} v{{ hexo_env('version') }}{% endif %}{#
#}</div>
{% endif %}

{% if theme.footer.powered.enable and theme.footer.theme.enable %}
  <span class="post-meta-divider">|</span>
{% endif %}

{% if theme.footer.theme.enable %}
 <div class="theme-info">{#
  #}{{ __('footer.theme') }} &mdash; {#
  #}<a class="theme-link" target="_blank"{{ nofollow }} href="https://github.com/theme-next/hexo-theme-next">{#
    #}NexT.{{ theme.scheme }}{#
  #}</a>{% if theme.footer.theme.version %} v{{ version }}{% endif %}{#
#}</div>
{% endif %}-->
```

2.修改网页底部的桃心

还是在`themes/next/layout/_partials/footer.swig`中，找到字段`with-love`;
```
<span class="with-love" id="animate">
    <i class="fa fa-{{ theme.footer.icon.name }}"></i>
  </span>
```
然后在[图标库](http://www.bootcss.com/p/font-awesome/#)中找到你想要的图标,修改如下:
```
<span class="with-love" id="animate">
    <i class="fa fa-heart"></i>
  </span>
```
其中heart为图标名字,只要icon-后面的就行了;

3.实现网站底部访问量显示
打开主题配置文件` _config.yml `,
修改如下代码
```
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user 
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```
  
# 10.为首页文章添加阴影边框效果
打开`next\source\css\_custom\custom.styl`文件,添加以下代码:
```
// 主页文章添加阴影效果
 .post {
   margin-top: 60px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
  }
```
重新运行下服务器,效果如下:
![shandown.png](https://upload-images.jianshu.io/upload_images/5549640-4cd76ff64eba4643.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 11.为首页添加自定义菜单栏标签
效果如下:
![menu.png](https://upload-images.jianshu.io/upload_images/5549640-05e159c9ef5842e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现如下:
进入站点根目录下,使用`GitBash`新建页面;
>$ hexo new page "music"
$ hexo new page "photo"
$ hexo new page "welfare"

这里我新建三个菜单标签,分别是音乐、摄影、福利.
![newPage.png](https://upload-images.jianshu.io/upload_images/5549640-aa5ab1f48c4f6296.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到,新建页面在source文件对于的文件夹里;

打开主题配置文件`_config.yml`找到字段`menu`,添加对应菜单;
![menu1.png](https://upload-images.jianshu.io/upload_images/5549640-842d5d2ec91aaf01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加完菜单标签项后,效果如下:
![menu2.png](https://upload-images.jianshu.io/upload_images/5549640-81c13462a61274a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到标签名还是英文的,并且pohto项是空白的,之前已经说过,空白代表这个图标找不到,所以换一个就行了,下面我们来完善一下,将标签名改为中文;

打开`next\language文件下的zh-CN`（中文）语言包,添加菜单标签项;
```
menu:
  home: 首页
  photo: 摄影
  music: 音乐
  welfare: 福利
  archives: 归档
  categories: 分类
```
![menu3.png](https://upload-images.jianshu.io/upload_images/5549640-f67c10898f2f31ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

名称一定要和我们主题中的名称一致,到这里我们的菜单栏也就完成了.

>到这里,关于博客的基础搭建也就全部做好了

本文只要介绍了博客外观美化的一些小技巧,使得我们的博客看起来更加的美观,关于博客内文章的美观及评论留言区等的优化、以及使用markdown写下第一篇博文等将会在下一篇高级篇手把手教大家,欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

1.[关于Hexo6.0搭建个人博客(高级篇)](https://www.jianshu.com/p/52753aafd478)
### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)


