# Single-port keyboard handler

This is a basic handler for a single PS/2 active keyboard. The device has the ability to both send and receive commands.

The PS/2 bus works as follows: The DATA pin is controlled by both the host and the device, but the CLK pin is usually controlled by only the device.
When reading a packet from the PS/2 device, a capacitor is charged whenever the CLK line goes low. Data is latched on every LOW clock edge, and the capacitor is allowed to discharge slowly.
After the entire packet is sent, the capacitor will begin to slowly discharge.
Once it reaches a certain theshold, the capacitor will activate a signal that will trigger the IRQB pin.
Since the lines for sending and receieving commands are shared, a control system will be implemented to prevent the receiver register from being overwritten. Also to prevent erroneous interrupts.

## Sending packets
Sending commands must be followed by a specific sequence of steps.
