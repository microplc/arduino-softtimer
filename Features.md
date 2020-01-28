# 特征及详情 #

浏览代码和注释是了解文档详情的好主意，特别是读头文件，会找到所有的可用特征。这里是软件的一些关键特征：


## [Task](https://github.com/prampec/arduino-softtimer/blob/master/src/Task.h) ##


任务是 SoftTimer 的最基本单元。一个任务定义一个时间和一项工作，更准确的说，你需要指定子程序运行周期时间和回调函数。

基本思想就是，你创建一个回调方法代表一个任务，这个任务被定时调用和运行。当然你总是可以删除任务通过 `SoftTimer.remove(task)` 方法, 万一他只需要运行一次呢。

请仔细浏览 [Task.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Task.h) 。你可以看到语句关键词，你需要指定以 ms 为单位的时间周期和回调函数。

你也可以在头文件中看到周期也可以在后期调整通过 `setPeriodMs(periodInMilliseconds)` 。进一步说，时间周期以 ms 为单位，你可以自由调整。

> 在任务中 nowMicros 属性被提供，它的意思是时间管理器已经需要检查当前的微秒数，你的程序一般也需要关注当前时间。检查微秒数两次是一个浪费CPU效率的事情，因此你可以利用这个属性。

回调发生的最后时间也储存在一个公共属性里，你需要理解的是，时间管理器将会调用你的任务当e **lastCallTimeMicros** 超过 **periodMicros** (添加的数值小于实际执行时间)。. 资深开发者会试图调整任务(e.g. 重置倒计时计时器)。

 > 当你使用本模块时，你会发现你喜欢使用 C++ 代码。. 实现这一点的方法是从Task类继承，并从你的任务逻辑来定义类, 下面文档包括一些好的例子所有这些特征具有继承特性，你可以遵从模板来使用。
**注意**: Arduino 不允许传递类成员方法，比如函数指针。你需要以静态函数扮演回调函数，这就是为什么总是传递回调方法作为参数的原因。


## [BlinkTask](https://github.com/prampec/arduino-softtimer/blob/master/src/BlinkTask.h) ##


所有编程教学都是以闪烁LED来开始。你经常需要通过LED来指示程序状态。

BlinkTask 工作在两种模式：
  * 不间断模式 - 永远闪烁
  * 计数模式 - 闪烁一定的数量

不间断模式有两种：
  * On-Off 重复两种状态
  *"on" 一段时间暂停一段时间

BlinkTask 任务工作级别为 HIGH (默认) or LOW.

使用 start() 函数来注册一个任务到时间管理器，启动一个闪烁详见 [BlinkTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/BlinkTask.h) 头文件

```
#include <SoftTimer.h>
#include <BlinkTask.h>

#define LED_PIN 13

// -- On for 200ms off for 100ms, repeat it 2 times, sleep for 2000 ms and than start again.
BlinkTask heartbeat(LED_PIN, 200, 100, 2, 2000);

void setup() {
  heartbeat.start();
}
```



## [TonePlayer](https://github.com/prampec/arduino-softtimer/blob/master/src/TonePlayer.h) ##


音调播放者展示了一种方法在特定的引脚上输出音乐，使用 tone() 和 noTone() 函数。 你可以指定旋律以非常聪明的方式，详见 [TonePlayer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/TonePlayer.h) 头文件

```
#include <SoftTimer.h>
#include <TonePlayer.h>

#define BEEPER_PIN  10

TonePlayer tonePlayer(BEEPER_PIN, 200); // -- Tone manager

void setup(void)
{
  tonePlayer.play("c1g1c1g1j2j2c1g1c1g1j2j2o1n1l1j1h2l2_2j1h1g1e1c2c2");
}
```

注意：你必须关注等一个曲调完成再开开始下一个曲调。否则下一个曲调会终止上一个曲调。你会听到非常有意思的音乐。



## [SoftPwmTask](https://github.com/prampec/arduino-softtimer/blob/master/src/SoftPwmTask.h) ##

通过这个任务你可以为引脚添加非硬件 PWM 功能， 详见 [SoftPwmTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/SoftPwmTask.h) 头文件

```
#include <SoftTimer.h>
#include <SoftPwmTask.h>

#define OUT_PIN  13

// -- Set up PWM to the out pin.
SoftPwmTask pwm(OUT_PIN);

void setup(void)
{
  // -- Register the task in the timer manager.
  SoftTimer.add(&pwm);
  
  // -- Writes a value of 128. That means output will "dimmed" half way. 
  pwm.analogWrite(128);
}
```


## [DelayRun](https://github.com/prampec/arduino-softtimer/blob/master/src/DelayRun.h) ##

这个类用于一段时间后触发什么事情。你可以指定一个跟随任务，当前任务完成后运行此任务，详见 [DelayRun.h](https://github.com/prampec/arduino-softtimer/blob/master/src/DelayRun.h) 头文件。

```
#include <SoftTimer.h>
#include <DelayRun.h>

#define OUT_PIN 13

// -- This task will turn off the LED after 1 second.
DelayRun offTask(1000, turnOff);
// -- This task will turn on the LED after 2 seconds.
// -- After the onTask we loop the offTask.
DelayRun onTask(2000, turnOn, &offTask);


void setup() {
  // -- We close the loop, so after offTask the onTask will start.
  offTask.followedBy = &onTask;

  pinMode(OUT_PIN, OUTPUT);
  
  // -- Turn on the LED;
  digitalWrite(OUT_PIN, HIGH);
  
  // -- Start the offTask to take effect after 1 second.
  offTask.startDelayed();
  
}

boolean turnOff(Task* task) {
  digitalWrite(OUT_PIN, LOW);
  return true; // -- Return true to enable the "followedBy" task.
}
boolean turnOn(Task* task) {
  digitalWrite(OUT_PIN, HIGH);
  return true; // -- Return true to enable the "followedBy" task.
}

```



## [Debouncer](https://github.com/prampec/arduino-softtimer/blob/master/src/Debouncer.h) ##


消抖任务推荐使用 PciManager 来管理引脚中断，然而你也可以手动处理中断，
当你创建消抖任务之后，你无须注册它到时间管理器。

如果你使用 PciManager，你仅仅需要注册消抖到 PciManager 。

如果你喜欢手动处理引脚中断，你需要使得 pciHandleInterrupt() 函数在引脚变化时被执行即可。

当按键处于按压状态时消抖将会调用你的 "onPressed" 回调函数，而结束按压时会调用 "onReleased" ，另外 "onReleased" 回调函数会收到总的被按压时间。

你可以应用消抖到常开和常闭电路都可以。详见 [Debouncer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Debouncer.h) 头文件。

```
// -- Pin change interrupt
#include <PciManager.h>
#include <SoftTimer.h>
#include <Debouncer.h>

#define INPUT_PIN 3

Debouncer debouncer(INPUT_PIN, MODE_CLOSE_ON_PUSH, onPressed, onReleased);


void setup() {
  Serial.begin(9800);
  PciManager.registerListener(INPUT_PIN, &debouncer);
  Serial.println("Ready.");
}

void onPressed() {
  Serial.println("pressed");
}
void onReleased(unsigned long pressTimespanMs) {
  Serial.print("Released after (Ms): ");
  Serial.println(pressTimespanMs);
}
```



## [Rotary](https://github.com/prampec/arduino-softtimer/blob/master/src/Rotary.h) ##


Rotary 是一个旋转编码器管理器。详见 [Rotary.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Rotary.h) 头文件。




## [Heartbeat](https://github.com/prampec/arduino-softtimer/blob/master/src/Heartbeat.h) ##


Heartbeat 是一个特殊的闪烁器。常用于方便指示你的项目状态。

Heartbeat 创建一个自定义的时间闪烁器并立即开始工作。详见 [Heartbeat.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Heartbeat.h) 头文件。




## [Dimmer](https://github.com/prampec/arduino-softtimer/blob/master/src/Dimmer.h) ##


With the dimmer you can easily adjust the PWM level of an output. The dimming is done in linear scale. Dimmer has some neat options, like hold/continue dimming, or revert direction any time. You can also set it up to be automatically stopped when limit (totally on/totally off) reached.

See [Dimmer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Dimmer.h) header file for details.



## [FrequencyTask](https://github.com/prampec/arduino-softtimer/blob/master/src/FrequencyTask.h) ##


Frequency task is just to play with the possibilities of the SoftTimer library. With the FrequencyTask you can generate square wave frequencies.

See [FrequencyTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/FrequencyTask.h) header file for details.
