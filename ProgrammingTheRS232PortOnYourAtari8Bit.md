# RS232 Programming for the Atari 8-bit
## NOTE: A good portion of this was gleaned from reading the 850 technical manual.

The Atari 8-bit series of computers have a built-in operating system that provides high-level commands for RS-232 communications, such as XIO, and low-level memory access commands like POKE and PEEK. Here’s an overview of how they work and how you could use them to configure a modem and listen for an inbound call.

### RS-232 Communications

The RS-232 port on Atari computers is often accessed through an interface module, like the Atari 850 Interface Module. This module converts the computer's SIO (Serial Input/Output) port to standard RS-232 serial ports, which you can then use to connect devices like modems.

### XIO Command

XIO is an Atari BASIC command used for extended I/O operations, such as controlling the RS-232 ports. It's more abstract than POKE and PEEK, providing high-level control over devices.

### Examples of XIO commands for RS-232:

- **XIO 36:** Sets baud rate, word size, stop bits, and readies the port for communication.
- **XIO 34:** Controls RTS, DTR, and XMT signals, which are crucial for establishing communication lines with the modem.
- **XIO 38: **Sets translation and parity modes.
- **XIO 40:** Initiates Concurrent I/O Mode, which is necessary for ongoing communication without blocking other operations.

## POKE and PEEK Commands
**POKE** is used to write data to a specific memory address, and **PEEK** is used to read data from a memory address. In the context of RS-232 communications, you might use **POKE** to directly manipulate hardware registers that control the modem signals, while **PEEK** could be used to check the status of these signals.

## Basic Example of Configuring a Modem
Here is a simple example of how to configure a modem for 9600 baud, 8 data bits, 1 stop bit, no parity, with DTR and RTS signals enabled, using XIO commands:

```basic
10 REM Initialize modem
20 XIO 36, #1, 14, 0, "R1:" ; Set baud rate to 9600
30 XIO 34, #1, 240, 0, "R1:" ; Turn on DTR and RTS
40 XIO 38, #1, 0, 0, "R1:" ; Set translation mode for no translation and no parity
50 XIO 40, #1, 0, 0, "R1:" ; Turn on Concurrent I/O Mode
```

## Listening for an Inbound Call

To listen for an inbound call, you would typically check the status of the Carrier Detect (CD) signal. This can be done using PEEK to check a system location that reflects the status of the CD signal, or by using an XIO command if the CD signal is tied to one of the RS-232 control signals:

```basic
100 REM Listen for inbound call
110 OPEN #1, 9, 0, "R1:" ; Open the modem device for input
120 STATUS #1, S ; Check the status of the modem
130 IF PEEK(747) = 0 THEN GOTO 120 ; Wait for the carrier detect
140 PRINT "Modem is now receiving a call"
```

In this example, the program opens the modem device, checks its status, and waits in a loop until the carrier is detected, indicating an incoming call. When the carrier is detected, it prints a message.

## Important Notes

- Make sure to close any open IOCBs when finished to avoid any conflicts.
- IOCB numbers other than 6 and 7 are recommended to avoid conflicts with graphics and LPRINT functions.
- Error handling is crucial; you should check for errors after each XIO command.
- The exact procedure can vary depending on the specific modem and its command set.
- The values used in POKE and PEEK commands, and the meaning of various bits in the XIO Aux1 and Aux2 parameters, can vary based on the specific hardware setup and should be referenced from the modem’s and the interface module’s documentation.
- The example provided is a simplified illustration of how you could use BASIC to interact with a modem on an Atari 8-bit computer. In practice, you would need to consult the documentation for the specific modem you're using, as well as the Atari Hardware Manual for detailed information on the correct memory locations and values to use.

## Basic Example for Modem Configuration and Listening for an Inbound Call

```basic
5 CLOSE #2
6 POKE 731,255 # Turn off the sound
10 A=0 # Variable initialization
15 XIO 36,#2,14,0,"R1:" # Set baud rate to 9600
20 XIO 34,#2,240,0,"R1:" # Turn on DTR and RTS
30 XIO 38,#2,64,0,"R1:" # Set translation mode, append LF to CR
40 OPEN #2,9,0,"R1:" # Open the modem device
1000 FOR D=1 TO 3000: NEXT D # Delay loop
1010 STATUS #2,C # Check modem status
1020 IF PEEK(747)<2 THEN GOTO 1000 # Wait for carrier detect
1030 FOR D=1 TO 3000: NEXT D # Delay loop
1035 XIO 40,#2,0,0,"R1:" # Start Concurrent I/O Mode
1040 PRINT #2;"ATA" # Send ATA command to answer call
1050 PRINT #2;"Hi there!" # Send greeting message over modem
1060 FOR D=1 TO 3000: NEXT D # Delay loop
1070 GOTO 5 # Loop back to start (close and reopen modem)
1080 POKE 731,0 # Turn the sound back on
```

This BASIC program is for use on an Atari 8-bit computer to configure a modem for communication and to listen for an incoming call. The program includes commands to set the baud rate, control signals, and translation modes, as well as a loop to check the modem's status for an incoming call.
