# Day 1 07/06/2026 1.5 hours

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

# Day 2 08/06/2026 5.7 hours

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


# Day 3 09/06/2026  3.05 hours

Im still working on the selectable fan votlage
still working on putting together a pair of buck converters so I can have a 5V and a 12V rail without needing the user to use external power supplies.

<img width="1695" height="829" alt="image" src="https://github.com/user-attachments/assets/fdb3a163-77c8-4d02-9cee-4219c074ca52" />

Im not using the mosfet recommened by TI since its really expensive, I found an alternative thats substantially cheaper, i do need to make my own footprint which is normally ok,
but this is REALLYY confusing me
<img width="1115" height="841" alt="image" src="https://github.com/user-attachments/assets/ee69fe5d-d6b2-4b34-b563-d07d65ea89a1" />

In the end i just used coordinates to align everything
<img width="598" height="529" alt="image" src="https://github.com/user-attachments/assets/68d8626d-703c-4311-8acb-6b1d7fba0486" />

Since i picked an IC thats super versatile
my 5v converter looks like this
<img width="1615" height="714" alt="image" src="https://github.com/user-attachments/assets/a0b12fb8-1355-47f0-a82e-9ea245601222" />
and my 12v like this
<img width="1651" height="729" alt="image" src="https://github.com/user-attachments/assets/e22e21a3-c4c8-4216-bbe1-5c4bc059390f" />

All that changes is two resistors comming out of FB that determine the voltage output.


Im getting reallyy confused how to get mosfets to do what i want 

<img width="1552" height="664" alt="image" src="https://github.com/user-attachments/assets/f02c0687-7abc-4685-a0bd-f06a7ff06221" />

I was going to have a 3 switch, dip switch ,and then use that to let the use select between 5, 12, and 24v.
and then have each switch control a mosfet

Ignore how awfully messy this is
<img width="1774" height="985" alt="image" src="https://github.com/user-attachments/assets/91265d29-c940-4a14-bea4-f7b58ccde64f" />

for the 5v rail it works
<img width="1700" height="965" alt="image" src="https://github.com/user-attachments/assets/b5443693-263e-4304-ae03-2567c6a3588e" />
but then after that
<img width="1649" height="942" alt="image" src="https://github.com/user-attachments/assets/16dbf417-46a9-4cf8-9349-56a6079d8e6d" />
the 12v or 24v rails will start backfeeding the 5v, and causing the source to be at 12v or 24v, which is higher than the drain at 5v, so current flows

I dont know how to fix this as I have no clue what i need to do to fix it.
So i need to watch youtube video or find some way to learn about them that dont just confuse me 

# 11/06/2026 7.3 hours

## getting on with it, addressing scope creep that happened before the project started
First thing i went and did was look at overcurrent protection for the fan ports, since i have so many, it would add AT LEAST $12 IC cost alone ignoring all the extra passives.
It would also add a bunch of complexity, so i dont need it, bye bye fan  OCP, long live the 12A or 30A fuse protecting the input terminals!

originally i wanted to do the voltage switching via dip switches, jumpers can just be kind of annoying, however i couldnt figure out how to set up an ideal switch model thingy with mosfets, so there would have been all sort of backcurrent issues. So i will do it the way that ive seen other boards do it, with jumpers. Im not the biggest fan of the jumper method since at high currents im semi worried about melting things, but I guess its "fine", its still a nice feature.
How many people use their JST-XH ports at 3A anyway?

Here is one of MANY attempts to get something to work in [Faldstad](https://www.falstad.com/circuit/), they often would have weird issues like the output volage being lower than I expected, with the cause of the voltage drop really not being clear. 
<img width="775" height="787" alt="image" src="https://github.com/user-attachments/assets/ae8725ed-fc5c-464a-85fb-19bc25cfb238" />


Still staying on the theme of fans, I need to pick mosfets for them, I think i will use the same mosfets that i used for the buck converters, massively overspecced , I DO NOT need 45A at 30V, however they are compact and cheap at ~$0.089 per mosfet,so still a perfectly reasonable choice . Yipeee ive avoided adding another part to the BOM.

So an indiviual fan circuit looks like this,
<img width="504" height="501" alt="image" src="https://github.com/user-attachments/assets/4295368a-42f5-4154-96ea-fa1ccc5511d6" />

And 12 of them like this.
<img width="454" height="1348" alt="image" src="https://github.com/user-attachments/assets/2396265c-72a3-406d-a004-2466f2a2193f" />



I chose to rename the FanH to Heaters1-3, it just felt correct give the power the mosfets are capable off. As the mosfets are capable of 45A, there will be a fuse rated at ~30A to protect the board and terminals.
<img width="1836" height="532" alt="image" src="https://github.com/user-attachments/assets/3289518e-b57b-426d-940e-7bc22f9eca3d" />


Ive also ditched overcurrent protection for the thermistors, I have added ESD protection in the form of an ESD diode, and some resistors to hopefully stop reduce how nasty any ESD getting to the op amp is.
<img width="1222" height="1502" alt="image" src="https://github.com/user-attachments/assets/5e497d27-92cb-48d0-a680-f0f9dd94bef7" />
I picked the MCP6004 as the op amp for this board, its compact , low power, cheap, and more than good enough for this application.

Thats all the thermistor stuff done!
<img width="2793" height="675" alt="image" src="https://github.com/user-attachments/assets/af0cc0fe-21ab-4b73-ac91-bab04aa21205" />

I did also shuffle arround all of the pins the thermistors were connected to because I forgot to check the pins had an ADC attached to them.
<img width="996" height="1223" alt="image" src="https://github.com/user-attachments/assets/4f974cfe-52a5-4da8-824c-27f34aaa663a" />


which means our list of things to do now looks like this
- [x] pick an MCU
- [x] thermistor op amps
- [x] sort out how im doing to do selectable voltage for the fans
- [x] pick mosfets for the fans
- [x] start on the schematic
- [ ] finish off schematics
- [ ] tidy up schematics
- [ ] bom optimisation
- [ ] routing

And the overall schematic now looks like this
<img width="2405" height="1689" alt="image" src="https://github.com/user-attachments/assets/27d8d0a8-7d93-45ca-9456-f6f877cc6092" />
the "tidy up schematics" box probably makes more sense now...


# 17/06/2026 lots of things comming together - 6 hours

Ive been findning and making footprints for a bunch of components which means that the PCB currently looks like this, an unrouted glob of components.
<img width="1129" height="766" alt="image" src="https://github.com/user-attachments/assets/e79f4f35-fdbd-4415-bfee-14577bea9517" />

The current issue im trying to solve is making the caps for the voltage converters both cheap and preformant.
Its just taking a chunk of time beacuse im also making simulations of the circuit and doing the calcuations specified in the data sheet of the bucks to try and work out how much voltage ripple the caps need to be able to withstand.



<img width="1156" height="1301" alt="image" src="https://github.com/user-attachments/assets/1deee8cb-a043-4d91-881f-5d0b050979e2" />
<img width="1494" height="1138" alt="image" src="https://github.com/user-attachments/assets/6cfd13ab-e2fd-4a14-a0c4-cc5bff00dd25" />
Just going for what the dataseet suggest that i would need, the default caps are arroundd $1.03 each, and the only replacements for them at the same specs are arround $0.90 each.
Once again my solution for this has been just replace the big cap with two smaller caps each with an ESR no more than twice of the original require ESR, with this method I think ive got those caps down to arround $0.12 per pair, which is cerainly a big improvment.
It comes at the cost of space.

Space is the other thing im fighting.
As you can see in the below image, im trying to fit all of the component into the footprint "Cheap JLC limmit" as that would mean the 4 layer PCB would be 100x100mm , which JLC offers really good prices for, and sizes past that incur an engineering fee, which i want to avoid to keep costs low.
<img width="1231" height="900" alt="image" src="https://github.com/user-attachments/assets/7d769f40-1aff-4df6-9322-f858caf83721" />

Im considering removing one or two of the heater since their terminals are really large.
Im also considering switching from the cheap pairs of caps to the singe expensive caps if the footprint saved lets me use the cheaper 100x100mm 4 layer size.

Im also considering putting components on both sides but that has its own issues for ease of assembly






