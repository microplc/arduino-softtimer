# 快速上手导读 #

##### 1. 在你的程序中引入 SoftTimer 。
```
#include <SoftTimer.h>
```

##### 2. 你需要创建一个带时间（ms）参数的任务 Task 及其召唤函数。例子中 callBack1 将每经过 2000 被召唤执行一次。
```
Task t1(2000, callBack1);

void callBack1(Task* me) {
  // 你自己的代码
}
```

##### 3. 注册你的任务 task 到 SoftTimer (时间管理器)
```
void setup() {
  SoftTimer.add(&t1);
}
```

##### 4. 你成功了。你可以添加更多的任务进去或使用捆绑任务声明

完整代码：
```
#include <SoftTimer.h>

// -- taskOn 将会每2s触发一次
Task taskOn(2000, turnOn);
// -- taskOff 将会每 1111ms 触发一次
Task taskOff(1111, turnOff);

void setup() {
  // -- Mark pin 13 as an output pin.
  pinMode(13, OUTPUT);

  // -- 注册任务到时间管理器，所有的任务都会立即启动
  SoftTimer.add(&taskOn);
  SoftTimer.add(&taskOff);
}

/**
 * Turn ON Arduino's LED on pin 13.
 */
void turnOn(Task* me) {
  digitalWrite(13, HIGH);
}

/**
 * Turn OFF Arduino's LED on pin 13.
 */
void turnOff(Task* me) {
  digitalWrite(13, LOW);
}
```

注意 "loop()" 方法已经被替代了。
