
# Retro computer board!
![image](https://github.com/user-attachments/assets/aa68e71b-fe0f-4458-8398-38f3ffe12879)


A retro computer motherboard for the W65C816S CPU.
Dual-core expandable, 16mb extended memory, four expansion ports.

## *WHY* did I make this??
I have always loved building things with my hands, especially electronics things. I also love computers, and always to learn how they work. I believe that this is part of that hobby, building myself a computer system to play around with. I want to develop software, as well as expandable hardware. My end goal is to write a complete opertaing system for this build, and have it run DOOM!

# Basic Specifications
The primary core is a W65C816S microcontroller, with 32kb of SRAM. EEPROM Space is only accessible to the primary controller. Each core has its own cache, or SRAM chip. Located in the upper-half of the first page, each is 32kb. If the primary controller wants to access EEPROM, I.E. to copy data to another portion of memory, it only needs to access the lower-half. If the primary wants to access the secondary core cache, it uses the expanded memory space. There are two types of busses: core-only and global. The global bus is arbitrated between the two cores. The primary gets priority, then the secondary. In order for the first to access secondary, it must first disable the secondary core. This allows it to always be able to write. The primary core must not access the secondary core too often, as it must be disabled beforehand. If the secondary core needs to access the main bus, it queues to wait before the primary CPU does an internal cycle. At that moment it will take whatever is in the queue registers. This means that the secondary must not access the main memory too often. The secondary cache is only exposed to the global bus if the primary is using it for a transfer. 

The system is powerful enough to run a basic operating system, and provides options to expand into a workable computer system.

## Advanced
The system uses two cores, the W65C816S core. It is not a drop-in replacement for the W65C02S, and samples data during the falling edge of the PHI2. Each core starts in a 6502-emulation mode, and then transitions to the native '816 mode. Each core can access up to 16MB of memory and has 16-bit internal registers. The primary core contains 32kb of EEPROM space as well as 32kb of SRAM space. The secondary core is disabled at runtime, and must be re-enabled later. The low 65kb of the memory (2^16 bytes worth) are used to communicate and run simple programs. The 32kb space acts as a sort of cache system for each core, and they can communicate largely based on the upper shared memory. Beyond the low 65kb there are devices and expansion memory. The secondary core is unable to reach the primary core, though the primary core can access the secondary core by an expansion device. The first devices after the local memory are the expansion devices. Not only are the expansion ports physically on the board mapped here, but so is the control register for the secondary core, as well as the built-in serial port controller. 

While the primary core is not actively using the global/shared memory, the secondary core is free to do so. To upload a program or start a second process, the secondary core must first be disabled and reset. Then, the cache is accessed and the program is copied. The reset vector will point to some location in the cache, and it will be re-enabled. **Do not set the reset vector elsewhere.**
As stated previously, when the secondary core needs to access shared memory, whether that be an expansion device or general memory, it latches the address. If reading, it sets a control bit and waits for a NMIB. If writing, it sets the control but the other way and latches the write data. When the write is completed, I.E. once the primary is not active, a NMIB signal is sent signaling that the memory transaction was completed. The WAI (wait for interrupt) can be used. 
The system is based off an expanion-type model, where it encourages expansion devices to be built and used. Some examples could be a VGA card, sound card, or a keyboard/mouse system.

![image](https://github.com/user-attachments/assets/1ce73aea-cf5a-4817-8b8f-1df78f4613dc)

## Primary core operation
The core is initiated during a power-on reset phase. The secondary core is disabled during this time, and no secondary transactions are happening. The primary will disable the built-in serial port built in directly to the secondary, and can work without using the secondary core at all. It can use the global bus like the internal core bus.

## Secondary core operation
There are two bits that control the secondary core: ENABLE (0) and RESET (1). The ENABLE bit is used to turn the core on or of. It uses the RDY pin to do this. Enabling this also allows the queue to begin transferring data. The IRQB pin is used for the serial adapter, and the NMIB is used when an external read request has been completed. The RESET pin is used to control the RESB pin on the secondary, to reset it to a known state. It is recommended to disable the core using the ENABLE bit before changing data on the secondary cache. When the primary CPU goes into an internal cycle, it takes whatever value is in the address latches and will attempt a memory transaction. The only accessible area for the secondary is the expansion planes and the expanded memory lanes. The RWB bit will control whether it is reading or writing. Since this all takes a single cycle, timing does not matter. The secondary should not attempt frequent external writes due to speed limitations. The secondary only has access to up to half the expandable memory space, as the last bit is used internally for deciding whether it is reading the MMU register. A NMI also happens after a write, to tell the core that the write succeeded before overwriting the register again.

## Built-in serial adapter
The built-in serial adapter is wired directly to the secondary core IRQB pin. If it should be disabled, it can be done so by accessing the internal registers. It uses the R6551 chip. Does not include RS232. When reading data from the serial port, it must use the R6551 as an expansion device, located in the external memory lane. This is so that it can be externally controlled by the primary core. Attempt to read by accessing the global data bus, and wait for an interrupt on the NMIB pin. WAI can be used. The same goes for sending data.
The secondary should rely in the WAI instruction to manage external reads and writes, but could also be triggered by an IRQB signal from the serial adapter. Internal control should be programmed to prevent this. NMI has priority, after that an IRQB will be sent. The flow should be as follows: first an IRQB signal is sent, then an attempt to read the device will be followed by a NMIB signal. When a NMIB is sent during the IRQB routine, it will cause a nested interrupt to occur, allowing it to service properly. 
![image](https://github.com/user-attachments/assets/e4ad2c7a-63f3-42f3-82e9-4308a4349ac1)

# Timing Control
Timing control is critical for this system. When PHI2 (the clock) goes from high to low, it indicated the start of a new cycle. VDA/VPA pins change about half-way during the PHI2 low period, and indicate whether the CPU is undergoing an internal cycle. Write and read cycles are all sampled on the falling edge. When VDA/VPA update shortly after the falling edge, the primary core is disconnected from the global bus, and it prepares to do a transaction by updating the address that was latched before. write data is also placed onto the data bus if necessary. On the next falling edge, data is written or data is read. If read, it then sends a NMI to tell the core that it read the data. From there it will read the data back from a specific register. When using it to read, it does not exactly read directly from the memory devices. Instead, it reads from a memory management unit that waits for the primary core to open. The secondary can read the MMU register, located in the upper half of the expandable memory space (bit 23). In order to disconnect the primary, BE is set high, disconnecting the data and address busses. Afterwards, the latches are turned on. 
Immediately after the bank address is pushed to the data bus, all read data begins to enter through either the cache space or the shared memory space. RDY is toggled by the control of the first bit in the control register, and is only sampled and updated on the following falling edge of the PHI2.

## Examples
If the primary wants to upload something to the secondary, I.E. change the task, it will first disable the core and push it into a reset. This prevent erroneous memory accesses. Then, with the cache mapped as an expansion memory device, begins accessing. The primary is **blocked** from accessing the secondary cache if the core is active. The only way for the two to communicate is through the shared global memory tables. After it is completed, re-enable the secondary.

# Expansion Ports
The machine contains four expansion ports, wired directly to the primary. Two of the slots are larger, and each one of the two is either wired to IRQB or NMIB. The other two are simply expansion devices. The larger ones have A0-A3, and the smaller ones have A0-A1 as register addresses. The shared memory on the global bus is beyond the 65K from the two cores.
The controller for the secondary, the controller for the serial port, and just about everything else is connected in through the expansion device encoder to save on space.

# Physical Board Explaination
### Primary Core
The primary core uses the W65C816S, with a 24-bit address bus and an 8-bit data bus. The highest byte of the address bus (A16-A23) is pushed onto the data bus during the low phase of PHI2, and is latched at the rising edge of the clock. The address bus comes stable around the same time. During these times, all the chips may stabilize into selecting the correct hardware for the write. Each core has a 74HC373 and a 74HC245 to moderate the data bus and the banked address busses. The BE pin will set the data bus, address bus and RWB pins to high-Z. The banked address byte is fed into a chip that detects if it is anything >0x00. This means that an external cycle is being attempted. If not, the selection uses A15 to select from either RAM or ROM to read/write from. In an external cycle, the primary is shut off from the rest of the device. During an external cycle, the full address is passed to the global address bus, and the data is either passed through from the data bus directly or read in through an octal buffer. The RWB pin is multiplexed along with the secondary.
The expansion slots are wired directly into the primary core NMIB/IRQB pins, and are pulled high on the board. 

### Secondary Core
The secondary core has much the same setup as the primary core, but has some key differences. It can work independently of the primary through its own cache, but lacks an EEPROM space. In order to perform an external read/write, when it detects an external memory transaction it always writes to a large set of registers. These registers latch A0-A22, as well as whether it wants to read or write. It may not be necessary for it to use the full 16MB address space. The secondary cache lies outside of the range that the secondary can reach. When it is latched, it also sets a D-flipflop into a HIGH state. The state of the first flip-flop is transferred over to the external memory device selector, which can choose which device to enable. It is only active during either an external cycle made by the primary or the secondary waiting to write/read data. Now that the devices have a stable address, when an external cycle happens, it sets the second flip-flop into a HIGH state. This will immediately reset both back to LOW, but sends a NMI to the secondary processor.
During an external cycle, the secondary processor latches are always pushed out to the global bus, as well as any data. If no core is desiring to read from the global data bus, then it may go floating. If the secondary desired to write previously, that old data would be pushed, otherwise nothing holds on to the data bus. Therefore the global data bus is always pulled LOW.
A15 is always pulled LOW, and in order to *read* data coming from the global busses, any address from 0x0000-0x7FFF will work. It simply takes the place of the EEPROM space. On the secondary the cache is in the upper half rather than lower.
The secondary processor has two control bits, ENABLE and RESET. ENABLE will control the processor state, pausing it and disabling the address/data busses. The RESET bit will cause it to reset back to a known state. The secondary processor will have all of its data and address busses disabled during this time. The cache can be accessed by the primary core processor during this time, while it is disabled. *It cannot access while running.* While the address and data busses are in a High-Z state, the cache can be selected as an expansion device, and the address/data/RWB for the primary processor are pushed in. The secondary local data bus are always pulled LOW. The secondary address bus is always matches the primary address bus while not enabled, as a way to drive it.

#### Serial adapter
As mentioned earlier, the serial adapter is wired directly to the secondary core, and works as any other expansion device would. The secondary should either disable the adapter or mask interrupts if it were to be doing something other than serial work.


#### Memory selector
The memory selector works entirely off the global address busses for selection, choosing serial devices or memory expansion cards. Devices are only selected if either there is an external cycle or the secondary is waiting to write data. 

# Datasheets
https://www.westerndesigncenter.com/wdc/documentation/w65c816s.pdf

# BOM
| Item               | Quantity | Price (USD) |
|--------------------|----------|-------------|
| 10K Resistor       | 10       | $1.85       |
| SN74HC373N Latch   | 10       | $9.77       |
| W65C816S6PG-14 MPU | 2        | $24.20      |
| L78L33 Regulator   | 5        | $1.40       |
| 10uF Capacitor     | 5        | $1.15       |
| LED (Red)          | 10       | $2.60       |
| SN74HC04N Inverter | 5        | $6.55       |
| SN74HC21N Gate     | 4        | $2.60       |
| Tactile Switch     | 5        | $1.50       |
| SN74HC245N Transceiver | 10    | $6.22       |
| AT28C256 EEPROM    | 1        | $11.62      |
| 150Ω Resistor      | 4        | $1.44       |
| AS6C62256 SRAM     | 2        | $9.50       |
| 1MΩ Resistor       | 4        | $1.44       |
| SN74HC138N Decoder | 5        | $5.50       |
| 1N914 Diode        | 5        | $0.50       |
| SN74HC10N Gate     | 4        | $2.52       |
| 10kΩ Resistor      | 20       | $0.30       |
| 1kΩ Resistor       | 10       | $0.27       |
| SN74HC32N Gate     | 4        | $2.20       |
| SN74HC688N Comparator | 6     | $8.52       |
| 4.7kΩ Resistor     | 10       | $0.47       |
| SN74HC126N Buffer  | 4        | $2.64       |
| SN74HC00N Gate     | 5        | $3.00       |
| 2N3904 Transistor  | 10       | $0.77       |
| 30pF Capacitor     | 10       | $1.18       |
| 0.1uF Capacitor    | 100      | $4.00       |
| 1MHz Oscillator    | 2        | $6.90       |
| SN74HC08N Gate     | 4        | $2.40       |
| SN74HC74N Flip Flop| 4        | $3.60       |
| 20-pin Socket      | 24       | $3.10       |
| 16-pin Socket      | 20       | $2.42       |
| 14-pin Socket      | 20       | $2.20       |
| 28-pin Socket      | 6        | $3.84       |
| 40-pin Socket      | 4        | $3.68       |

**Total: $141.85**
