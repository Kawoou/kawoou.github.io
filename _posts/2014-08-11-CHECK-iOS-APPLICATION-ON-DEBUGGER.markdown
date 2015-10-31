---
layout:     post
title:      "iOS 앱이 디버거 위에서 실행되는지 확인하기"
date:       2014-08-11 00:52:00
categories: Develop
tags:       ["Develop", "Debugger", "iOS", "Objc"]
---

요즘 나는 UrQA iOS 클라이언트를 개발하고 있다.<br />
이 서비스는 앱에서 버그가 발생할 경우 버그를 잡아내어<br />
서버로 보내주는 Crash Reporting Service이다.<br />

그래서 외부 사람들에게 배포하기 위해 Framework로 개발하던 중이었다.<br />

-

UrQA에서 버그를 캡처하는 방법은 Mach를 사용하여 Thread에서 버그를 캡처하고,<br />
그 Thread를 제외한 나머지 Thread를 일시정지시킨 상태에서 크래쉬 리포트를 작성한다.<br />
그리고 마지막으로 그 크래쉬 리포트를 전송 또는 저장한 후 **abort()**를 통해 앱을 종료시킨다.<br />
<br />

![]({{ site.baseurl }}/images/post/2014-08-11-CHECK-iOS-APPLICATION-ON-DEBUGGER-01.png)

-

그런데 이 과정에서 Xcode를 통해 Debugger가 연결된 상태에서 버그가 발생하면<br />
Debugger는 abort()가 호출된 그 위치를 버그가 발생한 위치로 판단해<br />
UrQA 서비스의 문제로 보이게 만드는 바람에 이 문제를 해결해야하는 상황이 발생한 것이다.<br />

그렇다고 이 문제를 DEBUG 플래그로 해결하자니<br />
배포시에는 Release 모드로 배포가 될테고,<br />
그렇다고 헤더파일에 volatile 변수를 만들어 그 변수를 사용하자니<br />
Release 모드더라도 Debugger가 연결될 수 있다는 문제가 발생했다.<br />

-

그렇게 난관에 빠져있는데 예전에 시스템 정보를 가져오는 **sysctl()** 함수가 생각났다.<br />
이 함수를 통해 프로세스의 플래그를 통해 디버거 상태를 받아올 수 있다는 것도 말이다.<br />

그래서 구글링을 해봤다.<br />
Mac Developer Library 링크가 바로 나왔다.<br />

-

끝!<br />
<br />

<br />

**[  Source Code  ]**

```cpp
bool beingDebugged(void)
{
    int                 junk;
    int                 mib[4];
    struct kinfo_proc   info;
    size_t              size;
     
    info.kp_proc.p_flag = 0;
     
    mib[0] = CTL_KERN;
    mib[1] = KERN_PROC;
    mib[2] = KERN_PROC_PID;
    mib[3] = getpid();
     
    size = sizeof(info);
    junk = sysctl(mib, sizeof(mib) / sizeof(*mib), &info, &size, NULL, 0);
    if (junk != 0)
        return YES;
     
    return ((info.kp_proc.p_flag & P_TRACED) != 0);
}
```

<br />

---

**Reference:**<br />

https://developer.apple.com/library/mac/qa/qa1361/_index.html<br />
<br />
