---
layout:     post
title:      "iOS에서 아이콘 폰트 로드하기"
author:     "Kawoou"
date:       2013-12-18 20:04:00
categories: Develop
tags:       ["CoreText", "iOS", "Programming", "아이콘폰트", "폰트"]
---

얼마전 Alarmy를 작업하면서 용량과 깨지지 않는 디자인을 위해 아이콘 폰트를 만들어 사용하게 되었다.<br />
하지만 암호화되지 않은 OTF파일을 제한하는지 일반적인 방법으로 로드가 되지 않아서 포기한 적이 있었다.<br />
(일반적인 방법이란 Info.plist의 UIAppFonts키에 등록하는 방법을 말한다)<br />

하지만, 여러 장점을 가지고 있는 아이콘 폰트를 포기할 순 없었고, 그래서 Apple Reference를 찾아보았다.<br />
그러던 도중 CoreText.framework에서 CTFontManagerRegisterGraphicsFont라는 함수를 제공함을 알게되었다.<br />

외부 폰트를 사용하는 방법은 간단했다.<br />
<br />

**1. 먼저 폰트를 리소스에 등록한다.**

![]({{ site.baseurl }}/images/post/2013-12-18-iOS-icon-font-load-01.png)

<br />

**2. CoreText.framework를 추가한다.**

![]({{ site.baseurl }}/images/post/2013-12-18-iOS-icon-font-load-02.png)

<br />

**3. AppDelegate에 헤더 파일을 로드시킨다.**

```Objective-C
#import <CoreText/CoreText.h>
```

<br />

**4. 폰트를 로드하는 코드를 작성한다.**

```Objective-C
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        CFErrorRef error;
        NSURL *url = [[NSBundle mainBundle] URLForResource:@"iconFont" withExtension:@"otf"];
        CGDataProviderRef fontDataProvider = CGDataProviderCreateWithURL((__bridge CFURLRef)url);
        CGFontRef newFont = CGFontCreateWithDataProvider(fontDataProvider);
         
        if(!CTFontManagerRegisterGraphicsFont(newFont, &error))
        {
            CFStringRef errorDescription = CFErrorCopyDescription(error);
            NSLog(@"Failed to load font: %@", errorDescription);
            CFRelease(errorDescription);
        }
        CGFontRelease(newFont);
        CGDataProviderRelease(fontDataProvider);
    });
}
```

<br />

---

**Reference:**

1. [Core Text Font Manager Reference](https://developer.apple.com/library/mac/documentation/Carbon/Reference/CoreText_FontManager_Ref/Reference/reference.html#//apple_ref/c/func/CTFontManagerRegisterGraphicsFont)