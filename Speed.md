# Introduction #

I have conducted extensive speed tests of this library, since its functionality is so fundamental to Interrupts.  See the PDF file at http://code.google.com/p/arduino-pinchangeint/downloads/detail?name=PinChangeInt%20Speed%20Test-1.7.pdf.  An ODT file (OpenDocument format) file is available by request.

# Details #

The library is tested against the PinChangeInt library and also against some External Interrupt pins.  It's interesting to note that version 1.02 of the ooPinChangeInt library is only 4% slower than version 1.5 of the PinChangeInt library in those tests.

The oldest version of the PinChangeInt library that's useful, version 1.1, clocked in at 28.92 microseconds for the interrupt handler time in my tests.  The latest version of the ooPinChangeInt library clocked in at 30.82 microseconds.  Not too bad for the various enhancements and tweaks made over time, and for the different coding style of the ooPinChangeInt library (it's more C++-like and allows you to abstract your sketch even more so than the PinChangeInt library).