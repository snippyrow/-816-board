# Computer input controller
Like all computers, I need a way for the user to send input to the machine.

## Features
This device has the ability to send and receive commands to PS/2 devices, including keyboards and mice. To switch between the two, special commands can be sent that are interpreted by both devices.

Two ports are included: one for a keyboard and one for a mouse. May or may not be colroed, but text underneath will specity. Each port has two registers that can be accessed by the system, as well as a status register:
| Vector | Name |
| -------- | ------- |
| 0x0 | KBD Send |
| 0x1 | KBD Recv |
| 0x2 | Mouse Send |
| 0x3 | Mouse Recv. |
| 0x4 | Int. Status |

When either a keyboard or mouse port receive a packet, it sends a signal along the IRQB line on the main core. If it misses a packet, it will be overwritten with the next one and cannot be recovered. The interrupt status register is as follows:
`0000PPMK`
The first bit read from this register declares whether the keyboard has sent the interrupt. The next bit declares if the mouse has sent the interrupt. If both are high, then both devices have sent the interrupt at the same time. If this happens, both packets are preserved in each register. The third bit in each register is the result of each parity bit from the ports. It is also overwritten on the next packet. The third bit is for the keyboard parity, and the fourth is for the mouse parity. If a parity errors happens, send another command to the respective device to re-send the command.

When reading each received packet data register, the interrupt returns to zero in the status register. For examples, reading 0x1 makes the first bit go to zero.

