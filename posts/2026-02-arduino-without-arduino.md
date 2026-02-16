<!--
.. title: Arduino without Arduino
.. slug: arduino-without-arduino
.. date: 2001-01-01 12:00:00 UTC
.. tags: diy, arduino, electronics, til
.. category: dev
.. link:
.. description:
.. type: text
.. status: private
-->

I always had difficulty understanding what Arduino is in essence. My current understanding is that Arduino stands for a whole set of related tools designed to be used together, in order to make it easier to get into microcontroller programming. The individual bits and pieces involved are:

- Arduino development boards, such as the Uno or Leonardo, typically running AVR chips such as ATmega328P or ATmega32U4
- Arduino bootloader, a program that runs on these boards and allows reprogramming them over USB
- Arduino IDE, an app that lets you write C code and upload it to the board; it comes bundled with the compilation toolchain (avr-gcc) and an upload tool (avrdude)
- Arduino standard library, a set of C modules which makes it easier to do common tasks e.g. `digitalWrite()` or `Serial.write()`
- A while ecosystem of libraries written against the Arduino standard library, eg. [ws12812B-arduino][ws12812B-arduino]

I always wanted to learn **what does Arduino IDE do under the hood?** How do I program a board myself by using the underlying tools directly? Read on to see what I found out.

<!--more-->

## Getting started

## Title

Here's what happens when you click "upload" in the Arduino IDE:

- Arduino preprocesses your sketch (`.ino` file) into a complete C program
- the compiler `avr-gcc` compiles your project (the "sketch") is compiled into a binary (bytes to be uploaded to the board)
- the IDE talks to the board through the serial port and tells it to restart
- the board restarts, the bootloader runs, opens a new serial port through USB
- the utility program `avrdude` sends the binary to the bootloader through the serial port
- the bootloader flashes the internal memory, then restarts the board
- the board reboots, the bootloader starts again, waits a little, then executes your program

Now let's retrace these steps manually without Arduino IDE.

## Dependencies

We just need two things:
- the compiler `avr-gcc`
- the uploader `avrdude` 

My home PC (still) runs Windows, but I do most of my development on a Fedora VM via WSL these days, so that's what I tried to use first. I was worried that getting serial communication on WSL would be problematic (not clue if/how forwarding the USB device into a VM can work), but good surprise: turns out I don't have to!

Instead, I ended up:

- installing `avrdude` on Windows - easy! I only needed to download the release from Github and unpack the two file (`avrdude.exe` and `avrdude.conf`) anywhere into my PATH
- installing `avr-gcc` on Linux - also easy; in my case `sudo dnf install avr-gcc avr-libc`
- running `avrdude.exe` from within WSL (since you can generally use Windows binaries from within the WSL VM)

Quick check to ensure we're good to go:

a) From Windows PowerShell

```powershell
avrdude --version
```

b) From Fedora

```shell
alias avrdude=/mnt/c/Users/Kos/.bin/avrdude.exe
avrdude --version
avr-gcc --version
avrdude

...
```

## Identifying the board parameters

There's a few things we have to know about the board beforehand:

1) **Which microcontroller is this?** I'm working with a Sparkfun Pro Micro, which sports the ATmega32U4. For avrdude this translates to the "part" parameter, this one is called `m32u4`.

2) **What is the clock frequency?** My Pro Micro runs the 32u4 in 8MHz on 3.3V, while an Arduino Leonardo (which uses the same IC) would run it on 16MHz on 5V. This is still the same microcontroller but it has to be configured properly using "fuses" - a few bytes of non-volatile memory documented in the microcontroller's [datasheet][datasheet]. I haven't fiddled with these just yet. The clock frequency depends on the fuse configuration and the board construction; in my case the fuse value tells the microcontroller to use an external crystal as the clock, and the crystal on the board is a 8 MHz crystal, so I should tell `avr-gcc` to define the constant `F_CPU=8000000`.

3) **Which programmer protocol to use?** Arduino uses the [Caterina][Caterina] bootloader; my Sparkfun board comes with a tweaked version of it (more on that later). Either way, when the board powers up, Caterina opens up a serial port over USB and listens for a while for a connection; the protocol used in this case is called [`avr109`][avr109] and `avrdude` can speak it.

3) **Which baud rate to use?** I don't quite get serial ports just yet. I understand there's a distinction between "real" serial port communication over UART and "virtual" serial ports over USB. The situation will be different on ATmega32U4 (which only uses a virtual serial port over USB) and ATmega328P (which is paired with a separate USB chip and internally talks to that chip using "actual" UART). I'm hearing 115200 is a typical value but I don't have a reference.


## Let's upload something

Here's an example C code for the Pro Micro that blinks the built-in diode. Note that unlike the Ardunio IDE sketches (".ino") the entry point is `main()`, and there's no `loop()` and `setup()` functions. There's also no `Arduino.h` here so we can't use `digitalWrite` and such.

```c
#include <avr/io.h>
#include <util/delay.h>

int main(void)
{
    // PB0 output (LED)
    DDRB |= (1 << PB0);

    while (1)
    {
        PORTB ^= (1 << PB0); // toggle LED
        _delay_ms(500);
    }
}
```

First I compiled it via `avr-gcc` and obtained an `.elf` file:

```shell
avr-gcc -m mcu=atmega32u4 -D F_CPU=8000000UL -Os -o main.elf main.c
```

The build artifact is an `.elf` file (standard binary format for executables). The second step is to create the `hex` file:

```shell
avr-objcopy -O ihex -R .eeprom main.elf main.hex
```

The way I'm reading it, this file reads the `main.elf` binary and saves it in the [Intel Hex][ihex] format which is what `avrdude` wants. If there's a section `.eeprom`, then it's skipped here since it's meant to go to another memory type (we're going to write to the flash memory).

The whole program binary looks like:

```
:100000000C9456000C9460000C9460000C946000FA
:100010000C9460000C9460000C9460000C946000E0
:100020000C9460000C9460000C9460000C946000D0
:100030000C9460000C9460000C9460000C946000C0
:100040000C9460000C9460000C9460000C946000B0
:100050000C9460000C9460000C9460000C946000A0
:100060000C9460000C9460000C9460000C94600090
:100070000C9460000C9460000C9460000C94600080
:100080000C9460000C9460000C9460000C94600070
:100090000C9460000C9460000C9460000C94600060
:1000A0000C9460000C9460000C94600011241FBE3E
:1000B000CFEFDAE0DEBFCDBF0E9462000C9471008A
:1000C0000C940000209A91E085B1892785B92FEF23
:1000D00039E688E1215030408040E1F700C000005F
:0600E000F3CFF894FFCFFE
:00000001FF
```

This is the representation that `avrdude` will send over the serial port. Once we have the file, let's upload it:

```shell
avrdude -v -p atmega32u4 -c avr109 -P COM6 -b 115200 -D -U flash:w:main.hex:i
```

COM6 in the above command is the Windows identifier of the serial port to be used - remember, I'm running the command from Linux/WSL but the `avrdude` commands points to `avrdude.exe` that runs on Windows directly, so it expects Windows-themed serial port names. If you're following along on "real" linux, you'll need to use something like `/dev/ttyUSB0`.

But hold on, how do we get the serial port in the first place? This seems to depend on the bootloader...

- When I connect my Arduino Leonardo to the PC, it immediately opens up a Serial port (COM7). I believe it waits a short while for an attempt at flashing the board, and when it times out it runs the app. The port remains open indefinitely.
- When I connect my Pro Micro, it doesn't open any Serial Port at all, so I can't reprogram it yet. I have to reset it manually (by shorting GND and RST pins). The manual reset tells the bootloader to open up a serial port. The port stays visible for 6 seconds - this is the window for me to upload the new code. This 6-second wait doesn't appear when the board initially powers on.

We're looking at another Arduino IDE-specific feature here: the "Upload" button in the IDE uses the omnipresent serial port to (somehow) send a "reboot" command to the microcontroller; the board restarts and waits a second or two for a new program. Since I'm not using any of the Arduino libraries, I don't have that available yet and I have to reset my board "manually" before running avrdude - I have a button wired up for that purpose. I'll look at replicating this feature by hand later...

That's it for now, I'll continue tinkering and report back if I find more context!


[Caterina]: https://github.com/adafruit/Caterina-Bootloader
[ws2812B-arduino]: https://github.com/PinkNoize/ws2812B-arduino
[datasheet]: ...
[avr109]: https://ww1.microchip.com/downloads/en/AppNotes/doc1644.pdf
[ihex]: https://en.wikipedia.org/wiki/Intel_HEX