
This how to guide explains how to setup your MPF configuration files
to interface with a Multimorphic P3-ROC pinball controller. Most of
this information is available elsewhere in the documentation, but this
guide pulls it all together in one place.



(A) Installing the P3-ROC drivers on your computer
--------------------------------------------------

In order to use a P3-ROC with MPF, you need to have the P3-ROC
software libraries installed on your computer. These are called
*libpinproc* and *pypinproc*. (Libpinproc is the actual interface
library, written in C, and pypinproc is a wrapper for that library
that makes it available to Python programs such as MPF.) The MPF all-
in-one installer for Windows or Linux will install the necessary
drivers for you. (If you don't want to use the all-in-one installer,
you can still follow the `platform-specific links`_ for installing MPF
which have the P-ROC software libraries pre-compiled and ready to copy
to your computer.)



(B) Configuring your platform settings for a P3-ROC
---------------------------------------------------

To use MPF with a P3-ROC, you need to configure your platform as
p3_roc, like this:


::

    
    hardware:
        platform: p3_roc
        driverboards: pdb


You also need to configure the `driverboards:` entry for P-ROC driver
boards (pdb).



(C) Configuring switches
------------------------

For switches, you can use all the settings as outlined in the `
`switches:``_ section of the config file reference.



Number
~~~~~~

The P3-ROC uses switches connected to SW-16 boards. You have two
options for entering switch numbers:


+ You can enter the raw switch number. Board address 0 is 0-15, board
  address 1 is 16-31, etc.
+ You can enter them as a combination of board/bank/switch, like
  `1/0/2`.



::

    
    switches:
      some_switch:
        number: 0/1/4




Debounce
~~~~~~~~

The P3-ROC doesn't support variable debounce times—it's either on or
off. So with P3-ROC controllers, a debounce value of `True` (or any
non-zero integer) means that debounce is enabled for that switch, and
a value of `False` (or `0)` means that debounce is disabled.



(D) Configuring coils
---------------------

There are a few things to know about controlling drivers and coils
with the P3-ROC.



Number
~~~~~~

For PD-16-based coils, the coil numbering format is *Ax-By-z.* The “A”
and “B” capital letters are required. (A means *Address*, B means
*Bank*). The lowercase x, y, and z letters should be replaced with
numbers to represent the following on a PD-16 driver board:


+ x : Board address (0-7)
+ y : Bank address (0 for A, 1 for B)
+ z : Output number (0-7)




Hold Power
~~~~~~~~~~

If you want to hold a driver on at less than full power, the P3-ROC
does this by using "on time" and "off time" parameters, in
milliseconds, that it switches back-and-forth. If you set a
`hold_power:` entry (which is a power value from 0 to 8), then the
following actual settings will be applied with a P3-ROC:


+ 0: solid off
+ 1: 1ms on / 7ms off
+ 2: 1ms on / 3ms off
+ 3: 3ms on / 5ms off
+ 4: 1ms on / 1ms off
+ 5: 5ms on / 3ms off
+ 6: 3ms on / 1ms off
+ 7: 7ms on / 1ms off
+ 8: solid on 100%


If you want more precise control over the timing of how your P3-ROC
drivers will be enabled, you can use the optional settings *pwm_on_ms*
and *pwm_off_ms*. For example, to configure a coil to enable with a
pattern of 2ms on / 9ms off, you'd use the following:


::

    
    coils:
      some_coil:
        number: A0-B1-6
        pwm_on_ms: 2
        pwm_off_ms: 9




(E) Configuring lights on a lamp matrix
---------------------------------------

If you're using your P3-ROC to control a lamp matrix via a PD-8x8,
then you need to configure the number for each lamp in your
matrix_lights: section with an entry that contains a bunch of letters
and numbers which specify the specific columns and row outputs that
make up each lamp. It’s probably easiest to look at an example.


::

    
    matrix_lights:
      some_light:
        number: C-A2-B0-0:R-A2-B1-0


Notice there are two parts to the number, separated by a colon. The
first part starts with `C` which means “column.” The `A2` means the
column output is on the PD-8×8 board at address 2. `B0` means that
it’s connected to “bank 0”, and the final `-0` means it’s connected to
“output 0”. The same is true for the row side after the colon. So
putting it all together, `C-A2-B0-0:R-A2-B1-0` means that the light
*some_light* is connected the columnwhich is on output 0 of bank 1 of
the PD-8×8 at address 2, and it’s connected to the row which leads
back to output 0 of bank 1 of the PD-8×8 running at address 2. (Phew!)
Luckily this is only something you have to work out once. :)



(F) Configuring LEDs connected to a PD-LED board
------------------------------------------------



LED numbering withPD-LED boards
```````````````````````````````

The `PD-LED board`_ directly drives single color LED outputs. When you
use it with RGB LEDs, you combine three output per LED. The PD-LED
supports both common cathode and common anode LEDs, so each LED you
buy has four pins (red, green, blue, and common). When you configure
the hardware number for a PD-LED RGB LED, you specify four parts. The
first part is the address of the PD-LED board on the serial chain (as
configured via the DIP switches on the PD-LED, the second is the
output number of the red segment, the third is the green segment, and
the fourth is the blue segment. You separate these all with dashes, so
an example PD-LED configuration might look like this:


::

    
    l_led0:
        number: 8-0-1-2


The example above configures the LED that you’ll refer to as “led0” as
the LED connected to PD-LED board at address 8, using outputs 0, 1,
and 2 as its R, G, and B connections. The PD-LEDallows you to use
either common anode or common cathode LEDs. (See the `PD-LED
documentation`_ for details. The type of LED would dictate whether you
hook it up between the PD-LED’s output and ground, or between the
output and 3.3v.) You can then use the configfile to specify which
type of LED you have, such as:


::

    
    l_shoot_again:
        number: 8-60-61-62
        polarity: True


True = common cathode (or common ground), False = common anode (or
common 3.3V) Note that DIP Switch 6 on the PD-LED board controls
whether the “default” state of the LEDs after a reset is high or low.
Basically it’s whether all the LEDs turn on or turn off when the board
is reset. Which position does what is dependent on whether you’re
controlling the anode or the cathode with your outputs, so basically
if you turn on your PD-LED and all your LEDs turn on, then flip DIP
switch 6 on the PD-LED to the opposite position and power cycle the
board.



(G) Configuring flashers
------------------------

If you have flashers which are connected up to a PD-16, then you
configure their number in the same way you configure a coil number as
outlined in Section C.



(H) Using the accelerometer
---------------------------

The P3-ROC has an accelerometer built-in which can be used to detect
vibrations (for tilt) as well as to detect whether the pinball machine
is level. At this point we don't have support in MPF for the
accelerometer, though we have started playing with it and it's
something that we'll add at some point.

.. _switches:: https://missionpinball.com/docs/configuration-file-reference/switches/
.. _platform-specific links: https://missionpinball.com/docs/installing-mpf/
.. _PD-LED documentation: http://pinballcontrollers.com/index.php/products/driver-boards/96-p-roc-20


