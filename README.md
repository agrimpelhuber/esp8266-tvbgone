# esp8266-tvbgone

This is a rough and dirty TV-B-Gone port of the original Arduino version, re-written for expressif ESP8266-based boards (wemos D1 mini works great for me.) It is meant to be flashed to an ESP8266-based board, using the Arduino Development Platform. Instructions how to use it with ESP8266 boards is found elsewhere. The IRremoteESP8266 library can also be downloaded using the library manager.

## Warnings

Please be warned: This is my first Github project, so expect lots of formal mistakes. This readme is probably the first one. English is also not my native language, so be kind.

Second warning: My programming style is based on trial & error, with a lot of copy & paste getting into the mix. Most of the time, I have no idea how things work, let alone why. If they do, I'm fine.

The intention of this Github project is to give something back to the community, since I profited so much.

The gist: I'm only partly willing to make this a "real" project, lacking the time and knowledge to do so. So don't criticise, this will only help to drive me away. If you want this project to be changed / improved in any way I can't provide, just do it - it's yours!

And: Any damage to hardware, software or body parts through this projects are simply your own fault, not mine.

## Credits

Based mainly on Ken Shirriff's (and other contributors', credits [here](https://github.com/shirriff/Arduino-TV-B-Gone)) Arduino port of the original TV-B-Gone

Based partly (important part, though) on Mark Szabo's [IRremoteESP8266](https://github.com/markszabo/IRremoteESP8266), also based on Ken Shirriff's work. Thanks, Ken!

And, of course, based on the original TV-B-Gone by Mitch Altman.

## Changes to the original code

I'm taking credit and blame for the following changes (most in the .ino file, unless stated otherwise):

* Got rid of all the PROGMEM / data_ptr stuff that an Arduino seems to need. It didn't work at all. Surprisingly, my board did not run into the RAM issues that were described everywhere. Just using normal pointers. (.ino file, WORLD_IR_CODES.h)
* Changed the decompression so that it reads the data from the const arrays, and writes the values into the rawData\[300\] array.
* Removed xmit() and all Arduino HW timer stuff, and replaced it with irsend.sendRaw(rawData, ...
  * Works with PWM-based IR codes, non-PWM ones (freq = 0) not yet tested!
  * Things can be so easy ... thanks, Mark!
* Removed the initialisation of the IR LED; this is done by irsend.begin(); automatically.
* Added a lot of yield() instructions. What I had taken for resets due to RAM problems turned out to be the ESP8266's watchdog, kicking in because it could not do any background tasks.
* Added a proper debouncing of the trigger switch, using a while loop (and a yield() within, of course).
* Removed all the sleep / wake stuff - didn't work on ESP8266, probably needs to be rewritten - or ignored.
* Remapped pins to my wemos D1 mini, see below (main.h)
* Changed LED to internal one, and swapped on / off levels
* Redefined #define freq_to_timerval(x), because we need the real frequencies (main.h)
* Added a 140th IR code for EU: My Samsung TV reacted on EU code 006 as power toggle, getting switched on!
  * Because I didn't want to change the original codes, I added EU code 140, which is a discrete NECv2 / Samsung power OFF.  (WORLD_IR_CODES.h)
  * It is sent around 25th position, because I wanted to counter code 006 very soon. Sorry for messing up the order!
* I didn't change (= was to lazy) to change the delay_ten_us() function to the ESP8266's own built-in microsecond delay function. It would have probably saved me a lot of trouble with the watchdog, see above.

Finally: I have also written a version that can be flashed over the air (OTA), but I want to first see how this goes before committing that version. It needs some more explanation.

## Hardware

I use a wemos D1 mini ESP8266-based board, because it is compact, yet has a built-in USB port. Thanks to that, I can power the IR remote through a USB power bank, which I carry around all the time anyway. Taken into account that I don't use the TV-B-Gone constantly, a dedicated power supply would be overkill. Nice side effect: The USB power also gives me 5V to power 3 IR LEDs.

The downside: On the D1, a lot of pins interfere with the USB / serial communication, internal LED, reset switch, flash interface and other stuff. I have finally found a pin usage where everything works fine. Here is my rough layout:

* D1 (Pin 5) => GND
  * Region selection (closed = EU): Stayed the same from the original code. Might need a switch and / or resistor, but the direct wiring works for me.
* D6 (Pin 12) ==> Switch ==> GND
  * Switch closes to trigger
* D5 (Pin 14) ==> resistor 2.2K ==> base of BC 337 transistor
  * Don't know whether it's the proper resistor or transistor, I just had them laying around. Works well enough for me.
* USB VCC 5V ==> resistor 10 Ohm ==> Anode (long pin) of first of 3 IR LEDs in a row
  * IR LED specs: 1,7V, 50mA. I hope my calculations work.
* Kathode of third IR LED ==> collector of BC 337 transistor
* Emitter of BC 337 ==> GND

The circuit is soldered onto a piggyback PCB on top of the wemos D1. And that package is inside a 44mm x 3mm x 25mm 3D printed housing.
