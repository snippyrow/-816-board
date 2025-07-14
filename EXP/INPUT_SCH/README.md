# Computer input controller
Like all computers, I need a way for the user to send input to the machine.

## Features
This device has the ability to send and receive commands to PS/2 devices, including keyboards and mice. To switch between the two, special commands can be sent that are interpreted by both devices.

Two ports are included: one for a keyboard and one for a mouse. May or may not be colroed, but text underneath will specity. Each port has two registers that can be accessed by the system:
| Vector | Name |
| -------- | ------- |
| 0x0 | KBD Send |
| 0x1 | KBD Recv |
| 0x2 | Mouse Send |
| 0x3 | Mouse Recv. |
