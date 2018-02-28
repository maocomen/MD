# 上手 · SDWebImage 

### 前言

在 iOS 项目开发中，SDWebImage 这个第三方库应该是使用最广泛的第三方图片下载库；网上面有很多关于 SDWebImage 源码解析的 Blog，但是看了这么多，对内部实现原理，设计模式还是一知半解的。

所以现在还是要断笔重拾，换个角度来分析 SDWebImage 这个库，一方面可以开拓视野，增长见识，学习作者的编码思想；另一方面可以将我的思考写出来，让大神们指导指导。

之前我也尝试过分析 SDWebImage 的源码，但是当初的思路是：

> 分析 SDWebImage 类结构 -> 分析 SDWebImage 总体设计逻辑 -> 研究每个类的实现 -> 总结

后来发现，一开始就从宏观角度去分析，很难取得很好的成效，所以也就导致当初的计划烂尾；这次打算从目标 ”如何实现轻量级异步图片下载器” 入手，慢慢去分析 SDWebImnage 里面的实现细节。

### SDWebImage 介绍

GitHub 传送门 [SDWebImage](https://github.com/rs/SDWebImage)

#### 简介

SDWebImage  是一个支持缓存的异步图片下载器；为了使用方便，提供了大量的分类，例如：UIImageView、UIButton、MKAnnotationView等。

#### 特性

- 提供 UIImageView、UIButton、MKAnnotationView 分类来支持图片下载和缓存
- 异步图片下载
- 异步的内存+磁盘缓存图片，并且自动清除缓存过期的图片
- 图片解压缩
- 保证相同的 URL 不会被下载多次
- 保证错误的链接不会多次重复去请求
- 保证主线程永远不会被阻塞
- 性能卓越
- 使用 GCD 和 ARC

#### 支持的格式

- 支持 UIImage 所支持的格式，WebP 格式

#### 前置条件（这里只针对最新的 SDWebImage v4.3.1 版本）

- iOS 7.0 以上
- Xcode 7.3 以上

### 小结

以上就是对 GitHub 上面 SDWebImage 介绍的一些翻译，我打算从上面的特性一个一个的慢慢实现，然后再研究 SDWebImage 里面对于这些功能是如何实现的，并且思考自己与 SDWebImage 实现有什么却别，它的设计思想是什么，从而自我进步。

