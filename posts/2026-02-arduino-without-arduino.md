<!--
.. title: Arduino without Arduino
.. slug: arduino-without-arduino
.. date: 2001-01-01 12:00:00 UTC
.. tags: diy, arduino, electronics
.. category: dev
.. link:
.. description:
.. type: text
.. status: private
-->

Intro...

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

Quick check to ensure the running:

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

1) Which microcontroller is this? I'm working with a Sparkfun Pro Micro, which sports the ATmega32U4. For avrdude this translates to the "part" parameter, this one is called `m32u4`.
2) The Pro Micro runs the 32u4 in 8MHz on 3.3V, while an Arduino Leonardo (which uses the same IC) would run it on 16MHz on 5V. This is still the same microcontroller but it has to be configured properly using "fuses" - a few bytes of non-volatile memory that. I haven't fiddled with these just yet. Depending on that .........
3) Which programmer to use? .......


## Let's upload something

Here's an example C code for the Pro Micro. Note the absence of Arduino libraries, but other than that, it doesn't look different.

```c
int main() {}
```

First I compiled it via `avr-gcc` and obtained an `.elf` file:

```
...
```

The second step is to extract the `hex` file which is the part I understand the least

Once we have the file, let's upload it. 

...



[Caterina]: https://github.com/adafruit/Caterina-Bootloader