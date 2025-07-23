# What happened??
The timings may be too tight with the original design, being forced to act in under 20ns. The fix is that instead of using each VGA pixel cycle to write data and preload, use four cycles to write and four to preload. This is good because it has MUCH longer to act, and I can therefore use a more covnentional 128k x 8 SRAM. It can easily act within 55ns, unlike the 12ns one.

## Advancements?
The first half of the 8-cycle period is for writing data. The multiplexers are set to the addresses in the registers, and a writing state is initialized. This will take upwards of 40 x 4ns, or 160ns. This is well within the allowed time. When powering up the card, the preload register is initialized to 0x00. The preload phase is on the other half, directly controlled by X2. WHen in the preload phase, it is set to read mode, and the address is switched to the counters. At the same time, the latch is enabled, allowing the data to be laoded. Since this is technically happening half way through the byte, everything will be shifted over some. To prevent the slow access time from bugging out some pixels, the data is only committed at the last cycle. Software should be able to counteract this.

## What works?
The signal generation works well, as it is within the allowable timeframe. I suspect that the main cause was the slowness of the 74LS20 chip, though I can't know for sure. The only way to test it for certain is to buy a 74HC/AC20 and plug it in. Overall the timing may be too restrictive for these chips, and in that case it may be worth it to simply convert over to the less harsh design. It may use more board space however, specifically around the control section.

## Extra parts
The previous design used a NAND and a 4-input NAND to control the operation. Instead, Alternate read/write cycles by using X2, which should be fast enough. A 3-input AND gate can be used to time the latching circuit, taking in X0, X1, X2. it would it irrespective of the clock phase.

## Conclusion
The write phase will not collide with the read phase at all, and this allows for better timings. Since each stage has 160ns to act, that is WAY more than enough time to work. The main focus is the preload time, which is arguably the most vital step. It allows three cycles, or 120ns, to read. This can then be latched, and since it is mostly irrespective of clock, takes 40ns. This is good enough. Things will be shifted over by one, since it will technically be loading the previous set of pixels and then displaying them later. Either software can shift, or nothing happens at all. Everything is shifted over by 7-8 pixels, and it should be basically invisible to the user.
