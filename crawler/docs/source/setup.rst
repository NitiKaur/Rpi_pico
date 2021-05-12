Setting up the Pico Port
=========================


Setting the port with J-Link
----------------------------
 
The Pico port has an external SWD connector. The 3 pins being SWCLK, GND, SWDIO.To setup the board using J-Link, the following setups are required-

	1. Soldering the 3-pin pin strip to the debug connector pads. 
	2. Solder the  pin Strip to the GPIO pads of the evaluation board. The relevant pad being Pad 36 (3V3(OUT))
        3. Connecting the J-Link to the evaluation board, using jumper wires as follows:

        +---------------+------------+-----------+
	|  Pin RP Pico  | Pin J-Link |   Signal  |   
	+---------------+------------+-----------+
	|"DEBUG" "SWCLK"|      9     |   SWCLK   |     
	+---------------+------------+-----------+
	|"DEBUG" "GND"  |      4     |   GND     |      
	+---------------+------------+-----------+
        |"DEBUG" "SWDIO"|      7     |   SWDIO   |
        +---------------+------------+-----------+     
        |"3V3" Pad 36   |      1     |   VTREF   |
        +---------------+------------+-----------+


Using SEGGER Flying Wire Adapter
--------------------------------
For a more reliable, simpler solution, the SEGGER Flying Wire Adapter is recommended.

Adapts from the standard 20 pin 0.1" J-Link connector to a 10-pin row, allowing to connect J-Link to target PCBs via flying wires.

Used in the following cases -

	1. When hooking up prototype HW where no final connector is decided yet. 
	2. Manually connecting a J-Link to ST Discovery boards. 






For more information see this `USER MANUAL <https://www.segger.com/downloads/jlink/UM08001>`_

