## Reverse engineered schematic parts for the Robomow RM-510 lawn mower 

I am repairing a Robomow RM-510 lawn mower which mowing motor does not works. Test fails with error 55.

*(In the case if you ran into the error 55 issue check the lid close checking Hall sensor...)*

The same board is present in the RM-200 (at least it is possible to set it in the service menu)

The main focus of my work is the mowing motor driving circuit:

![SCH](https://raw.githubusercontent.com/martonmiklos/robomow_rm_510_mainboard_schematic/master/Robomow_PCG5000G1.png)


### IR remote 


IR receiver type is unknown the following is present on it's back:

```
3    
8   K
   O S 
    C

```

* MCU is clocked from a 16 MHz crystal
* IR remote receiver IC OC output is connected to the MCU's P2.5 pin (INPC15)
* Timer1 used for measuring time interupts on the INPC15
* Timer1 has 160 prescaler -> 1 timer increment == 10 us 100 kHz-n jár
* Timer1 CH5 set to measure time on both edges (`MOV.B   #3, g1tmcr5     ; CH5 on IR input measure both edges`)
* Upon IC interrupt:
    * The timer value (G1TM5) stored 
    * The time difference from the last impulse lenght is calculated (current width - prev width)
    * Check if the current is longer than 4560 us + the previous

* Timer1 overflow interrupt generated when the 15th bit overflows (G1BCR0 IT set to 0)
* Timer1 overflow has a timeout counter variable which incremented in the overflows 2 times
    * timer_counter_1,2, ir_timeout_counter counts 0-1-2 (after 0ms - 655,35ms - 1310,7ms)

    
IR protokoll:
- Impulzus == élek között eltelt idő. Le és felszálló ág nem számít.
- Preamble ami áll: több mint egy 11,1 ms-nél hosszabb impulzus
- 36 bit
    - ami 0,81 ms-nél hosszabb és 1,81-nél rövidebb az egyes bit lehet
    - ami 0,81 ms-nél rövidebb az meg a 0-s bit lehet
- a végén legalább 1 db 11,1 ms-nél hosszabb impulzus

- A bejött adatot ellenőrzi:R1H = ((process_state * 4) - 16)
- Az első 12 bit-nek 0x5A5-nek kell lennie
- 4 bit felel meg 1 bitnek, ha 0x5 akkor a kimenet 0 ha 0xA akkor a kimeneti bit meg 1 
