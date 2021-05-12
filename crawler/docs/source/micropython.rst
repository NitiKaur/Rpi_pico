Getting started with MicroPython on RaspberryPi Pico Port
=========================================================

These instructions focus on Windows, however the guide can be used for Linux and Mac installation as well.
 using the tools for your operating system instead.


**1. Installation of MicroPython on Raspberry Pi Pico**


MicroPython is basically Python programming language for microcontrollers. The simple to understand syntax coupled with extensive libraries designed to make programming various  development boards easier for beginners.While regular Raspberry Pi boards do use Python, you'll need to follow a dedicated Raspberry Pi tutorial as the steps here don't apply to single-board computers.

The Raspberry Pi Foundation has made it incredibly easy to install MicroPython onto the Pi Pico. It uses the UF2 file extension, designed specifically for flashing microcontrollers over USB. Instead of needing a special programmer or piece of software, you can copy code over like you would a file to a pen drive or external hard drive. 

The MicroPython environment is available as a downloadable UF2 file from the `Pi Foundation website <https://www.raspberrypi.org/documentation/rp2040/getting-started/#getting-started-with-micropython>`_




.. image:: /img/Pi_Pico_uf2.png
    :alt: ESP32 board
    :width: 640px

To install the MicroPython environment on to your Raspberry Pi Pico, follow these steps:

1. Download the MicroPython UF2 file from the Raspberry Pi website
2. Hold down the BOOTSEL button on your Pico and plug it into your computer's USB port.
3. Open Explorer, and open the RPI-RP2 directory like you would any other hard drive
4. Drag and drop the UF2 file into the RPI-RP2 directory

 You can now open a terminal program like Putty to talk to the Pi Pico over the USB Serial port, but there's a much better way to interact with your Pico: The Thonny IDE.

**2. Install Thonny IDE**


Thonny is an open-source Python integrated development environment (IDE) designed for beginners. It's powerful, easy to understand, and already comes with MicroPython and Raspberry Pi Pico support.

To get Thonny, download it for free from the official website by clicking the `link <https://thonny.org/>`_ in the top right corner.

..image:: /home/niti/Downloads/Thonny_site.png

When the download finishes, install and open the Thonny IDE. You'll be asked what language you'd like Thonny to run in before being greeted with a new Thonny window. Make sure your Pi Pico is plugged in, click on the button on the bottom right of the window that reads Python, and change it to MicroPython (Raspberry Pi Pico).

The REPL window should change to show you are now running on the Pico, and you can test it out with a quick Hello World!

Let's test it.

.. code-block:: python3

	print('Hello World')

The output should be-
	
	Hello World

**3. Programm the Raspberry Pi Pico**


MicroPython is identical in syntax to regular Python, and if you aren't familiar, it's worth `learning the basics of Python <https://www.makeuseof.com/python-hello-world/>`_  to understand Pi Pico code better. If you don't know Python, don't worry! This tutorial uses example code to get you going without needing any previous programming experience.

The Raspberry Pi Foundation provides example code to help you get started coding the Pico, which is available from its official `GitHub <https://github.com/raspberrypi/pico-micropython-examples>`_ repository. To get the examples, click on Code > Download ZIP and extract them to a directory of your choice. In Thonny, use Ctrl + o or select File > Open to open the blink.py example. The code should look like this:

.. code-block:: python3

	from machine import Pin, Timer
	led = Pin(25, Pin.OUT)
	tim = Timer()
	def tick(timer):
	    global led
	    led.toggle()
	tim.init(freq=2.5, mode=Timer.PERIODIC, callback=tick)

Click the green run button. A popup will ask you where you want to save the file. Select your Raspberry Pi Pico, and rename the file to main.py.

..image:: /home/niti/Downloads/thonny_ide.png


.. image:: /img/thonny_ide.png
    :alt: ESP32 board
    :width: 640px
You should see your LED blinking! Renaming the file to main.py is optional, though if you want your code to run when the Pico is connected to an external power source rather than a computer, you'll need to do it. The Pico looks for a main.py when it boots up for instructions, and if it isn't there, it won't do anything.

Another neat thing you may notice is that the REPL is still active. The timer and LED are working in the background now, leaving you free to send more commands to the Pico through the REPL


**4. Something More Advanced**


Getting an LED to blink is a great start, but to get a sense of just how useful the Raspberry Pi Pico can be, let's test the onboard temperature sensor. Once again, the Raspberry Pi foundation makes this easy to do. It provides example code to read from the onboard sensor, convert it into human-readable temperature information, and print it to the Thonny REPL.

..image:: /home/niti/Downloads/temp_sensor.png

Open adc > temperature.py in the examples folder, or simply copy the raw code directly from GitHub into Thonny, before saving it as main.py. The code should look like this:

.. code-block:: python3

	import machine
	import utime
	sensor_temp = machine.ADC(4)
	conversion_factor = 3.3 / (65535)
	while True:
	    reading = sensor_temp.read_u16() * conversion_factor
    
	    # The temperature sensor measures the Vbe voltage of a biased bipolar diode, connected to the fifth ADC channel
	    # Typically, Vbe = 0.706V at 27 degrees C, with a slope of -1.721mV (0.001721) per degree. 
	    temperature = 27 - (reading - 0.706)/0.001721
	    print(temperature)
	    utime.sleep(2)

Click the green run button, and the code should start to run, printing the current ambient temperature into the Thonny REPL.

**5. Let your imagination go wild**


Now that you are set up to program the Pico, you can experiment with its features using the MicroPython library. There are already many beginner projects and tutorials for the Pi Pico, and the Raspberry Pi Foundation has even released an official book on the Pico, available from the `Raspberry Pi website <https://www.raspberrypi.org/blog/new-book-get-started-with-micropython-on-raspberry-pi-pico/>`_


