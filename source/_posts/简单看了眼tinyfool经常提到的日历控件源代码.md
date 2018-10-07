---
title: 简单看了眼tinyfool经常提到的日历控件源代码
date: 2016-11-27 17:11:26
tags: 胡说八道
---
先贴上源码地址：[Here](https://code.google.com/archive/p/iphonecal/source/default/source)

代码是是08年写的，这点还是非常服的，这个时候我还在上高中。08年到现在OC在一些特性上改进很多，其中使用的`CFGregorianDate`相关api在iOS 8.0之后也不建议使用了，所以这里我只说我确认不是好的实践的部分。

<!-- more -->

**先来看下头文件:**

~~~objc
//
//  CalendarView.h
//  ZhangBen
//
//  Created by tinyfool on 08-10-26.
//  Copyright 2008 __MyCompanyName__. All rights reserved.
//

#import <UIKit/UIKit.h>

@protocol CalendarViewDelegate;

@interface TdCalendarView : UIView {
	CFGregorianDate currentMonthDate;
	CFGregorianDate currentSelectDate;
	CFAbsoluteTime	currentTime;
	UIImageView* viewImageView;
	id<CalendarViewDelegate> calendarViewDelegate;
	int *monthFlagArray; 
}

@property CFGregorianDate currentMonthDate;
@property CFGregorianDate currentSelectDate;
@property CFAbsoluteTime  currentTime;

@property (nonatomic, retain) UIImageView* viewImageView;
@property (nonatomic, assign) id<CalendarViewDelegate> calendarViewDelegate;
-(int)getDayCountOfaMonth:(CFGregorianDate)date;
-(int)getMonthWeekday:(CFGregorianDate)date;
-(int)getDayFlag:(int)day;
-(void)setDayFlag:(int)day flag:(int)flag;
-(void)clearAllDayFlag;
@end



@protocol CalendarViewDelegate<NSObject>
@optional
- (void) selectDateChanged:(CFGregorianDate) selectDate;
- (void) monthChanged:(CFGregorianDate) currentMonth viewLeftTop:(CGPoint)viewLeftTop height:(float)height;
- (void) beforeMonthChange:(TdCalendarView*) calendarView willto:(CFGregorianDate) currentMonth;
@end
~~~

代码格式我就不说了，空格写的很随意，大小写对于我这种强迫症来说也很受不了。最重要的是我认为这雨苹果framework中命名方式大不同。
interface中的命名大多数都还不错，但是`-(void)setDayFlag:(int)day flag:(int)flag;`这个是什么鬼，我认为`- (void)setFlag:(int)flag forDay:(int)day;`更好。

**下面来看下源文件：**

源文件中定义了全局的几个变量：

~~~objc
const float headHeight=60;
const float itemHeight=35;
const float prevNextButtonSize=20;
const float prevNextButtonSpaceWidth=15;
const float prevNextButtonSpaceHeight=12;
const float titleFontSize=30;
const int	weekFontSize=12;
~~~

这几个变量主要是来控制试图的大小、字体的大小，只有在这个文件内部使用，但没有用`static`修饰。

**学C语言的时候，大概都做过这么一个练习：*获取某年某月的天数*。来看看tinyfool的实现：**

~~~objc
-(int)getDayCountOfaMonth:(CFGregorianDate)date{
	switch (date.month) {
		case 1:
		case 3:
		case 5:
		case 7:
		case 8:
		case 10:
		case 12:
			return 31;
			
		case 2:
			if((date.year % 4==0 && date.year % 100!=0) || date.year % 400==0)
				return 29;
			else
				return 28;
		case 4:
		case 6:
		case 9:		
		case 11:
			return 30;
		default:
			return 31;
	}
}
~~~

这样的实现没有问题，但我认为查表是更好的方式。  
<font color=red>Update: 微博上有讨论这个问题，表示这样写不会有什么问题，编译器会对这种情况做优化。</font>

**实现中有很多类似的实践（个人认为这不是好的实践）：**

~~~objc
- (void)movePrevMonth{
	if(currentMonthDate.month>1)
		currentMonthDate.month-=1;
	else
	{
		currentMonthDate.month=12;
		currentMonthDate.year-=1;
	}
	[self movePrevNext:0];
}
~~~

**再来看下这个类的析构函数：**

~~~objc
- (void)dealloc {
    [super dealloc];
    free(monthFlagArray);
}
~~~

Excuse me. 这里虽然不太可能出现崩溃的情况，但不是应该先释放子类持有的资源？  
<font color=red>Update: tinyfool说在早期SDK的bug比较多，要实验各种方法避免内存泄漏，有些写法比较诡异，但是在某个版本下，反而效果是好的。</font>

**最后，总结一些tinyfool的变量命名风格：`title_Month`、`weekfont`、`tabHeight`、`s_width`。**

---
Update: 之前写的时候描述不太恰当，因此修改了一部分。