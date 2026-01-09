Simple Interrupt Controller

Overview:
This project implements an 8-channel interrupt controller using Verilog HDL.
The controller is designed to interface with a processor and manage multiple interrupt sources efficiently.

It operates in two configurable modes:
-Polling mode
-User-defined priority mode

Features:

-Supports 8 interrupt inputs
-Two modes of operation:
  -Polling (no priority)
  -Custom priority (priority defined during initialization)
-Handshake-based communication with the processor
-Reset-safe operation with error detection

Operating Modes:
After a reset, the controller waits for a valid command from the processor through the bus.
The least significant 2 bits of the bus determine the operating mode:

Mode Bits	  Function
01	        Polling Mode
10	        Custom Priority Mode
The controller remains idle until a valid mode selection is detected.

Polling Mode:
To select polling mode, the processor drives the bus with xxxx_xx01 for one clock cycle.
Once detected, the controller enters the polling state.

Operation Flow:
-All interrupt sources are scanned sequentially in a loop
-If any interrupt is active:
  -intr_out is asserted
  -The controller waits for processor acknowledgement
-The processor acknowledges via intr_in using a High → Low → High transition
-The controller then places 01011_intrID on the bus
  -intrID corresponds to the active interrupt source
-This value remains on the bus until another acknowledgement is received
-After servicing the interrupt, the processor sends:
  -A final acknowledgement on intr_in
  -A confirmation code 10100_intrID on the bus
-If the ID or condition code does not match, the controller resets
-Otherwise, polling resumes for the next interrupt

Custom Priority Mode:
Custom priority mode functions similarly to polling mode, but interrupts are checked based on a predefined priority order instead of a fixed sequence.
Priority Configuration:
-After reset, the processor sends input in the format xxxyyy10
  -xxx → highest priority source ID
  -yyy → second highest priority source ID
-The processor must provide 4 such configuration cycles to assign priorities to all 8 sources

Runtime Behavior:
-The controller continuously checks interrupt sources from highest to lowest priority
-Once an active interrupt is detected, servicing proceeds the same way as polling mode
-The difference lies in the handshake codes:
  -Controller sends: 10011
  -Processor acknowledges with: 01100

Controller sends: 10011

Processor acknowledges with: 01100

Timing Diagram
===============================================================================

intr_out         __________________
            ____|                  |___________________________________________

intr_in     __________________      ______________      ___________      _____
                              |____|              |____|           |____|

data_bus    _______________________|||||||||||||||||||||___________||||||_____
           

-------------------------------------------------------------------------------
    Note - The first time the data_bus is active is when the controller
    drives the bus. Next time when it's active, the processor drives it.
-------------------------------------------------------------------------------
    Note - The timing diagram remains the same on both polling and custom
    priority modes. Only thing that changes is the ack data on the bus.
-------------------------------------------------------------------------------

Condition Codes
===============================================================================

    Polling:
        From Controller     -   01011
        From Processor      -   10100

    Custom Priority
        From Controller     -   10011
        From Processor      -   01100

===============================================================================
