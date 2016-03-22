# Introduction #

How to use the ooPinChangeInt library in your own programs.  Here I assume you have properly installed the library.  See the Installation wiki page for more information.

This library has been developed and is supported on the Arduino Uno/Duemilanove only but should work on any ATmega328-based Arduino.

This library will likely not work with the Arduino Mega. It appears that the only port that shares PinChangeInterrupts? between the ATmega328 and ATmega2560 is Port B.



# Overview #
## The Quick and Dirty Method ##
### #include ###
In your sketch, at the top of the file, put:
```
#include <ooPinChangeInt.h>
```
### Create a Class ###
Create a C++ class that the library will call whenever your pin(s) are interrupted:
  * In the Arduino IDE, find the little menu in the down arrow at the upper right of your sketch window.
  * Open a new tab, and create another file.  Say we want to create a class called "`pushbuttonswitch`".  So we create a file called `pushbuttonswitch.h`.  Technically, there should not be a need to create the class in a new file but because of some vagaries in the way the Arduino IDE arranges your sketch's code, if you create classes in the main sketch file there may be circumstances where your sketch will not work.

There are 3 requirements for your class and file:
  * The class must subclass the `CallBackInterface` class, which comes with the ooPinChangeInt distribution.  This is easy to do, see the example below.
  * The class must include a method called `void cbmethod()`, which is the method (aka, "function") that will be called when an interrupt occurs.
  * The file must also include the basic Arduino include files, if you want to use Arduino things- pin names and such.
In the below example you can find an example of a `pushbuttonswitch` class.

### Attach the Interrupt ###
In the class, we attach to an interrupt using `PCintPort::attachInterrupt()`.  This is just an example; you don't _have_ to do it right in your class.  Instead, you can do it in your sketch, but this is the more C++-like way to do it (that is, "encapsulation within objects," if you want the technical term for it).  To attach an interrupt, you must decide on the mode- that is, if you want to interrupt on RISING, FALLING, or CHANGE of signal.  Then you call the `PCintPort::attachInterrupt()` in the manner shown below.

If you want to be fancy, you won't hardcode the mode in the object, you'll do it in the object's constructor or you'll call a method which sets the code.

This, by the way, is this object's constructor:
```
   pushbuttonswitch (uint8_t _pin, char *_name): pin(_pin), name(_name) {
     init();
   };
```
And here is the full object:
```
// This example works in both Arduino 023 and earlier, and Arduino version 1.0.
#if defined(ARDUINO) && ARDUINO >= 100
  #include <Arduino.h>
#else
  #include <pins_arduino.h>
  #include <WProgram.h>
#endif

class pushbuttonswitch : public CallBackInterface
{
 public:
   uint8_t count;
   uint8_t pin;
   char *name;

   pushbuttonswitch (uint8_t _pin, char *_name): pin(_pin), name(_name) {
     init();
   };

   void cbmethod() {
     Serial.print("Int. pin "); Serial.print(pin, DEC);
     Serial.print(" "); Serial.println(name);
   };
   
   uint8_t getCount() {
     return count;
   }
   
   char *getName() {
     return name;
   }
   
   uint8_t reset() {
     count=0;
   }

 private:
  void init () {
    pinMode(pin, INPUT);
    digitalWrite(pin, HIGH);
    PCintPort::attachInterrupt(pin, this, FALLING);
  };
};
```
### Optimize ###
You are able to apply some optimization switches right in your sketch.  Read over these #defines and decide if any apply to you.  You can add them to your code **ahead** of the `#include ooPinChangeInt.h`.  Here is the section of code from the .h file that lists all the available `#define`s:
```
// You can reduce the memory footprint of this handler by declaring that there will be no pin change interrupts
// on any one or two of the three ports.  If only a single port remains, the handler will be declared inline
// reducing the size and latency of the handler.
// #define NO_PORTB_PINCHANGES // to indicate that port b will not be used for pin change interrupts
// #define NO_PORTC_PINCHANGES // to indicate that port c will not be used for pin change interrupts
// #define NO_PORTD_PINCHANGES // to indicate that port d will not be used for pin change interrupts
// if there is only one PCInt vector in use the code can be inlined
// reducing latency and code size
// define DISABLE_PCINT_MULTI_SERVICE below to limit the handler to servicing a single interrupt per invocation.
// #define       DISABLE_PCINT_MULTI_SERVICE
```
How to decide if any of those apply to you?  Read over the [Logic](http://code.google.com/p/arduino-pinchangeint/wiki/Logic) page, the **ATmega328p Pins and Ports** section.  Now review your desired pin connections, and correlate your pins with the ATmega328p ports.   If you are not using a particular port, `#define NO_PORTx_PINCHANGES` for that port "x".  For example,
```
#define NO_PORTC_PINCHANGES
```
Remember: Once you define that a port has no pin changes on it, make sure you do not attach any interrupts to pins on that port! There is precious little error checking in this code.

The `DISABLE_PCINT_MULTI_SERVICE` define is described below.
# Reference #
## Public Methods ##
```
    static void attachInterrupt(uint8_t pin, CallBackInterface* cbIface, int mode);
```
Attaches an interrupt to the given pin.  The interrupt will call the function `userFunc`, based on the `mode`.  The arguments are:
  * `pin` - The pin on the Arduino that you want to attach an interrupt to.
  * `cbiface` - A reference or pointer to an object that subclasses the CallBackInterface class.  That class is provided in the cb library, which is distributed with ooPinChangeInt.  See the examples for how to subclass a C++ class.  I won't claim that there's no hocus pocus behind it, because there is, but you can ignore all that and just subclass by following the example- it's easy to do.  Note that subclassing the CallBackInterface class carries with it a certain responsibility:  That of creating a method `void cbmethod()` within your class.  This method will be called upon interrupt of the given pin with the given mode.
  * `mode` - The interrupt will trigger based on `mode`, which can be one of **RISING**, **FALLING**, or **CHANGE**
```
   static void detachInterrupt(uint8_t pin);
```
Stops interrupting on the given pin.
## ooPinChangeInt Optimizations ##
There are a small number of parameters that the programmer can modify to limit the amount of memory the library uses.  You can add these `#define`s to your sketch.  They must be added to the sketch ahead of the `#include <ooPinChangeInt.h>`.

### Optimizations:Memory Utilization ###
There are 3 #define's that control how many interrupt vectors are enabled.
```
//#define       NO_PORTB_PINCHANGES
//#define       NO_PORTC_PINCHANGES
//#define       NO_PORTD_PINCHANGES
```
The corresponding library source code looks like this:
```
#ifndef NO_PORTB_PINCHANGES
ISR(PCINT0_vect) {
    PCintPort::pcIntPorts[0].PCint();
}
#endif
```
So if you know, for example, that you have only pins on PORTB, you can insert the lines at the top of your sketch as follows:
```
#define       NO_PORTC_PINCHANGES
#define       NO_PORTD_PINCHANGES
```
This will not set up the corresponding interrupt vectors on PORTC and PORTD and it will inline the single remaining interrupt routine.  I have measured the effect on interrupt routine speed; see the [Speed wiki page an the arduino-pinchangeint project](http://code.google.com/p/arduino-pinchangeint/wiki/Speed).

### Optimization:DISABLE\_PCINT\_MULTI\_SERVICE ###
For servicing multiple interrupts.

Normally, the ooPinChangeInt interrupt routine will attempt to handle all changed pins on a port.  This can alleviate the overhead to enter and exit the interrupt routine.  However, if you don't want this- for example, with rapid interrupts your CPU may never exit the interrupt routine- then you can define this macro.  Again, you add this macro to your sketch, ahead of `#include ooPinChangeInt.h`.
```
#define       DISABLE_PCINT_MULTI_SERVICE
```
This will likely make the interrupt code slightly faster, as well.  I haven't measured its effect, but I imagine it to be less than 10% of the total interrupt code time.

# More Information #
See the [Logic](http://code.google.com/p/arduino-pinchangeint/wiki/Logic) wiki page in this project for more detailed information about the operation of the library.
# Software Interrupts #
It is possible to interrupt an Arduino pin from your own sketch.  This would cause an interrupt just like connecting an external switch to your Arduino, but you can do it right from your own sketch.  I use this to test the speed of the interrupt code, by having the interrupt triggered by a change on the port from the sketch.  You may come up with another use for such a thing.

Remember that you must provision the pin as an **output** pin, so you should not have any switches connected to it.  You can connect an LED or transistor or gate or what have you (ie, devices that are driven by the pin's output) to the pin.

Note that the Arduino's `digitalWrite()` library function carries some overhead.  Instead of using it, you may want to modify a port by writing directly to a register.  Here's a quick example of creating a software interrupt by writing to a register.  Also, see the `PinChangeIntSpeedTest.pde` sketch that comes in the Examples folder.  This example was distilled from that sketch.

```
#define       NO_PORTB_PINCHANGES
#define       NO_PORTC_PINCHANGES

#include <ooPinChangeInt.h>
#include <cbiface.h>

// Defining your class in an Arduino sketch is not fully supported.  In this sketch,
// it works.  But it may trip you up.  For example, using an object as an argument
// in a C-style function will probably fail.
class speedy : public CallBackInterface
{
 public:
   uint8_t id;
   long count;
   speedy (): id(0) { init(); };
   speedy (uint8_t _i): id(_i) { init(); };

   void init() {
     count=0;
   }

   // CallBackInterface requires you to define this method.  It's what
   // will be called when the pin is interrupted.
   // Here, we simply set var0 to the low byte of the Arduino's clock.
   // In the real world, you'd probably do something important.
   void cbmethod() {
     count++;
   }

   uint8_t getID() {
     return (id);
   }

   long getCount() {
     return (count);
   }

   void reset() {
     count=0;
   }
};

volatile uint8_t *pinT_OP, *led_port;
volatile uint8_t *pinT_IP;
uint8_t pinT_M, not_pinT_M, led_mask, not_led_mask;

#define PINLED 13
// IMPORTANT:  See the #define NO_PORTx_PINCHANGES, above!
// With those define'd, this only works for PORTD pins.
// This pin will be configured as on output.  Don't connect an input device
// (switch or what have you) to this port, or you could fry the Arduino!
#define PINTEST 2

int i=0;
speedy speedster=speedy(PINTEST);

void setup()
{
  Serial.begin(115200); Serial.println("---------------------------------------");
  pinMode(PINLED, OUTPUT); digitalWrite(PINLED, LOW);
  pinMode(PINTEST, OUTPUT); digitalWrite(PINTEST, HIGH);
  // *****************************************************************************
  // *** set up LED
  // *****************************************************************************
  led_port=portOutputRegister(digitalPinToPort(PINLED));
  led_mask=digitalPinToBitMask(PINLED);
  not_led_mask=led_mask^0xFF;
  // *****************************************************************************
  // *** set up test pin
  // *****************************************************************************
  pinT_OP=portOutputRegister(digitalPinToPort(PINTEST)); // output port
  pinT_IP=portInputRegister(digitalPinToPort(PINTEST));  // input port
  pinT_M=digitalPinToBitMask(PINTEST);                   // mask
  not_pinT_M=pinT_M^0xFF;                       // not-mask
  *pinT_OP|=pinT_M;
  // *****************************************************************************
  PCintPort::attachInterrupt(PINTEST, &speedster, CHANGE);
} // end setup()

unsigned long milliStart, milliEnd, elapsed;
void loop() {
  uint8_t k=0;
  i=0;
  *pinT_OP|=pinT_M;        // pintest to 1
  Serial.print("Testing pin: "); Serial.print(PINTEST, DEC); Serial.print(" ");
  Serial.print(" interrupt's id: ");
  Serial.println(speedster.getID(), DEC);
  delay(1000);
  Serial.print("Start..");
  delay(1000); Serial.println("*");
  *led_port|=led_mask;
  milliStart=millis();
  while (i < 1000) {
    k=0;
    while (k < 100) {
      *pinT_OP&=not_pinT_M;    // pintest to 0
      *pinT_OP|=pinT_M;        // pintest to 1
      k++;
    }
    i++;
  }
  milliEnd=millis();
  *led_port&=not_led_mask;
  elapsed=milliEnd-milliStart;
  Serial.print("Elapsed: "); 
  Serial.println(elapsed, DEC);
  Serial.print("Interrupts: ");
  Serial.println(speedster.getCount(), DEC);
  speedster.reset();
  delay(500);
}
```
# Example #
**Note: This example as shown is defective, in that it uses Serial.print() statements in `cbmethod()`.  Serial.print() uses an interrupt which will conflict with the PinChangeInterrupt that calls cbmethod(), and make the Arduino hang... eventually.  It will work for a little while, most likely.  But your best bet is to actually download the code and look at the ooPinChangeInt example there, which uses a ring buffer instead and (I promise) won't hang.**

Here is an example with three pushbuttons: connected to pins 2, 3, and 4.  One side of the pushbutton is connected to ground.  The other side connects to the Arduino pin.  For a more elaborate example, see the ooPinChangeIntTest sketch that comes shipped with the library.

This example comes in two files:  The sketch and the header (.h) file which contains the class pushbuttonswitch.  It can be found in the Examples directory of the ooPinChangeInt distribution.

As you look at this example, I would like to point out a few things:

Since the nitty-gritty details of the switches are stored in the class- that is, they are "encapsulated" in the class- the code is more organized and easier to maintain than the corresponding C code.  For example, each switch you connect to the Arduino needs a little housekeeping:  You have to set its mode to `INPUT` or `OUTPUT` and you have to turn on the pullup resistor.  Additionally for this purpose you have to attach an interrupt to it.  In C you would either:
  * Iterate over a series of numbers in a for loop.  This assumes that the pins are all in sequential order.
  * Populate an array with each pin number, then loop over that array so as to utilize the common code to set the values for each pin, or
  * Use explicit statements to set each pin.

In the first method, you have to ensure that your pins are created sequentially.  In the next method, you have to create an array to manage your pins.  In the last method, you may forget to properly set all the pins.  These are small niggling details that at first blush come to but little grief- why, you've been doing it this way for years, after all.  However, they seem clumsy next to the C++ method:
```
  void init () {
    pinMode(pin, INPUT);
    digitalWrite(pin, HIGH);
    PCintPort::attachInterrupt(pin, this, FALLING);
  };
```
...Each and every pin will be initialized in the same way.  What if you make a mistake in this code?  It will manifest the first time you use it and once you fix it, it's fixed for every switch you create.  There are no if statements, loops, or arrays.  Every switch you create is encapsulated in an object, and each object is created in the same way.  If you want to modify your switch code, you do it once in the object and all switches are updated.

But wait!  That's not all!  When you create a switch:
```
pushbuttonswitch redswitch=pushbuttonswitch(2, "red");
```
you now no longer refer to the switch by its pin number (i.e., "The switch attached to pin 2"), but by its name:  "The red switch".  You are no longer thinking about it in terms of where you plugged it into the microcontroller- after all, was that really what you set out to do in the beginning?  "I desire to connect a switch to pin 2 of a microcontroller!"

No.  You wanted to make a switch turn on a red LED, and you needed the versatility that a microcontroller offers.  This is called "thinking in the problem domain," and it is what C++ was designed for.

For this elegance you pay, twice:  First, in adjusting your mindset from procedures to objects, and second with a bit of a speed and code size penalty.  Whether you can afford either is up to you, but if you can, I'd say the results are well worth it.

So without further ado, here is an example sketch and the accompanying `.h` file.  This sketch will print out a short message upon each and every interrupt.  Meanwhile, the loop is running and it will print out all the transitions that have taken place over the 500 milliseconds (more or less) for each iteration of the loop.
```
// ooPinChangeInt.pde
// ooPinChangeInt Example by GreyGnome aka Mike Schwager.  Version numbers here refer to this sketch.
// Version 1.0 - initial version
// You can reduce the memory footprint of ooPinChangeInt by declaring that there will be no pin change interrupts
// on any one or two of the three ports.  If only a single port remains, the handler will be declared inline
// reducing the size and latency of the handler.
// #define NO_PORTB_PINCHANGES // to indicate that port b will not be used for pin change interrupts
// #define NO_PORTC_PINCHANGES // to indicate that port c will not be used for pin change interrupts
// #define NO_PORTD_PINCHANGES // to indicate that port d will not be used for pin change interrupts
// if there is only one PCInt vector in use the code can be inlined
// reducing latency and code size
// define DISABLE_PCINT_MULTI_SERVICE below to limit the handler to servicing a single interrupt per invocation.
// #define       DISABLE_PCINT_MULTI_SERVICE

#include <ooPinChangeInt.h>
// ooPinChangeInt ships with this little library.
#include <cbiface.h>

// To use the library, define a class that subclasses CallBackInterface.
// And also, include a method (C++ talk for "subroutine") called "cbmethod()" in the class.
// Use this class as a template to create your own; it's not hard.  You don't
// even have to understand what you're doing at first.

#include "pushbuttonswitch.h"

pushbuttonswitch redswitch=pushbuttonswitch(2, "red");
pushbuttonswitch greenswitch=pushbuttonswitch(3, "green");
pushbuttonswitch blueswitch=pushbuttonswitch(4, "blue");

void setup()
{
  Serial.begin(115200); Serial.println("---------------------------------------");
  redswitch.getCount();
}

void loop() {
  reportit(redswitch);
  reportit(greenswitch);
  reportit(blueswitch);
  delay(500);
}

void reportit(pushbuttonswitch& aswitch) {
  uint8_t tmp;
  if ((tmp=aswitch.getCount()) > 0) {
    Serial.print(aswitch.getName());
    Serial.print(" accumulated: "); Serial.print(tmp, DEC); Serial.println(" transitions.");
  }
}
```
Here is the .h file, `pushbuttonswitch.h`.  It contains the class pushbuttonswitch.
```
#if defined(ARDUINO) && ARDUINO >= 100
  #include <Arduino.h>
#else
  #include <pins_arduino.h>
  #include <WProgram.h>
#endif

// How do you subclass?  Like this; it's simply the " : public CallBackInterface"
class pushbuttonswitch : public CallBackInterface
{
 public:
   uint8_t count;
   uint8_t pin;
   char *name;

   pushbuttonswitch (uint8_t _pin, char *_name): pin(_pin), name(_name) {
     init();
   };

   void cbmethod() {
     Serial.print("Int: "); Serial.print(pin, DEC);
     Serial.print(" "); Serial.println(name);
   };
   
   uint8_t getCount() {
     return count;
   }
   
   char *getName() {
     return name;
   }
   
   uint8_t reset() {
     count=0;
   }

 private:
  void init () {
    pinMode(pin, INPUT);
    digitalWrite(pin, HIGH);
    PCintPort::attachInterrupt(pin, this, FALLING);
  };
};
```