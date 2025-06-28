# Expansion RAM Module

![image](https://github.com/user-attachments/assets/699f88fe-73ee-48c1-aba6-fd045b23d609)

## What is it?
This is a special module made for my custom motherboard. It can hold exactly 2^21 bits of information, or around 2MB. The motherboard has space for two of these modules, adding up for a total of 4MB. USing the breakout pins, up to 16MB can be added. The board would just be too large.

## How does it work?
The module contains four AS6C4008 SRAM chips. Each one has an access time of about 55 nanoseconds, making it quite fast. Each chip can store 512KB of data, four of them adding up to 2MB. It works virtually the same as the cache memory directly soldered to the motherboard, just larger. The controller system is also quite simple. One first chip is a 74HC138, which selects the correct chip in the module. It uses pins A20-A21 to define which one it needs. The second one controls whether to read or write from the selected chip. as defined by the PHI2 and RWB pins on the expansion slot. It also comes with a large capacitor built into the module. As to prevent random data los due to pwoer fluctuations. While each memory chip has built-in protections, it's best to use extra security if there is enough board space. It also comes with a power LED, telling the user the right orientation and whether it's connected to power.

## How do you use it?
The module expands away from the expansion cards, which are themselves supposed to stretch backwards. That way, they don't hit each other during installation! The edge connector I used provides enough friction to hold the cards in place.

## Extra images
![image](https://github.com/user-attachments/assets/7b5a0751-b5f0-443d-bfda-81a8b71ae658)
![image](https://github.com/user-attachments/assets/507bcd8c-cd09-4106-8845-81582420786c)
