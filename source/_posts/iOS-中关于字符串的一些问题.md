---
title: iOS 中关于字符串的一些问题
date: 2015-9-30 14:08:09
tags: iOS
---
本文是看[Text Programming Guide for iOS](https://developer.apple.com/library/content/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009542-CH1-SW1)的简单总结。

<!-- more -->

先来一张图片看看排版中有关的概念。

{% asset_img concept.png %}

## 如何选择？
UIKit、TextKit、CoreText。如何选择？ 

大多数情况下，UIKit 可以满足你的需求。  

在 UILabe、UITextField、UITextView 下层有一个强大的排版引擎，就是 TextKit。如果你需要自定义排版过程或者要干预排版行为，那就使用 TextKit。TextKit 提供高质量的印刷服务（typographical service）让一ing用存储、排列（lay out）、展示排版特征，如字间距、换行符等。

如果这还不能满足你的需求，那就去用 Core Text 吧。TextKit是基于 Core Text 的。

## 排版有关的概念
### 字符与字形
字符（Characers）：这个比较抽象，直接举例子吧。比如，英文字符的大写“A”，汉字的“你”等。尽管自负通过一个显示区域内可识别的形状表示，但是字符并不等于就是那个形状。这个形状可以称为“字型”。
字形（Glyphs）：一个字符的不同表示形式。如大写字母A的字形：

	
{% asset_img GlyphsOfCharacterA.png %}

### 文字布局
直接来看个图好了。

{% asset_img GlyphMetrics.png %}

## 使用Textkit
TextKit 让你能够完全控制用户界面元素的文字渲染。
来看个图，这个图展示了 TextKit 与 iOS 中其他文字、图形框架的关系：

{% asset_img TextKitFrameworkPosition.png %}

### TextKit 中主要的几个对象
{% asset_img PrimaryTextKitObjcts.png %}

上面这个图展示了数据在 TextKit 的几个主要对象之间的流动。Text Storage 存储 Text View展示的文字，并且由 Layout Manager 排列到一个由 Text Container 定义的区域。

`NSTextContainer`：定义一个文字可以排列的区域。一般情况下是一个矩形区域。但是，你可以定义其子类来创建其他形状的区域，如圆形、五边形等。除此之外，TextContainer 还维护了一个 BezierPath 的数组来定义上述区域中不会被布局的部分。

`NSTextStorage`：定义基本的存储机制。`NSTextStorage`是'NSMutableAttributedString'的子类，存储并维护由文字系统操作的属性（attributes）。确保文字和属性在编辑操作时保持不变。除此之外，一个`NSTextStorage`对象维护一组`NSLayoutManager`的对象，并在有任何字符或者属性变化时通知这些布局对象，以便重新布局或者重新显示。

`NSLayoutManager`：协调其他的文字处理对象。它协调存储在`NSTextStorage`对象中的数据渲染成视图上的问题。它把 Unicode 字符映射为字形，并监督字形布局在`NSTextContainer`对象定义的区域内。

### 文字属性
文字属性主要由三类：字符属性、段落属性、文档属性。字符属性包括字体、颜色、角标等，这些属性可以跟一个范围内的字符相关联。段落属性包括缩进、行间距等。文档属性包括页面大小、页边距、页面缩放百分比等。

字符属性、段落属性平时都用的比较多，不多说。只是要注意，如果要使用自定义的属性，就要子类化`NSLayoutManager`。重写`drawGlyphsForGlyphRange:atPoint:`方法。

编辑属性字符串会造成矛盾的，需要通过`fixAttributesInRange:`来修正。

### 以编程的方式改变 Text Storage
主要分三个步骤：

1. 发送`beginEditing`消息；
2. 发送一些编辑消息。每发送一个消息，text storage就会调用`edited：range：changeInLength：`来追踪影响的范围；
3. 发送`endEditing`消息。这会造成它给代理发送`textStorage:willProcessEditing:range:changeInLength:`消息，并且调用自己的`processEditing`方法，以修复属性的改变。

### 布局文字
布局管理（NSLayoutManager）对象是 Text Kit 中文字显示的核心控制对象。布局管理对象执行以下这些操作：

* 控制文字存储（NSTextStorage）和文字容器(NSTextContainer)对象；
* 为字符生成字形；
* 计算字形位置并且存储这些信息；
* 管理字形的范围和字符(Manages ranges of glyphs and characters);
* 当 view 请求的时候，绘制字形；
* 计算每一行文字的边界矩形（Computes bounding box rectanle for lines of text）；
* 控制断字（hyphenation）；
* 操纵字符属性和字形属性。

布局管理对象执行文字布局分为两步：生成字形、排列字形。这两步都是惰性的，并且在计算完成后，会缓存这些信息，以提高后续调用时的性能。

## Core Text
> 如果使用 Core Text 或者 Core Graphics 来绘制文字，一定要记得对CTM(current text matrix)做变换。

{% asset_img CoreTextLayoutObjects.png %}

这里说几个需要注意的点。`CTFramesetterRef`需要用一个属性字符串和一个图形路径（graphics path）作为输入来初始化。framesetter 会生成一个或者多个frame。

内部生成frame的过程是这样的，framesetter调用typesetter对象来把属性字符串中的字符转化为字形，并把字形填进行，用行来填frame。

一行里边会包含一个或者几个`CTRun`对象，每个run是一串有相同属性和方向的字形。

Core Text 的具体使用，看唐巧的这篇文章吧：<http://blog.devtang.com/2013/10/21/the-tech-detail-of-ape-client-3/>

---
转载请注明出处：https://xjshi.github.io
