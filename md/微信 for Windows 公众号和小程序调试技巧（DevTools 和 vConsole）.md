> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1789262-1-1.html)

> [md]# 微信 for Windows 公众号和小程序调试技巧（DevTools 和 vConsole）## 前言技术发展日新月异。

![](https://avatar.52pojie.cn/data/avatar/000/31/60/54_avatar_middle.jpg)周易 _ 本帖最后由 周易 于 2023-5-24 14:27 编辑_  

微信 for Windows 公众号和小程序调试技巧（DevTools 和 vConsole）
===============================================

前言
--

技术发展日新月异。微信 for Android 早已告别 x5，拥抱 xweb，许多教程也慢慢跟上了这一改变。然而，对于同样发展的微信 for Windows，许多教程的改变只是结尾增加了一句建议回滚使用老版本。仿佛 DevTools 从新版本微信消失了似的。其实并不尽然，新版本不仅提供了 DevTools，还提供了 vConsole。

公众号调试技巧
-------

公众号调试常见的场景例如部分设备排版行为不一致、文字显示不全，需要审查元素。这可以通过 DevTools 完成。难点在于如何打开 DevTools。对于打开 DevTools，存在大量教程，但许多都针对老版本甚至直接建议回滚版本。

对于新版本微信，可以使用如下启动参数。

```
--enable-chrome-inspector

```

小程序调试技巧
-------

微信开发者工具事实上提供了相当完备的调试功能，因此需要的场景并不多。

如果需要使用 vConsole，可以使用如下启动参数。

```
--enable-vconsole

```

附录：修改启动参数
---------

部分萌新可能不知道如何修改启动参数，因此介绍一种简单的方式。首先复制微信快捷方式，属性中修改即可。