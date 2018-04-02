iOS的内存管理：使用引用计数，包括手工管理引用计数（MRC）和自动引用计数（ARC）。

引用计数（Reference Count）是一个简单而有效的管理对象生命周期的方式。当我们创建一个新对象的时候，它的引用计数为1，
当有一个新的指针指向这个对象时，我们将其引用计数加1，当某个指针不在指向这个对象时，我们将其引用计数减1，当对象的引用
变为0时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。

引用计数的使用场景：在面向对象的程序设计架构中，用于对象之间传递和共享数据。

----

ARC背后的原理：依赖编译器的静态分析能力，通过在编译时找出合理的插入引用计数管理代码，从而彻底释放程序员。<br>
ARC切换到MRC，在工程设置在文件后加上“-fno-objc-arc”。

ARC下的内存管理问题：<br>
1、循环引用<br>
（1）不正确的使用Block；<br>
（2）两个或两个以上的对象之间的相互引用；<br>
（3）NSTimer的不正确使用；<br>
 解决方法：弱引用（__weak 和 weak），主动断开循环引用。<br>
 检查方法：使用 Instruments 中的 Leaks ，或者使用 [MLeaksFinder](https://github.com/Tencent/MLeaksFinder) 在 debug 状态下检查 View 层的内存泄漏。

2、与 Core Foundation对象交互，需要手动管理 Core Foundation 对象的内存<br>
（1）底层的 Core Foundation 对象，在创建时大多以 XxxCreateWithXxx 这样的方式创建；
```
  // 创建一个 CFStringRef 对象
  CFStringRef str= CFStringCreateWithCString(kCFAllocatorDefault, “hello world", kCFStringEncodingUTF8);
  // 创建一个 CTFontRef 对象
  CTFontRef fontRef = CTFontCreateWithName((CFStringRef)@"ArialMT", fontSize, NULL);
```
（2）对于 Core Foundation 对象的修改，需要使用 CFRetain 和 CFRelease，和OC中的retain和release等价；
```
  // 创建一个 CTFontRef 对象
  CTFontRef fontRef = CTFontCreateWithName((CFStringRef)@"ArialMT", fontSize, NULL);
  // 引用计数加 1
  CFRetain(fontRef);
  // 引用计数减 1
  CFRelease(fontRef);
```
（3）CoreFoundation 和 OC 之间对象的转换，合理的使用\_\_bridge、\_\_bridge\_retained、\_\_bridge\_transfer<br>
    * \_\_bridge：只做类型转换，不修改相关对象的引用计数，原来的CoreFoundation 对象不在用时，需要调用 CFRelease 方法。
    * \_\_bridge\_retained：类型转换后，将相关对象的引用计数加1，原来的 Core Foundation 对象不在用时，需要调用 CFRelease 方法。
    * \_\_bridge\_transfer：类型转换后，将改对象的引用计数交个ARC管理，Core Foundation 对象在不用时，不在需要调用 CFRelease 方法。

参考：<br>
[理解iOS的内存管理](https://blog.devtang.com/2016/07/30/ios-memory-management)<br>
[MLeaksFinder](http://wereadteam.github.io/2016/07/20/MLeaksFinder2/)
