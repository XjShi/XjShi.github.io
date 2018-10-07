---
title: 使用 Quartz 2D 擦除图片
date: 2015-06-30 15:59:19
tags: iOS
---
Quartz 2D 是一个强大的二位图像绘制引擎，在开发中如果遇到需要高度自定义的控件，我们就可能需要用Core Graphics进行绘制。

这几天一同事开发一个聊天中的一个子模块，A 画一幅图，然后发给 B。这需要使用到 Core Graphics，我看了他的代码，图形绘制、填充等 API 使用当然没有问题，但是在需要擦出的时候，他的实现方式是粗暴的绘制跟背景相同颜色的线条。如此不优雅的实现方式，当然不能说服我。

那么优雅的实现方式是什么？通过肉眼枚举`CGContext.h`中的 API，我找到了这个方法`CG_EXTERN void CGContextClearRect(CGContextRef cg_nullable c, CGRect rect)`。

<!-- more -->

下边直接通过代码来了解一下，这里我通过擦除一张图片的一部分来举例。

~~~objc
- (void) touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
{
     UITouch *touch = [touches anyObject];
     CGPoint point = [touch locationInView:imageView];

     UIGraphicsBeginImageContext(bingbingImageView.frame.size);
     [imageView.image drawInRect:bingbingImageView.bounds];
     
     CGRect rect = CGRectMake(point.x, point.y, 20.0f, 20.0f);
     CGContextClearRect(UIGraphicsGetCurrentContext(), rect);
     imageView.image = UIGraphicsGetImageFromCurrentImageContext();
     UIGraphicsEndImageContext();
}
~~~

看下效果：

擦除前：
{% asset_img before.png %}

擦除后：
{% asset_img after.png %}