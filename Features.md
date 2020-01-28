# Detailed description of the provided features #

If you want to have detailed documentation it is always a good idea to browse the code for comments. Aspecially suggested to read the header files, where you will find all the available features well documented.

Here you will find some key features of the software explained.


## [Task](https://github.com/prampec/arduino-softtimer/blob/master/src/Task.h) ##


Task is the basic building block of SoftTimer. A task defines a timing and a job. To be more precise, you need to specify the time of calling period, and the callback function to be called.

The basic idea here, is that you will create a job in a callback method, that will be called each time the specified time was passed. Of course you always have the possibility to remove the Task from the timer manager in the job with `SoftTimer.remove(task)`, e.g. in case it is only need to be called once.

Please take a look at the [Task.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Task.h). You can see the constructor reflects the words above: you need to specify a timing period in milliseconds, and the callback function.

You can also see in the header file, that the period can be changed later with `setPeriodMs(periodInMilliseconds)`. Further more, the period is stored in microseconds basis in a public property, that you are free to adjust.

> In the task a nowMicros property also provided. The idea for this is that the timer manager already needs to check for the current microseconds, and your job might also interested in the current time. Checking the microseconds two times is a waste of cpu-cycles, so you can have it in this property for your own use.

The time of the last calling occurrence also stored in a public property. You must understand, that the timer manager will call your job, when the **lastCallTimeMicros** plus the **periodMicros** is passed (added value is less than the actual time). Advanced developers might want to tweak the system by modifying the lastCallTimeMicros of a job (e.g. reset a countdown timer).

 > When you are about to use the SoftTimer module. You will find yourself, that you will like to write proper C++ code. The way to achieve this, is to inherit from the Task class, and define your job logic in this class. Separated from other logic and reusable. The following part of this documentation contains good example for that: all of these features are inherits Task. You might want to follow them as templates.
**Note**: Arduino does not let you to pass class member methods, as function pointer. You always need to have a static function to act as a callback. This is why the Task is always passed to the callback method as parameter.


## [BlinkTask](https://github.com/prampec/arduino-softtimer/blob/master/src/BlinkTask.h) ##


Every physical computing project starts with a blinking of a led.
You often need to have an indicator that shows the state of your program.

BlinkTask works in two mode:
  * Perpetual mode - Blinks forever.
  * Count mode - Blinks for an amount of occasion.

Perpetual mode has two kinds:
  * On-Off repetition - Repeat on and off states.
  * After a count of "on" times suspend for some time.

BlinkTask can work with on level of HIGH (default) or LOW.

Use start() function to register the task in the Timer Manager, so start blinking. See [BlinkTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/BlinkTask.h) header file for details.

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


Tone player plays a melody on a specified output pin using the tone() and noTone() Arduino functions. You can specify the melody in quite tricky way, see [TonePlayer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/TonePlayer.h) header file for details.

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

Note, that you must take care to wait for the melody to finish before playing the next melody. Otherwise the next play will abort the previous melody. You may find the DelayRun task interesting.



## [SoftPwmTask](https://github.com/prampec/arduino-softtimer/blob/master/src/SoftPwmTask.h) ##

With this task you can add PWM functionality for pins that did not have hardware PWM. See [SoftPwmTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/SoftPwmTask.h) header file for details.

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

This class is to launch something after an amount of period. You can even specify a "followedBy" task, which will be run after this one has finished. See [DelayRun.h](https://github.com/prampec/arduino-softtimer/blob/master/src/DelayRun.h) header file for details.

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


The debouncer task recommends to use the PciManager to manage the pin change interrupts. However you may handle interrupts manually.
After creating the debouncer task, you do not need to register it to the Timer Manager.

If you are using the PciManager, you only have to register the debouncer to the PciManager.

If you would like to handle PCI manually, you need make the pciHandleInterrupt() function to be called on pin change.

Debouncer will call your "onPressed" callback function when the button has a sable pressed state, and the "onReleased" on the end of the press. The "onReleased" callback also receive the total time passed on the pressing state.

You can use this debouncer both on Normally Opened and on Normally Closed circuits.

See [Debouncer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Debouncer.h) header file for details.

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


Rotary is a rotary encoder driver managed by SoftTimer.

See [Rotary.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Rotary.h) header file for details.




## [Heartbeat](https://github.com/prampec/arduino-softtimer/blob/master/src/Heartbeat.h) ##


Heartbeat is a special blinker. It is intended to use a visual indicator for your project more easy.

Heartbeat creates a custom timed BlinkTask and starts is immediately.

See [Heartbeat.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Heartbeat.h) header file for details.




## [Dimmer](https://github.com/prampec/arduino-softtimer/blob/master/src/Dimmer.h) ##


With the dimmer you can easily adjust the PWM level of an output. The dimming is done in linear scale. Dimmer has some neat options, like hold/continue dimming, or revert direction any time. You can also set it up to be automatically stopped when limit (totally on/totally off) reached.

See [Dimmer.h](https://github.com/prampec/arduino-softtimer/blob/master/src/Dimmer.h) header file for details.



## [FrequencyTask](https://github.com/prampec/arduino-softtimer/blob/master/src/FrequencyTask.h) ##


Frequency task is just to play with the possibilities of the SoftTimer library. With the FrequencyTask you can generate square wave frequencies.

See [FrequencyTask.h](https://github.com/prampec/arduino-softtimer/blob/master/src/FrequencyTask.h) header file for details.
