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

# June 7th: Finished Major Re-design

I discovered an issue in my original design, the system that enabled the secondary processor was faulty and would never enable it properly. I also discovered issues with the way the primary CPU writes data to memory, as it would sample data on the wrong half of each cycle. This would cause severe issues. Additionally I decided to change up the design of the system, giving each processor 32kb of cache. This way each co-processor can work fully independantly of the other, only needing to halt in case it needs to access the global memory bus, which exists outside the first 65k. I also wrote a ton of documentation in the README.md file, which extensively explains how the board works.

![image](https://github.com/user-attachments/assets/affd5070-b7a5-4b95-a454-886f6870c5b1)

**Total time spent: 9h**

# June 11th: Half-way there!

I looked over the entire schematic, checking it twice, and began laying out my new PCB. All good things must come to an end, and it doesn't look as nice as the other one yet. I changed it to use mostly through-hole ICs to make it look more "retro", as well as being easier to prototype with.

![image](https://github.com/user-attachments/assets/08cccd33-277d-4958-aa48-284da7f62b94)

**Total time spent: 2h**

# June 13-14th: Almost done..

I spent this time completing the board layout in the kicad designer, as well as checking the schematic again. I found a few small issues and correct them, though I will probably need to check things one more time before completion. I changed it to have only *two* RAM expansion cards, at 2MB each. This leaves it with 4MB of RAM + expansion cards + local cache. I have put everything into an auto-router, which is going as I write this. I think that I will need to spend a lot more time fixing some of the artifacts of the process, though this is *far* faster than doing it manually.

![image](https://github.com/user-attachments/assets/b45435d5-d0d2-401a-a7ff-c939ad0e8692)
![image](https://github.com/user-attachments/assets/ed439c49-b33b-4a7b-b126-771cfc5cfd59)
![image](https://github.com/user-attachments/assets/a8eb8c3a-6c51-49f3-a865-e2f88ed1bd48)

**Total time spent: 4h**

# June 16-18th: FInal checks

I sepent the last bit fixing some minor hardware bugs, as well as completing the PCB layout and tracing. The kicad ERC returns zero errors, so at this point it's pretty much done and ready to run a program. It's been a great journey, and now I just need to do the tedious work of fixing all the autorouting artifcats. This includes right-angled corners and useless trace geometry. I really hope this project works out! The final size is 180x137mm. Unfortunatly checking a project like this for errors takes a lot of time, so more issues may come to light in later revisions.

![image](https://github.com/user-attachments/assets/cdf86245-714a-4a66-a2e1-26cbfe7128cd)
![image](https://github.com/user-attachments/assets/ef8a2141-7cc9-4c85-a4fb-1a2c9eaa64ff)

**Total time spent: 5h**

**June 23rd: A minor upset**
After joining a forum and asking some questions about my design, I realized that I had done some things incorrectly. I was latching data during the falling edge of PHI2, which in hinesight was incorrect. Due to this I had to change some of my labels as well as switching out most 74HC374's with 74HC373's instead. These work better with the regular bus timings. The pins are almost indentical, so it was a quick fix. I also added some stability by making the address decoding logic check the VDA/VPA pins to see if the bus is valid. This will demand a re-layout and new routing run on the PCB, which should not take long. The price for the project should not change much.

![image](https://github.com/user-attachments/assets/2805c379-85f0-4480-981d-174fac76b9cb)

**June 24th: COntinuation**
I completed the board layout and routing process, as well as correcting some minor fixes. The board will soon be production ready, the only thing left is to add more comments to the silkscreen layer.

![image](https://github.com/user-attachments/assets/7948b789-c76a-42ab-8da1-1eb38f204e11)
![image](https://github.com/user-attachments/assets/addd5a5b-dc31-42ec-bc1d-251e52163adf)

