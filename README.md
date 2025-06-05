# Retro computer board!

A retro computer motherboard for the W65C816S CPU.
Dual-core expandable, 16mb extended memory, four expansion ports.

# Basic Specifications
The primary core is a W65C816S microcontroller, with 32kb of SRAM. EEPROM Space is only accessible to the primary controller. Each core has its own cache, or SRAM chip. Located in the upper-half of the first page, each is 32kb. If the primary controller wants to access EEPROM, I.E. to copy data to another portion of memory, it only needs to access the lower-half. If the primary wants to access the secondary core cache, it uses the expanded memory space. There are two types of busses: core-only and global. The global bus is arbitrated between the two cores. The primary gets priority, then the secondary. In order for the first to access secondary, it must first disable the secondary core. This allows it to always be able to write. The primary core must not access the secondary core too often, as it must be disabled beforehand. If the secondary core needs to access the main bus, it queues to wait before the primary CPU does an internal cycle. At that moment it will take whatever is in the queue registers. This means that the secondary must not access the main memory too often. The secondary cache is only exposed to the global bus if the primary is using it for a transfer. 

## Primary core operation
The core is initiated during a power-on reset phase. The secondary core is disabled during this time, and no secondary transactions are happening. The primary will disable the built-in serial port built in directly to the secondary, and can work without using the secondary core at all. It can use the global bus like the internal core bus.

## Secondary core operation
There are two bits that control the secondary core: ENABLE (0) and RESET (1). The ENABLE bit is used to turn the core on or of. It uses the RDY pin to do this. Enabling this also allows the queue to begin transferring data. The IRQB pin is used for the serial adapter, and the NMIB is used when an external read request has been completed. The RESET pin is used to control the RESB pin on the secondary, to reset it to a known state. It is recommended to disable the core using the ENABLE bit before changing data on the secondary cache. When the primary CPU goes into an internal cycle, it takes whatever value is in the address latches and will attempt a memory transaction. The only accessible area for the secondary is the expansion planes and the expanded memory lanes. The RWB bit will control whether it is reading or writing. Since this all takes a single cycle, timing does not matter. The secondary should not attempt frequent external writes due to speed limitations.

## Built-in serial adapter
The built-in serial adapter is wired directly to the secondary core IRQB pin. If it should be disabled, it can be done so by accessing the internal registers. It uses the R6551 chip. Does not include RS232. When reading data from the serial port, it must use the R6551 as an expansion device, located in the external memory lane. This is so that it can be externally controlled by the primary core. Attempt to read by accessing the global data bus, and wait for an interrupt on the NMIB pin. WAI can be used. The same goes for sending data.

