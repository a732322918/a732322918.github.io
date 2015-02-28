---
layout: default
title: foundation框架
---

1、NSObject +load伪代码：

{% highlight objc %}
	// 第一步
	void +[NSObject load](void * self, void * _cmd) {
    	rax = __NSInitializePlatform();
    	return;
	}

	// 第二步
	int __NSInitializePlatform() {
    LOBYTE(rax) = *(int8_t *)__NSInitializePlatform.inited;
    if (LOBYTE(rax) != 0x0) {
            LOBYTE(rax) = *(int8_t *)__NSInitializePlatform.inited;
            return rax;
    }
    else {
            *(int8_t *)__NSInitializePlatform.inited = 0x1;
            __CFInitialize();
            rax = __NSGetEnvironmentVariable("NSDebugEnabled");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    *(int8_t *)_NSDebugEnabled = 0x1;
            }
            rax = __NSGetEnvironmentVariable("NSZombieEnabled");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    *(int8_t *)_NSZombieEnabled = 0x1;
            }
            rax = __NSGetEnvironmentVariable("NSDeallocateZombies");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    *(int8_t *)_NSDeallocateZombies = 0x1;
            }
            rax = __NSGetEnvironmentVariable("NSDisableAutoreleasePoolCache");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    *(int8_t *)__NSDoAPCache = 0x0;
            }
            ___NSSetCStringCharToUnichar(0x0);
            __NSToDoAtProcessStart();
            rax = __NSGetEnvironmentVariable("NSDOLoggingEnabled");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    *(int8_t *)__NSDOLoggingEnabled = 0x1;
                    *(int8_t *)__NSDOLoggingEnabled2 = 0x1;
            }
            rax = __NSGetEnvironmentVariable("NSUnbufferedIO");
            if ((rax != 0x0) && (LODWORD(LOBYTE(LOBYTE(*(int8_t *)rax) | 0x20) & 0xff) == 0x79)) {
                    setvbuf(**__stdoutp, 0x0, 0x2, 0x0);
                    setvbuf(**__stderrp, 0x0, 0x2, 0x0);
            }
            _CFSetOutOfMemoryErrorCallBack();
            [NSThread currentThread];
            rax = __NSSetupDispatchDataBridge();
    }
    return rax;
	}

{% endhighlight %}