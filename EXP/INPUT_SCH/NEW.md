# Single-port keyboard handler

This is a basic handler for a single PS/2 active keyboard. The device has the ability to both send and receive commands.

The PS/2 bus works as follows: The DATA pin is controlled by both the host and the device, but the CLK pin is usually controlled by only the device.
When reading a packet from the PS/2 device, a capacitor is charged whenever the CLK line goes low. Data is latched on every LOW clock edge, and the capacitor is allowed to discharge slowly.
After the entire packet is sent, the capacitor will begin to slowly discharge.
Once it reaches a certain theshold, the capacitor will activate a signal that will trigger the IRQB pin.
Since the lines for sending and receieving commands are shared, a control system will be implemented to prevent the receiver register from being overwritten. Also to prevent erroneous interrupts.

## Sending packets
Sending commands must be followed by a specific sequence of steps. To begin sending, the host must pull the CLK lane LOW for >100us. To do this, use a monostable multivibrator to pull it low. This is the only time that the host will pull the clock lane low. Usually when a packet is received, it would go through a latch ticking high for a moment, but that action is inhibited at the moment a transaction is started. Due to various CLK speeds, a second capacitor system is set up. At the first clock pulse, and if the writing register is written, a signal goes LOW that inhibits the interrupt register. This has an interval LONGER than the first chain, meaning that no interrupts are triggered. Once it gets re-enabled, it allows it to send interrupts again. As a product of this, the incoming register gets voerwritten by the new data. This should not be an issue, since input registers should always be read from at the moment of an interrupt.

Data is latched by the device on the RISING edge of the CLK lane, and data should be updated on the FALLING edge. A 4-bit counter is for selecting each data bit to send in a packet. There are: START (0), byte, parity, STOP (1). When a write is issued, the CLK lane is pulled. When the interval is over, the counter will have been reset to zero, and it can pull the data ane LOW for the start bit immediatly. SInce the release of the CLK lane will be detected as a rising CLK edge, the counter will try to update to one. Therefore place everything one bit higher than expected. The counter should go into two 8-1 multiplexer ICs, This will select the correct bit to be sent. The counter should be disabled after the packet is sent, as an incoming packet will cause it to roll over and cause contention. Therefore when the counter exceeds 11, disable any future counting. 

## Host parity
PS/2 Uses ODD parity, and a 74HC280 should be sufficient to do this. It is added on after the data byte and is computed immediatly on the contents of the outgoing register. When the system attempts to write data, at the moment CLK is pulled LOW, the counter selecting each data bit is reset. During this initial pull-down phase, the card will not be allowed to pull the data bus.
