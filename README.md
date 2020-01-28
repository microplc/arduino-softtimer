# Arduino SoftTimer library [![Build Status](https://travis-ci.org/prampec/arduino-softtimer.svg?branch=master)](https://travis-ci.org/prampec/arduino-softtimer) #

## 描述 ##

使用 SoftTimer 会让你达到一个较高的编程水平，但它是易用且轻量级的。

你经常会遇到需要多任务交叉工作的问题，使用 SoftTimer 程序员可以在任务中组织繁忙的逻辑，SoftTimer 的时间管理器会照顾好所有人物的调度运行。 

当你使用 SoftTimer 无需再声明Arduino的 "loop" 函数，你所有的代码将变为 *event driven*，所有的过程异步进行，阻塞类代码 (比如 delay()) 可以不再使用。

我还试图增加了一些实用的工具到 SoftTimer 中 (比如 blinker, pwm, debouncer, rotary 等)

You can install SoftTimer library directly from the Library Manager of the Arduino IDE. Follow this link for details: [How to Install a Library](https://www.arduino.cc/en/Guide/Libraries#toc3)

编译 SoftTimer 库需要一个外部依赖库 PciManager library 同样可以用库管理器安装。

文档详见：

# [Link to Documentation](https://github.com/microplc/arduino-softtimer/blob/master/SoftTimer.md) #
