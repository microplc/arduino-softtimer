# 为什么使用 SoftTimer ？ #

您可以为代码添加类似多任务的功能。比如你在等待一个输入的同时闪烁LED，或者发出报警声而不至于阻塞运行。
如果你面临更复杂的问题，你可以组织你的代码在任务中。你无需关心这些任务的时间，任务管理器会为你做好这些，有助于你创建简单而易于管理的代码。

SoftTimer 是一个良好的面向对象解决方案，用起来简单而有意思。

SoftTimer 占用内存资源很少。



# 它是如何工作的？ #


SoftTimer 类是一个时间管理器，它时常扫描注册的任务看是否到运行它的时刻。如果到了 (触发时间已过) 回调函数就会触发运行。无论回调函数运行多久，下一个运行根据上一次的起始时间也被计划好了。开始的周期在任务中已经被定义了，任务周期如果有必要也可以更改。

注意 SoftTimer 不会在同一时间运行多个任务，下一个任务的运行必定是当前任务被执行完毕。当一个回调函数运行时间超过其周期，时间运行可能会失败。任务的检查和运行会按照注册的顺序进行。你可以注册周期为0的多个任务，任务会运行完一个接着运行下一个。如果你仅仅有一个周期为0的任务，它实际上就等于原来 Arduino 的 setup-loop 方法。

```
#include <SoftTimer.h>

Task t1(0, myLoop);

void setup() {
  SoftTimer.add(&t1);
  // -- 你自己的代码，只运行一次
}

void myLoop(Task* me) {
  // -- 你自己的主程序代码，重复运行
}
```

你可以注册添加或删除任务在任何时候，当然你也可以稍后调整计时。

注意 SoftTimer (从 v2.0 起) 以 ms 为时基。这也就是说，你可以注册超过一个小时为周期的任务。 


# 选项 #

在 SoftTimer.h 中提供了一些选项。你可以包含和注释掉下面的宏定义来启用和禁用这些选项。

* __ENABLE_LOOP_ITERATION__ - 通过阻止 loop() 迭代你可以获得效率提高的好处。另外有一些平台需要 loop() 。如果你面对一些奇怪的现象，你可以尝试引入 ENABLE_LOOP_ITERATION 选项， ENABLE_LOOP_ITERATION 默认是禁用的。
* __STRICT_TIMING__ - 默认情况下，下一个任务计划的开始是从上一次执行的开始，但如果其他任务不能及时完成，执行可能会发生变化。通过 STRICT_TIMING 选项可使下一次执行计划为预期时间。选项 STRICT_TIMING 默认禁用，因为它可能会导致任务中的不饱满情况。


# 平台相关注释 #

由于 SoftTimer 消灭了 "loop()" 函数，在一些平台上还有一些额外的进程需要运行在 loop 循环，有可能会带来一些问题。
详见上面 ENABLE_LOOP_ITERATION 选项的介绍。

比如 ESP8266 内部看门狗通常由系统在 loop 循环中复位。所以对于 ESP8266 你有两个办法应对，一个是引入 ENABLE_LOOP_ITERATION 选项或者手动复位看门狗甚至禁用看门狗。
比如你创建一个喂狗任务：
```c
#include <SoftTimer.h>

// -- 定义方法签名.
void feedWatchdog(Task* me);

Task watchdogFeederTask(100, feedWatchdog);

void setup() {
  SoftTimer.add(&watchdogFeederTask);
}

void feedWatchdog(Task* me) {
  ESP.wdtFeed();
}
```
