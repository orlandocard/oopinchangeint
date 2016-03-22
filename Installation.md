# Introduction #

The ooPinChangeInt code is delivered in a zip file with associated examples and other code.  Here's how to install it.



# Details #
## Find the Libraries Directory (Folder) ##
Find your Arduino's libraries folder.  If you don't know where that is, look in your Arduino IDE for the library path:  open up the Preferences menu item and look at the **Sketchbook location** entry.  The libraries path will be in the `libraries` directory beneath the Sketchbook location.

## Linux or MacOS ##
On Linux or MacOS, cd to it.  The command line `unzip` tool will extract everything in the archive but some libraries are not necessary for the regular operation of ooPinChangeInt; see below in the Extras section.

For Arduino 023 and previous, unzip like so:
```
unzip /path/to/oopinchangeint-v1.02.zip ooPinChangeInt/* cppfix/* cbiface/*
```

For Arduino 1.0 and newer:
```
unzip /path/to/oopinchangeint-v1.02.zip ooPinChangeInt/* cbiface/*
```
cppfix is not necessary because Arduino 1.0 provides the new() and delete() operators like a proper C++.

The unzip utility properly creates all subdirectories in the zip archive.

## Windows ##
### Using an Unzip tool ###
On Windows, open up your unzip tool.  It may allow you to specify a location where you want to unzip your files.  Make sure it will unzip to the libraries location as discussed above..  Also make sure that you recreate full paths, if your zip tool has that option as well.

For Arduino 023 and earlier, copy the ooPinChangeInt, cbiface, and cppfix folders to your Arduino libraries folder.

For Arduino 1.0 and newer the cppfix folder is not necessary.

### Windows Can Do It! ###
Alternatively, you can simply open up the zip archive and open another window to your Arduino libraries folder.  Then drag the PinChangeInt and cbiface folders over to the libraries folder.

## Restart Arduino IDE ##
You must restart the Arduino IDE in order for your new libraries to be recognized.

## Included in the Archive ##
The following files and directories are included in the archive:
```
ooPinChangeInt/ooPinChangeInt.h
ooPinChangeInt/keywords.txt 
ooPinChangeInt/Examples/PinChangeIntSpeedTest/PinChangeIntSpeedTest-1.3.pde
ooPinChangeInt/Examples/PinChangeIntSpeedTest/PinChangeIntSpeedTest.pde
ooPinChangeInt/Examples/SoftwareInterruptsExample/SoftwareInterruptsExample.pde
ooPinChangeInt/Examples/ooPinChangeInt/ooPinChangeInt.pde
ooPinChangeInt/Examples/ooPinChangeInt/pushbuttonswitch.h
ooPinChangeInt/Examples/ooPinChangeInt/uint8ToString.h
ooPinChangeInt/Examples/ooPinChangeIntTest/ooPinChangeIntTest.pde
ooPinChangeInt/Examples/ooPinChangeIntTest/pushbuttonswitch.h
ooPinChangeInt/Examples/ooPinChangeIntTest/uint8ToString.h
cppfix/cppfix.h
MemoryFree/MemoryFree.cpp   
MemoryFree/MemoryFree.h
ByteBuffer/ByteBuffer.cpp   
ByteBuffer/ByteBuffer.h
ByteBuffer/examples/ByteBufferExample.pde
cbiface/cbiface.h
```

### Extras Included in the Zip File ###
Examples:
  * ooPinChangeInt - basic functionality sketch
  * PinChangeIntSpeedTest - Used for judging the relative speed of different interrupt scenarios.
  * SoftwareInterrupts - demonstrates software interrupts.
  * ooPinChangeIntTest - my test sketch for ensuring the proper operation of the librory.

The MemoryFree library is included for running speed and size tests.  It is necessary for the PinChangeIntSpeedTest example.

The ByteBuffer library is included for running the ooPinChangeIntTest and ooPinChangeInt sketches.  The Serial library uses interrupts which interfere with the Pin Change Interrupts (or vice-versa), so one cannot use the Serial library to print status information in an interrupt.  Thanks to Sigurður Örn for a nice library.