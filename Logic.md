The fundamentals of this library are very close to the PinChangeInt library.  See http://code.google.com/p/arduino-pinchangeint/wiki/Logic for the details of the Pin Change Interrupt system in the Arduino, and how the PinChangeInt library works.

The key difference between this library and the PinChangeInt library is in the `attachInterrupt()` method.  In PinChangeInt, the method is called like this:
```
void PCintPort::attachInterrupt(uint8_t arduinoPin, PCIntvoidFuncPtr userFunc, int mode)
```

Here in the ooPinChangeInt method, it's called like this:
```
void PCintPort::attachInterrupt(uint8_t arduinoPin, CallBackInterface* cbIface, int mode)
```
And we attach the cbiface to a pin like this, in the attachInterrupt() method:
```
    // Now we attach the object which has subclassed the CallBackInterface
    p->pinCallBack=cbIface;
```

In the PinChangeInt version, the userFunc is a simple pointer to a function that returns void, like an old-fashioned C function call.

In ooPinChangeInt, cbIface is a pointer to an object of type CallBackInterface.  This means that we have handed a full-fledged object to the attachInterrupt method.  And once we have a pointer to an object, we can call any public method in that object.  That's exactly what happens in our interrupt handler; here is the code from the PCint() interrupt handler method:
```
        (*(p->pinCallBack)).cbmethod();
```

This is why we need to subclass the CallBackInterface class:  Because we need to choose a common method so that the compiler can construct our code for us properly.  Since I don't know what your code is and what it's used for, I provide this contract:
> I will call your method and do whatever you ask, Dear Programmer, so long as you name your method `cbmethod()`, return void, and have no arguments.  I trust you don't find that so restrictive since it is, after all, a member of an object and therefore it has access to all other members of that object.

# So... Why??? #
The PinChangeInt library has been happily Pin Change interrupting for months now, so why the change?  What's the big deal?

As it turns out, I was using the PinChangeInt library for a while and learning how it works, but something bothered me.  I was creating a lot of arrays and such, and spending time correlating the Arduino's design to my switch layout.  The PinChangeInt library created objects- the Ports- like a good C++ program, but it was doing everything else in a very C-like fashion.  Frankly, it bugged me.

So I endeavored to create a library in a completely C++ style to see how it worked.  How much speed would I lose?  How more bloated is the C++ code?  And, how would it change my coding style?  The answers are:  Not much, not much more, and quite nicely thank you.

Here I will compare the techniques I used for two libraries.  The first, AdaEncoder, uses the PinChangeInt library.  It reads and reports on the rotation of rotary encoders, which are essentially two switches that work in tandem with one another.

The second is called Tigger.  This uses the ooPinChangeInt library.  Its job is to keep track of the activity of a bouncing switch, and keep track of the state of the switch via its internal variables.

To my mind the techniques I used are representative of the typical approach each style will necessitate.
## PinChangeInt Style ##
With PinChangeInt, the idea is to set up a data structure that represents a pin, to populate it on a pin-by-pin basis and, when triggered, to gather information from that pin and correlate it to your data structures in software:
  * Setup: define a data structure; populate it according to the necessary pins.
```
struct encoder {
    volatile uint8_t* port;

    uint8_t bitA;
    uint8_t bitB;

    uint8_t turning;    // Flag to keep track of turning state
    int8_t clicks;      // Counter to indicate cumulative clicks in either direction
    int8_t direction;   // indicator

    char id;

    encoder *next;
};
static encoder* AdaEncoders[20]; // maximum of 20 possible pins on ATmega328, 0-19
...
    encoder *newencoder=(encoder*) malloc(sizeof(struct encoder));
    newencoder->turning=0;
    newencoder->clicks=0;
    newencoder->id=id;
    newencoder->next=NULL;
...
    AdaEncoders[pinA]=newencoder;
    AdaEncoders[pinB]=newencoder;
```
Now when triggered, gather information from that pin and correlate it to your data structures in software:
```
    encoder *tmpencoder=AdaEncoders[PCintPort::arduinoPin];
    portState=*tmpencoder->port;
```

## ooPinChangeInt Style ##
In the Object Oriented style, you model a device in software.  Here is how I designed my Tigger library, which is used for switches:
```
class Tigger : public CallBackInterface {
    public:
        uint16_t getCount(); // read the number of state changes from the switch.  Reset the count to 0 upon read.
        void cbmethod();  // required method for a subclass of CallBackInterface
    protected:
        uint8_t _arduinoPin;
        uint8_t _delay;
        uint16_t count;
        uint8_t _resistor;
        uint8_t _trigger;
        long starttime;
}
```
Now when the interrupt triggers it calls the model's (object's) cbmethod():
```
void Tigger::cbmethod() {
    if (_delay != 0) {
        uint8_t oldSREG = SREG; 
        cli();
        tigermillis = timer0_millis;
        SREG = oldSREG;
        if (tigermillis-startTime <= _delay) return; 
        startTime=tigermillis;
    }
    count++;
}
```
At any time in your code, you can query the object using a method such as `getCount()`.   In general your main code and your interrupt are decoupled so doing this should come fairly naturally.