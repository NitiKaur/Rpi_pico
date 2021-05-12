Connection to the MicroPython REPL
##################################

When MicroPython boots for the first time, it will sit and wait for you to connect and tell it what to do. You can load a .py file from your computer onto the board, but a more immediate way to interact with it is through what is called the readevaluate-print loop, or REPL (often pronounced similarly to "ripple").
	
	*Read* -  MicroPython waits for you to type in some text, followed by the enter key.
	
        *Evaluate* - Whatever you typed is interpreted as Python code, and runs immediately.
	
        *Print* -  Any results of the last line you typed are printed out for you to read.
        
        *Loop* -  Go back to the start — prompt you for another line of code.

There are two ways to connect to this REPL, so you can communicate with the MicroPython firmware on your board:
over USB, and over the UART serial port on Raspberry Pi Pico GPIOs.

USB
===
**Connecting from a Raspberry Pi over USB**

The MicroPython firmware is equipped with a virtual USB serial port which is accessed through the micro USB
connector on Raspberry Pi Pico. Your computer should notice this serial port and list it as a character device, most likely
/dev/ttyACM0.

	TIP-You can run ls /dev/tty* to list your serial ports. There may be quite a few, but MicroPython’s USB serial will start
	with /dev/ttyACM. If in doubt, unplug the micro USB connector and see which one disappears. If you don’t see
	anything, you can try rebooting your Raspberry Pi.

You can install minicom to access the serial port:

      +----------------------------+
      |	$ sudo apt install minicom |
      +----------------------------+
and then open it as such:

      +-----------------------------+
      |	$ minicom -o -D /dev/ttyACM0|
      +-----------------------------+

Where the -D /dev/ttyACM0 is pointing minicom at MicroPython’s USB serial port, and the -o flag essentially means "just do
it". There’s no need to worry about baud rate, since this is a virtual serial port.
Press the enter key a few times in the terminal where you opened minicom. You should see this:
      +----+
      |	>>>|
      +----+

This is a prompt. MicroPython wants you to type something in, and tell it what to do.
If you press CTRL-D on your keyboard whilst the minicom terminal is focused, you should see a message similar to this:

     
        MPY: soft reboot                                                               
	MicroPython v1.13-422-g904433073 on 2021-01-19; Raspberry Pi Pico with RP2040   
	Type "help()" for more information. 
	>>>

This key combination tells MicroPython to reboot. You can do this at any time. When it reboots, MicroPython will print
out a message saying exactly what firmware version it is running, and when it was built. Your version number will be
different from the one shown here.


UART
====

.. warning::

	REPL over UART is disabled by default.

The MicroPython port for RP2040 does not expose REPL over a UART port by default. However this default can be
changed in the ports/rp2/mpconfigport.h source file. If you want to use the REPL over UART you’re going to have to build
MicroPython yourself.
Go ahead and download the MicroPython source and in ports/rp2/mpconfigport.h change MICROPY_HW_ENABLE_UART_REPL to 1
to enable it.

	#define MICROPY_HW_ENABLE_UART_REPL (1) // useful if there is no USB

Then continue to follow the instructions in Section 1.3 to build your own MicroPython UF2 firmware.
This will allow the REPL to be accessed over a UART port, through two GPIO pins. The default settings for UARTs are
taken from the C SDK.

The table below shows the Default UART ettings in MicroPython

	+--------------+--------------+
        | *Function*   |  *Default*   |
        +--------------+--------------+
        |UART_BAUDRATE |    115200    |
        +--------------+--------------+
        |UART_BITS     |      8       |
        +--------------+--------------+
        |UART_STOP     |      1       |   
        +--------------+--------------+
        |UART_TX       |    Pin 0     |
        +--------------+--------------+
        |UART_RX       |    Pin 1     |
        +--------------+--------------+
        |UART_TX       |    Pin 4     |
        +--------------+--------------+
        |UART_RX       |    Pin 5     |
        +--------------+--------------+

This alternative interface is handy if you have trouble with USB, if you don’t have any free USB ports, or if you are using
some other RP2040-based board which doesn’t have an exposed USB connector.

.. note::

	This initially occupies the UART0 peripheral on RP2040. The UART1 peripheral is free for you to use in your Python code
	as a second UART.

The next thing you’ll need to do is to enable UART serial on the Raspberry Pi. To do so, run raspi-config

      +--------------------+
      |	$sudo raspi-config |
      +--------------------+

and go to Interfacing Options → Serial and select "No" when asked "Would you like a login shell to be accessible over
serial?" and "Yes" when asked "Would you like the serial port hardware to be enabled?". You should see something like

 ---------------------------------------------image

Leaving raspi-config you should choose "Yes" and reboot your Raspberry Pi to enable the serial port.
You should then wire the Raspberry Pi and the Raspberry Pi Pico together with the following mapping:


UART
####

.. note:: 

	REPL over UART is disabled by default. See Section 2.2 for details of how to enable REPL over UART

Example usage looping UART0 to UART1.

.. code-block:: python3

	 from machine import UART, Pin
 	import time
 
 	uart1 = UART(1, baudrate=9600, tx=Pin(8), rx=Pin(9))
 
 	uart0 = UART(0, baudrate=9600, tx=Pin(0), rx=Pin(1))
 
 	txData = b'hello world\n\r'
 	uart1.write(txData)
	time.sleep(0.1)
	rxData = bytes()
	while uart0.any() > 0:
	rxData += uart0.read(1)

	print(rxData.decode('utf-8'))

ADC
###

An analogue-to-digital converter (ADC) measures some analogue signal and encodes it as a digital number. The ADC on
RP2040 measures voltages.

An ADC has two key features: its resolution, measured in digital bits, and its channels, or how many analogue signals it
can accept and convert at once. The ADC on RP2040 has a resolution of 12-bits, meaning that it can transform an
analogue signal into a digital signal as a number ranging from 0 to 4095 – though this is handled in MicroPython
transformed to a 16-bit number ranging from 0 to 65,535, so that it behaves the same as the ADC on other MicroPython
microcontrollers.
RP2040 has five ADC channels total, four of which are brought out to chip GPIOs: GP26, GP27, GP28 and GP29. On
Raspberry Pi Pico, the first three of these are brought out to GPIO pins, and the fourth can be used to measure the VSYS
voltage on the board.
The ADC’s fifth input channel is connected to a temperature sensor built into RP2040.
You can specify which ADC channel you’re using by pin number, e.g.

.. code-block:: python3

	adc = machine.ADC(26) # Connect to GP26, which is channel 0

or by channel

.. code-block:: python3

	adc = machine.ADC(4) # Connect to the internal temperature sensor
	adc = machine.ADC(0) # Connect to channel 0 (GP26)

An example reading the fourth analogue-to-digital (ADC) converter channel, connected to the internal temperature
sensor:

.. code-block:: python3

        import machine
	import utime
 
        sensor_temp = machine.ADC(4)
 	conversion_factor = 3.3 / (65535)
 
	while True:
 	reading = sensor_temp.read_u16() * conversion_factor
 
	The temperature sensor measures the Vbe voltage of a biased bipolar diode, connected to
	the fifth ADC channel
	Typically, Vbe = 0.706V at 27 degrees C, with a slope of -1.721mV (0.001721) per degree.
	temperature = 27 - (reading - 0.706)/0.001721
	print(temperature)
	utime.sleep(2)

Interrupts
##########

You can set an IRQ like this:

.. code-block:: python3

	from machine import Pin

	p2 = Pin(2, Pin.IN, Pin.PULL_UP)
	p2.irq(lambda pin: print("IRQ with flags:", pin.irq().flags()), Pin.IRQ_FALLING)   

It should print out something when GP2 has a falling edge

Multicore Support
#################

Example Usage:

.. code-block:: python3 	

	import time, _thread, machine
 
 	def task(n, delay):
 	led = machine.Pin(25, machine.Pin.OUT)
 	for i in range(n):
 	led.high()
 	time.sleep(delay)
	led.low()
	time.sleep(delay)
	print('done')

	_thread.start_new_thread(task, (10, 0.5))

Only one thread can be started/running at any one time, because there is no RTOS just a second core. The GIL is not
enabled so both core0 and core1 can run Python code concurrently, with care to use locks for shared data

I2C
###

.. code-block:: python3

	from machine import Pin, I2C
 
 	i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=100000)
 	i2c.scan()
	i2c.writeto(76, b'123')
 	i2c.readfrom(76, 4)
 
	i2c = I2C(1, scl=Pin(7), sda=Pin(6), freq=100000)
	i2c.scan()
	i2c.writeto_mem(76, 6, b'456')
	i2c.readfrom_mem(76, 6, 4)

I2C can be constructed without specifying the frequency, if you just want all the defaults.

.. code-block:: python3

	from machine import I2C

	i2c = I2C(0) # defaults to SCL=Pin(9), SDA=Pin(8), freq=400000


.. warning::

	There may be some bugs reading/writing to device addresses that do not respond, the hardware seems to lock up in some cases

	+----------------+----------------+
        |*Function*      |   *Default*    |
        +----------------+----------------+
        | I2C Frequency  |  400,000       |
        +----------------+----------------+
        | I2C0 SCL       |  Pin 9         |
        +----------------+----------------+
        | I2C0 SDA       |  Pin 8         |
        +----------------+----------------+
        | I2C1 SCL       |  Pin 7         |
        +----------------+----------------+
        | I2C1 SDA       |  Pin 6         |
        +----------------+----------------+

SPI
###

.. code-block:: python3

 	from machine import SPI
 
 	spi = SPI(0)
 	spi = SPI(0, 100_000)
 	spi = SPI(0, 100_000, polarity=1, phase=1)
 
	spi.write('test')
 	spi.read(5)
 
	buf = bytearray(3)
	spi.write_readinto('out', buf)


.. note::

	The chip select must be managed separately using a machine.Pin.

PWM
###

.. code-block:: python3

	# Example using PWM to fade an LED.
 
 	import time
 	from machine import Pin, PWM
 
 
	# Construct PWM object, with LED on Pin(25).
	pwm = PWM(Pin(25))
 
	# Set the PWM frequency.
	pwm.freq(1000)

	# Fade the LED in and out a few times.
	duty = 0
	direction = 1
	for _ in range(8 * 256):
	duty += direction
	if duty > 255:
	duty = 255
	direction = -1
	elif duty < 0:
	duty = 0
	direction = 1
	pwm.duty_u16(duty * duty)
	time.sleep(0.001)


PIO Support
###########

Current support allows you to define Programmable IO (PIO) Assembler blocks and using them in the PIO peripheral,
more documentation around PIO can be found in Chapter 3 of the RP2040 Datasheet and Chapter 4 of the Raspberry Pi
Pico C/C++ SDK book.
The Raspberry Pi Pico MicroPython introduces a new @rp2.asm_pio decorator, along with a rp2.PIO class. The definition of
a PIO program, and the configuration of the state machine, into 2 logical parts:
• The program definition, including how many pins are used and if they are in/out pins. This goes in the @rp2.asm_pio
definition. This is close to what the pioasm tool from the SDK would generate from a .pio file (but here it’s all
defined in Python).
• The program instantiation, which sets the frequency of the state machine and which pins to bind to. These get set
when setting a SM to run a particular program.
The aim was to allow a program to be defined once and then easily instantiated multiple times (if needed) with different
GPIO. Another aim was to make it easy to basic things without getting weighed down in too much PIO/SM
configuration.
Example usage, to blink the on-board LED connected to GPIO 25

.. code-block:: python3

	import time
	import rp2
 	from machine import Pin
 
	# Define the blink program. It has one GPIO to bind to on the set instruction, which is an
  	output pin.
 	# Use lots of delays to make the blinking visible by eye.
 	@rp2.asm_pio(set_init=rp2.PIO.OUT_LOW)
	def blink():
 	wrap_target()
	set(pins, 1) [31]
	nop() [31]
	nop() [31]
	nop() [31]
	nop() [31]
	set(pins, 0) [31]
	nop() [31]
	nop() [31]
	nop() [31]
	nop() [31]
	wrap()

	# Instantiate a state machine with the blink program, at 2000Hz, with set bound to Pin(25)
 	(LED on the rp2 board)
	sm = rp2.StateMachine(0, blink, freq=2000, set_base=Pin(25))

	# Run the state machine for 3 seconds. The LED should blink.
	sm.active(1)
	time.sleep(3)
	sm.active(0)

The idea is that for the 4 sets of pins (in, out, set, sideset, excluding jmp) that can be connected to a state machine,
there’s the following that need configuring for each set:

	1. base GPIO
	2. number of consecutive GPIO
	3. initial GPIO direction (in or out pin)
	4. initial GPIO value (high or low

In the design of the Python API for PIO these 4 items are split into "declaration" (items 2-4) and "instantiation" (item 1).
In other words, a program is written with items 2-4 fixed for that program (eg a WS2812 driver would have 1 output pin)
and item 1 is free to change without changing the program (eg which pin the WS2812 is connected to).
So in the @asm_pio decorator you declare items 2-4, and in the StateMachine constructor you say which base pin to use
(item 1). That makes it easy to define a single program and instantiate it multiple times on different pins (you can’t
really change items 2-4 for a different instantiation of the same program, it doesn’t really make sense to do that).
And the same keyword arg (in the case about it’s sideset_pins) is used for both the declaration and instantiation, to
show that they are linked.
To declare multiple pins in the decorator (the count, ie item 2 above), you use a tuple/list of values. And each item in the
tuple/list specified items 3 and 4. 

IRQ
###

.. code-block:: python3

	import time
	import rp2

	@rp2.asm_pio()
	def irq_test():
	wrap_target()
	nop() [31]
	nop() [31]
	nop() [31]
	nop() [31]
	irq(0)
	nop() [31]
	nop() [31]
	nop() [31]
	nop() [31]
	irq(1)
	wrap()

	rp2.PIO(0).irq(lambda pio: print(pio.irq().flags()))

	sm = rp2.StateMachine(0, irq_test, freq=2000)
	sm.active(1)
	time.sleep(1)
	sm.active(0)


UART TX
#######

.. code-block:: python3

        # Example using PIO to create a UART TX interface
 
 	from machine import Pin
 	from rp2 import PIO, StateMachine, asm_pio
 
	UART_BAUD = 115200
	PIN_BASE = 10
 	UM_UARTS = 8
 

	@asm_pio(sideset_init=PIO.OUT_HIGH, out_init=PIO.OUT_HIGH, out_shiftdir=PIO.SHIFT_RIGHT)
	def uart_tx():
	# Block with TX deasserted until data available
	pull()
	# Initialise bit counter, assert start bit for 8 cycles
	set(x, 7) .side(0) [7]
	# Shift out 8 data bits, 8 execution cycles per bit
	label("bitloop")
	out(pins, 1) [6]
	jmp(x_dec, "bitloop")
	# Assert stop bit for 8 cycles total (incl 1 for pull())
	nop() .side(1) [6]

	# Now we add 8 UART TXs, on pins 10 to 17. Use the same baud rate for all of them.
	uarts = []
	for i in range(NUM_UARTS):
	sm = StateMachine(
	i, uart_tx, freq=8 * UART_BAUD, sideset_base=Pin(PIN_BASE + i), out_base=Pin
 	 (PIN_BASE + i)

	sm.active(1)
	uarts.append(sm)

	We can print characters from each UART by pushing them to the TX FIFO
	def pio_uart_print(sm, s):
		for c in s:
	sm.put(ord(c))

.. note::

	You need to specify an initial OUT pin state in your program in order to be able to pass OUT mapping to your SM instantiation, even though in this program it is redundant because the mappings overlap.
