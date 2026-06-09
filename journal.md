# Day 1 07/06/2026 x hours

start1 - 22:20 
stop 1 - 22:52 

start2 - 22:55  
stop2 - 23:58

## So im starting this project
My first experience with modifying a klipper board was moddifying the [klipper expander by timmit99](https://github.com/timmit99/Klipper-Expander), to have bigger mosfets as I wanted to attach some chunkier fans that wanted over 4A.

which turned it from 
this
<img width="1553" height="492" alt="image" src="https://github.com/user-attachments/assets/2ce4ca81-3c2d-4fcc-8d53-0970e199ed34" />


to this
<img width="1736" height="541" alt="image" src="https://github.com/user-attachments/assets/24d2374f-6d72-4bb2-a17a-3c26fed86162" />

Same mounting holes, mostly the same circuitry, just bigger mosfets and a type C connector instead of a micro b port.

## time to set out what i want from this

- at LEAST 20 thermistor inputs
- all thermistor input to have overvoltage protection, i dont want to accidentally kill anything with static as i dont know where users will be sticking them
- 12 fan ports with 1-2.5A 
  - all of these fan ports should have a selector to pick between 5v, 12v or 24v/Vbus
- 4 fan parts with ~5A of max current
- USB c in with ESD protection/isolation

I should probably also make a checklist of what i need to get done

- [ ] pick an MCU
- [ ] sort out OVP for the thermistors
- [ ] sort out OCP for the fans
- [ ] sort out how im doing to do selectable voltage for the fans
- [ ] pick mosfets for the fans
- [ ] bom optimisation

## picking an MCU IC
I probably want some sort of STM32 since the klipper support for those is really good,
Time to hunt in the [ST mcu portfolio](https://www.st.com/content/st_com/en/stm32-mcu-developer-zone/mcu-portfolio.html) for something that looks good.

Ive stopped hunting in ST's tool and now im hunting in LCSC, if im buying the part from LCSC, then its useful to check it there so i dont go to find it there and be dissapointed it not there yet. 

Andddd i thought id found chip that was vuagely in the right direction, and realised another constraint i needed to look out for, does klipper support the MCU ? I know i could theoretically try and implament klipper support for a random STM32 chip however i feel like that might be difficult and i dont even know where i would start, so i will leave that as a future me project!

Anyway i found the list of currently supported MCUs for klipper, yipeeeeeee!
<img width="482" height="965" alt="image" src="https://github.com/user-attachments/assets/27bd036d-6aac-4828-b6b3-8500df7fa95b" />

The first MCU I picked wasnt on LCSC
The second one didnt have klipper support
<img width="1499" height="417" alt="image" src="https://github.com/user-attachments/assets/023acac7-f437-4bad-b99d-50177773847a" />
My third MCU seems to be suitable, the [STM32F07ZGT6](https://www.lcsc.com/product-detail/C19156.html)

It features:
- an ARM Cortex-M4 cpu core running at 168MHz
- 82 I/Os
- an LQFP-100(14x14) package so i can actually solder it by hand
- 512KB of built in programme storage so i can have klipper on it
- usb 2.0 support
- a built in oscilator
- has klipper support


checking the data sheet, the variant i picked has 24ADC channels which surpasses the minimum number of thermistors i want.
<img width="1175" height="443" alt="image" src="https://github.com/user-attachments/assets/03a2d08b-d5b1-4541-80d2-8dab00df6e7a" />

I could pick a smaller, cheaper STM32 and then use a number of ADS1115 4 channel ADCs via I2C, as [klipper supports those](https://www.klipper3d.org/Config_Reference.html#ads1x1x), however that might add a chunk  of mess to the config, and at ~$2 per IC, its a lot more expensive per thermistor than just buying a bigger STM32

The size (20mm x 20mm) is a tad annoying, but it is what it is, im not sure how else i expect to get so many IO/s in an LQFP package without it being big, only way to make it smaller would be BGA or similar, which i dont feel like dealing with.

So currently it looks like 
- [x] pick an MCU
- [ ] sort out OVP for the thermistors
- [ ] sort out OCP for the fans
- [ ] sort out how im doing to do selectable voltage for the fans
- [ ] pick mosfets for the fans
- [ ] start on the schematic
- [ ] bom optimisation
Onto the next part! I suspect this might be a tad tricky and involve me learning a chunk of new things

spent some time learning about op amps as i kept spotting them in some thermistor circuits.

# Day 2 08/06/2026 x hours

start1 - 11:23 
stop 1 - 12:18

start2 - 12:38
stop2 - 14:21

start3- 16:00
stop3 - 17:37

start4 - 21:19
stop4 22:36

first task of toady

## sort out OVP for the thermistors



First solution ive found is maybe using a TSV diode to protect them, however Im not sure how well that would do, and leakage current could be a BIG issue

Ive spotted is that some other designs use OP amps to buffer their inputs , if an op amp can help reduce reading errors, im all for it.

the OPA320, OPA323 and OPA2320 
<img width="1353" height="561" alt="image" src="https://github.com/user-attachments/assets/8d54323b-b71f-43d8-ace2-8b5b1ece8639" />

If the OPA2320 , is a dual channel version of the OPA320, why is the OPA2320 arround 25% cheaper than the OPA320? Is there something im missing? Im aware of pricing this whole time because if i need to add 20channels worth of something price is going to be important.

Im also just going through all of the ADC temperature sensors klipper supports incase one of them is a value gem compared to an STM32.
No gems, everything was quite pricey.

Anyway back to hunting for op apms and knowledge of how to implament them with OVP

Another op amp im looking at is the MCP6004 and that series

## starting on the schematic

I wanted a break from the op amp stuff, time to do something im a bit more comfortable with, schematics!

<img width="1142" height="570" alt="image" src="https://github.com/user-attachments/assets/1e34d046-45ca-43d7-99f2-8fb85c5d7fe8" />

now im curious if i could do the thermistor op am circuits as a heirarchical sheet to make it a bit less messy, anyway

It also looks like the LQFP144 package i picked can pull a good ammount of power

<img width="973" height="1004" alt="image" src="https://github.com/user-attachments/assets/2cebbfa7-2471-41ac-abc7-e747bb0913ad" />

<img width="1180" height="263" alt="image" src="https://github.com/user-attachments/assets/b1c3d710-eee8-434d-b8cc-30cf64e62964" />

I will probably use a pair of ideal diodes circuits to the OR the usb 5V and 12-24V input for heaters for the drop down for the STM32.
That allows to have power via USB for flashing, but also means i can have most of the power comming from the 24V PSU not the USB bus.

Nope ive decided against that, i will just add a jumper to pick between them, i dont feel like adding $2-4 of components.

trying to pick resistors for the voltage divider is suprisingly tricky
<img width="1520" height="1136" alt="image" src="https://github.com/user-attachments/assets/089b5ec5-25b6-426c-85a3-8ca824fb32a2" />

Anddd there we go high efficiency 3.3v reg, that took a while
<img width="2327" height="926" alt="image" src="https://github.com/user-attachments/assets/af427afc-dc48-40d4-8ed6-98ceb9490f8a" />

I also found another project with similar goals 
that being [prunt3d](https://prunt3d.com/), they used an ESD diode that appears wayy smaller than the one i was going to use ,which is nice, so ill use the [D55V0M1B2WS-7](https://www.lcsc.com/product-detail/C1976275.html) instead of the [ST SMBJ48A-TR](https://www.lcsc.com/product-detail/C133659.html) i was going to use

I did a little bit more work on the 3.3V reg, I may have forgotten a cap, however I added it back in so its fine.
Ive also been picking parts for some things as I go and adding them into my cart in LCSC
<img width="1637" height="671" alt="image" src="https://github.com/user-attachments/assets/04381399-cd52-42d9-977f-4c2e31b27101" />


Currently the schematic looks like this <img width="983" height="1618" alt="image" src="https://github.com/user-attachments/assets/a90a1ac0-5932-4f27-90c4-97078532834a" />

currently the checklist looks like this, same ammount of things need doing, but ive still made good progress on the supporting components.
- [x] pick an MCU
- [ ] sort out OVP for the thermistors
- [ ] sort out OCP for the fans
- [ ] sort out how im doing to do selectable voltage for the fans
- [ ] pick mosfets for the fans
- [ ] start on the schematic
- [ ] bom optimisation

## Selectable voltage for the fans

For some reason i kind of feel like doing more volage regulator stuff

, if I want 5V and 12V options for fans, I probably need to drop down the 24V to that



