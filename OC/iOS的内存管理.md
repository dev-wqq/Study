iOS的内存管理：使用引用计数，包括手工管理引用计数（MRC）和自动引用计数（ARC）。

引用计数（Reference Count）是一个简单而有效的管理对象生命周期的方式。当我们创建一个新对象的时候，它的引用计数为1，
当有一个新的指针指向这个对象时，我们将其引用计数加1，当某个指针不在指向这个对象时，我们将其引用计数减1，当对象的引用
变为0时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。

引用计数的使用场景：在面向对象的程序设计架构中，用于对象之间传递和共享数据。

ARC切换到MRC，在工程设置在文件后加上“-fno-objc-arc”。

ARC背后的原理：依赖编译器的静态分析能力，通过在编译时找出合理的插入引用计数管理代码，从而彻底释放程序员。

ARC下的内存管理问题：
1、循环引用
（1）不正确的使用Block；
（2）两个或两个以上的对象之间的相互引用；
（3）NSTimer的不正确使用；
 解决方法：弱引用（__weak 和 weak），主动断开循环引用。
 检查方法：使用Instruments中的Leaks，或者使用MLeaksFinder在debug状态下检查View成的内存泄漏。

2、与CoreFoundation对象交互，需要手动管理，CoreFoundation对象的内存
（1）底层的CoreFoundation 对象，在创建时大多以XxxCreateWithXxx 这样的方式创建；
```
  // 创建一个 CFStringRef 对象
  CFStringRef str= CFStringCreateWithCString(kCFAllocatorDefault, “hello world", kCFStringEncodingUTF8);
  // 创建一个 CTFontRef 对象
  CTFontRef fontRef = CTFontCreateWithName((CFStringRef)@"ArialMT", fontSize, NULL);
```
（2）对于CoreFoundation 对象的修改，需要使用 CFRetain 和 CFRelease，和OC中的retain和release等价；
```
  // 创建一个 CTFontRef 对象
  CTFontRef fontRef = CTFontCreateWithName((CFStringRef)@"ArialMT", fontSize, NULL);
  // 引用计数加 1
  CFRetain(fontRef);
  // 引用计数减 1
  CFRelease(fontRef);
```

参考：

[理解iOS的内存管理](https://blog.devtang.com/2016/07/30/ios-memory-management)
