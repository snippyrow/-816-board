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

`ECCCPPMK`

The first bit read from this register declares whether the keyboard has sent the interrupt. The next bit declares if the mouse has sent the interrupt. If both are high, then both devices have sent the interrupt at the same time. If this happens, both packets are preserved in each register. The third bit in each register is the result of each parity bit from the ports. It is also overwritten on the next packet. The third bit is for the keyboard parity, and the fourth is for the mouse parity. If a parity errors happens, send another command to the respective device to re-send the command.

When reading each received packet data register, the interrupt returns to zero in the status register. For examples, reading 0x1 makes the first bit go to zero.
Sending commands happens in hardware, without need for system intervention. The PS/2 clock signal for each port is derived from the system clock, being divided by a value selected in the three C bits. These three C bits are the only writable bits in the control register, and go towards a decoder. It can select from 1/16 system speed up to 1/2048 speed. This is important because this system can take advantage of a wide range of clock speeds, and this would mess with the PS/2 bus speed. It must be around 10-20khz. If the divisor is set larger, then the host must write commands slower. If it takes 11 bits per packet, at 16 block cycles per bit, then the host must not send another command for 176 clock cycles. For 1mhz (1/64), that increases to about one every 700 cycles.

The host CAN send a command to a mouse within this timeframe, and both are sent and received independantly.

The E bit in the control register can be set to 0 to disable the input card, which is set to at default. This disables the IRQB line on the card, allowing another device to use it if needed.

Receiving packets happen asynchronously, as the CLK/DATA lines are usually pulled high by a resistor. When CLK first goes low, it triggers a capacitor to quickly charge and slowly discharge. Data is latched on each clock falling cycle, into a shift register. There are two 8-bit registers to hold all of the bits (11), and the parity bit is outputted in the control register when read. After the packet is completed, the capacitor will discharge and set a latch. This is OR'd with keyboard/mouse latches, so both must be acknowledged before the line goes dead. Sending commands also happens over hardware. Data is written into the register by the system, and a counter is reset. Two decoders are set to send the start bit, packet byte, parity and stop bit. Probably will be two 74HC238's. After the counter is over a certain value (11), the bus stops pulsing the CLK signal and the DATA bus is permanently released. The counter is ticked on every divided clock pulse. Pairty is computed in hardware against the contents of each input register automatically. By the time all the bits are sent, the parity computation will be completed. Parity is computed against an 8-bit number using eight XOR gates, or a single 8-input XOR gate.

Alternativly, use a 74HC280 to generate odd parity. It implements everything internally. In order to handle the counter properly, implement a 4-input NAND or AND gate to check 0b1010 (10) and stop the counter from going any further. ALso run that to block the CLK from being changed. Data should be updated on the DATA bus after the rising edge of the CLK and will be latched by the device during the falling edge.
