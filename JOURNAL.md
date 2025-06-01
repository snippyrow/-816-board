---
title: "65816 Retro Motheroard"
author: "snippyrow"
description: "A retro computer motherboard designed for the 65816 CPU."
created_at: "2025-05-24"
---

# May 28th: Began laying out PCB!

For the past few weeks I worked on the schematic, going from a quad-core to a dual-core architecture. I had to cut back and expand on several ideas, though it all seems to work according to the datasheets!
![Snapshot_2025-05-28_17-15-16](https://github.com/user-attachments/assets/2ae250ee-296f-4aa9-8f08-87a8cfb39491)

![Snapshot_2025-05-28_17-15-46](https://github.com/user-attachments/assets/23730b69-aaf7-411c-a7af-059da4089f09)

**Total time spent: ~12h**

# May 29th: Finished laying out PCB

Finished laying out the custom PCB. In the process I removed one of the expansion RAM slots to save on board space, bringing the total amount of expandable memory down to 6MB. Additionally I removed the decoupling capacitors connected to the RAM, as I believe they take up too much space and can be easily added to the individual expansion sticks.
![image](https://github.com/user-attachments/assets/a0c5f4c1-9659-4ebe-b7bc-4e6fede37bdd)
![image](https://github.com/user-attachments/assets/b7a93fac-50e2-421d-9608-881698ac1a8f)

**Total time spent: 3h**

# May 31st: Updated PCB Layout Again..

After checking the design voer, I realized that improvements could be made. Added mounting holes around the corners, as well as the implementation of a serial port directly on the main board. While it does not use RS232, it uses an arduino-compatable 0/5V signal. The 4-pin header comes with a 5V supply pin, allowing a device to expand it out to any type of standard required. Serial connects into the secondary core directly, allowing it to take some of the load off the main CPU. Appears like an expansion device on the BUS.

![image](https://github.com/user-attachments/assets/90689280-ac9b-46c8-8ba1-bec794d0d639)
![image](https://github.com/user-attachments/assets/1ebc73b5-60cf-4b7b-a63a-8d72b632d185)

**Total time spent: 3h**
