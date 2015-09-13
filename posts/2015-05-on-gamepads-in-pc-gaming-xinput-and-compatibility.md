<!--
.. title: On gamepads in PC gaming, XInput and compatibility
.. slug: on-gamepads-in-pc-gaming-xinput-and-compatibility
.. date: 2015-05-24 20:13:34 UTC
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

"Which gamepad should I buy for PC?"

A reasonable question, but with no single good answer, other than "er, it depends, what game would you play on it?". Gamepad support on PC is decent nowadays, much better than it used to be before the advent of Xbox360, but there's a mess here. Let's try to untangle things a bit.

Gamepads used to be available in tremendous variety and have now pretty much converged to a standard layout: 2 analog sticks, a D-pad, 4 face buttons, 2 shoulder buttons, 2 analog triggers, select, start and 2 secret buttons that you mostly press by accident. There's some variety on top of that, like vibration, left analog positioning, touchpads, motion sensors, but the general idea stays the same between PC, PlayStation 2-4, Xbox 1-360 and even Android (kind of, if you count the NVidia Shield). Would you believe that?

<!--more-->

# Competing Interfaces

For ages, PC games supported gamepads via DirectInput, a generic interface that also handled other controllers like joysticks, steering wheels, pretty much anything that has up to 8 analog axes, 128 physical buttons and an optional "hat switch" (or a d-pad). Supporting gamepads well in PC games wasn't easy back then (a developer wouldn't know what to expect from the device) and every player was prepared to spend his first few minutes in Settings, configuring every button to his liking and figuring out a scheme that works.

Microsoft saw the problem and made an interesting move: why not enrich DirectX with another interface? XInput was born, with a very simple goal in mind: support Xbox 360 gamepads and that's it. This move has interesting consequences: now a game developer exactly knows how an XInput device looks like and gains much more control over how the game is experienced: he can bind the actions to "Left shoulder" rather than an anonymous "button 6". Nice! Moreover, an Xbox 360 pad can work with a PC without any problems and a game can offer consistent experience on PC and Xbox ("press Y" instead of "Press button 3").

Unfortunately, twice the standard = twice the trouble. Let me just start with the fact that I needed to buy a new pad for PC just because Darksiders had no idea how to work with a DirectInput gamepad. Every PC game developer, from now on, has to answer the question: Do I support DirectInput devices? Xinput devices? Both? XInput is tempting because you know the layout upfront and don't have the implement e.g. custom key bindings, it's also easy to suggest actions to the player, but still you cut some audience off. 

So ideally, every PC gamer should have both a DirectInput and an Xinput gamepad at home, right? Oh, actually a few of each, in case you'd like to play with friends and you can't know what the next game's going to support.

Is there a way out of this madness? Can we have a gamepad that would Just Work (tm) with both standards, in all games?

# Let's work this around...

The problem, luckily, haven't gone unnoticed, and there's a solution. Hell, there are even TWO competing solutions, haven't we already establish that two is better than one?

## XInput-DirectInput compatibility layer

Let's start with the Microsoft approach. When you plug the Xbox360 controller into the PC, a game would basically notice not one, but two gamepads, one with DirectInput and one with XInput. This makes *some* sense: old games will still recognise the X360 gamepad as a DirectInput device, while for modern games the programmers are advised to [apply some voodoo][voodoo] to find out which DirectInput devices are actually Xinput pads and handle them differently. Except that the way to do it is complicated, unreliable ('// Check if the device ID contains "IG_"'? seriously?) so if someone gives up and decides to just drop DirectInput support, I won't really blame them.

[voodoo]: https://msdn.microsoft.com/en-us/library/windows/desktop/ee417014%28v=vs.85%29.aspx

But the plus side is that you can still enjoy old games with your brand new XInput gamepad! That is, if you don't try any crazy things, like attempting to use the trigger buttons.

[From MSDN:][voodoo]

> some functionality provided by XInput will be missing from the DirectInput implementation:
> 
> - The left and right trigger buttons will act as a single button, not independently

The following explanation is quite confusing but it basically means that DirectInput games will see both triggers as one combined axis, left trigger goes towards -1, right trigger towards +1, and pressing both doesn't work. It allegedly stems from the fact that in Xinput the triggers are analogs with range 0..1 each, while DirectInput only handles axes from -1..1 (with zero being neutral). For racing games you can still use the triggers for acceleration and braking, but other than that you'd mostly need to do without them.

(I don't get why the driver wouldn't map each trigger to a separate DirectInput axis that would only use the upper half of the full -1..1 range of motion? 0 for neutral, 1 for pressed, -1 never. Doesn't DirectInput handle 6 axes easily? I guess there's more to it.)

## Switchable pads

There's another approach around: take a normal gamepad and give it a physical XInput switch. So you're playing a DirectInput game? You're good. XInput game? Flip the switch and you have an Xbox-compatible controller. What more to expect? It works quite well assuming you actually know what DirectInput and XInput is. Gamers shouldn't *need* to know that. That said, it actually works quite well once you get going: Even in XInput mode, the DirectInput fallback is still there, so **you can actually leave the switch in XInput mode all the time**. And when you actually run into a game where you'd rather have LT/RT work as binary buttons rather than a combined axis, you just flip the switch back to DirectInput.

Now it cannot be too easy. For example the Logitech F310 series also as a different face button layout in Xinput and DirectInput modes: ABXY = 2314, consistently with the previous-gen Logitech Dual Action button layout. This isn't a big deal if you get to configure the keys, but still gets awkward sometimes. Speed Link Xeox Pro offers consistent numbering in DirectInput and XInput (ABXY = 1234), so in many games you won't even notice a big difference between the modes.

# Cleaning up

I must say I really like what is happening, despite all the troubles. I find XInput to be a very welcome citizen in the world of PC gaming, it relieves us of configuring the buttons for every game and making things Just Work for the new titles. Also I can't stress enough how much I appreciate that PC gamepad buttons finally have names! "Press RT" is just so much easier to comprehend than "Button 6" or whatever, especially if you used to have 4 pads, each with different numbering. Begone, old times!

Yeah, gamers need to upgrade their equipent to enjoy the new titles, since I don't expect game developers who rightfully leverage XInput to perform the DirectInput compatibility dance forever. Support for old titles is still there in new gamepads: If you buy a switchable model like the Xeox Pro or the F310 then you're pretty much covered, and even if you go with a plain X360/Xbone pad or a quality machine like Razer Sabretooth or Mad Catz Pro, the magic Xinput compatibility layer will still work for you in 90% of old titles.

Good job, Microsoft!

---

*Story time!*

*I have an old PS/2 digital gamepad that you'd plug between the keyboard and the PC; it was completely invisible to the system and instead you'd configure it by capturing keypresses and mapping them to buttons and d-pad directions. Basically a hardware Joy2key - some effort needed, but that's the price for 100% game compatibility! It even had an 4/8-direction d-pad mode switcher... And in case you didn't have a PS/2 keyboard, no worries, it also supported 5-pin DI* :-)
