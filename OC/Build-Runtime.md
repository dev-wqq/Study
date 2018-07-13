如何编译 runtime（objc4-723）?
---
下载源码、解压
> * [选择系统版本](https://opensource.apple.com/) ，分别下载：obj4、Libc(825.40.1)、dyld、libauto、libclosure、libpthread、xnu;
> * [开源项目Tarballs目录](https://opensource.apple.com/tarballs/)搜索下载launchd项目
> * 解压

编译（⌘+B）
1. 环境配置问题 
```
1. 在objc4-723/ 目录下创建include 文件夹， 在 objc->TARGETS->objc->Build Settings->Search Paths->Header Search Paths 中添加配置:
  $(SRCROOT)/include

2. ld: can not open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ 
MacOSX10.12.sdk/AppleInternal/OrderFiles/libobjc.order
解决方法：
将 Build Settings->Linking->Order File 下的
  $(SDKROOT)/AppleInternal/OrderFiles/libobjc.order 
改成
  $(SRCROOT)/libobjc.order 
or 
  libobjc.order

3.ld: library not found for -lCrashReporterClient
解决方法：
  Build Settings -> Linking -> Other Linker Flags里删掉"-lCrashReporterClient"
```
2. ⌘+B, 'xxx.h' file not found 和 'fileName/xxx.h' file not found;
```
case 1:  'xxx.h' file not found (通用)
解决方法：
  $ find . -name "xxx.h"
  $ find . -name "xxx.h" | xargs -I{} cp {} ../objc4-723/include/

case 2: 'fileName/xxx.h' file not found
解决方法：
  $ find . -name "xxx.h"
  $ mkdir ../objc4-723/include/fileName/
  $ find . -name "xxx.h" | xargs -I{} cp {} ../objc4-723/include/fileName/
or 如果搜索结果为多个文件
  $ find . -name "xxx.h" | grep fileName | xargs -I{} cp {} ../objc4-723/include/fileName/

case 3: 'os/lock_private.h' file not found   
解决方法：
  搜不到这个文件，所以注释掉

case 4: 'System/pthread_machdep.h' file not found 
解决方法：
  在Libc的老版本中Libc-825.40.1.tar.gz, 重复case2的操作

case 5: 在 #include_next <CrashReporterClient.h>， 'CrashReporterClient.h' file not found 
解决方法：
  在Build Settings->Preprocessor Macros 中加入：LIBC_NO_LIBCRASHREPORTERCLIENT
```

3. 没有声明的标志符 or 没有定义的type
```
1.Use of undeclared identifier 'XXX_XXX'   
  $ grep -re "def.*XXX_XXX" . 
or
  $ grep -rne "#define.*XXX_XXX" .
```
```
2.Unknown type name 'ttt_ttt'
  $ grep -re 'typedef.*ttt_ttt' .
```
```
Use of undeclared identifier '_PTHREAD_TSD_SLOT_MACH_THREAD_SELF'
1. 查找在定义
  $ grep -rne "#define.*_PTHREAD_TSD_SLOT_MACH_THREAD_SELF" .
得到结果：
  $ ./libdispatch-913.20.5/private/tsd_private.h:73:#define _PTHREAD_TSD_SLOT_MACH_THREAD_SELF __TSD_MACH_THREAD_SELF
2. 在报错的文件中引入: 
  # include <pthread/tsd_private.h>
3. 重复"file/xxx.h" file not found 问题处理操作。
```

4.Typedef redefinition with different types, typedef XXXX
```
$ cd ../objc4-723/include/
$ grep -rne "typedef.*XXXX" .
找到重复定义的，注释掉重复定义。
```

5. Unknown type name 'os_unfair_lock' && Use of undeclared identifier 'OS_UNFAIR_LOCK_INIT'
```
#include <os/lock.h>
```

6.objc-os.h文件里在void lock() 和 void unlock()的实现里有两个锁相关错误： 
```
1.OS_UNFAIR_LOCK_DATA_SYNCHRONIZATION 处的 Use of undeclared 'OS_UNFAIR_LOCK_DATA_SYNCHRONIZATION' 
2.os_unfair_lock_unlock_inline 处的 Use of undeclared 'os_unfair_lock_unlock_inline'

解决方法：
void lock()里：
  // os_unfair_lock_lock_with_options_inline (&mLock, OS_UNFAIR_LOCK_DATA_SYNCHRONIZATION);    
  os_unfair_lock_lock(&mLock);             
void unlock()里：
  // os_unfair_lock_unlock_inline(&mLock); 
  os_unfair_lock_unlock(&mLock);           
```

7.objc-lockdebug.mm文件里的两个assert函数体错误，有两个解决方法
```
1.在两函数上面加入代码
  extern "C" void os_unfair_lock_assert_owner(os_unfair_lock *);
  extern "C" void os_unfair_lock_assert_not_owner(os_unfair_lock *);
2. 直接注释掉错误
  void
  lockdebug_mutex_assert_locked(mutex_t *lock) {
      // os_unfair_lock_assert_owner((os_unfair_lock *)lock);
  }
  void
  lockdebug_mutex_assert_unlocked(mutex_t *lock) {
      // os_unfair_lock_assert_not_owner((os_unfair_lock *)lock); 
  }
```
8.  Use of undeclared identifier  DYLD_MACOSX_VERSION_10_11、DYLD_MACOSX_VERSION_10_12、DYLD_MACOSX_VERSION_10_13
```
在 dyld_priv.h 文件顶部加入一下宏：
  #define DYLD_MACOSX_VERSION_10_11 0x000A0B00 
  #define DYLD_MACOSX_VERSION_10_12 0x000A0C00 
  #define DYLD_MACOSX_VERSION_10_13 0x000A0D00
```




