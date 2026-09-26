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

# 22/06/2026 AHHH THE ROUTINGGG AND OTHER STUFF 30 hours

So Ive done a lot of stuff since the last journal , but a LOT of it was the same sort of stuff

A LOT reading data sheets like [this one 
](https://www.ti.com/lit/ds/symlink/lm25145.pdf?ts=1782024874461)
I had particular "fun" with that one trying to make sure i followed as much of the layout guideance as possible!
<img width="1483" height="1955" alt="image" src="https://github.com/user-attachments/assets/5c45a262-b513-4251-be5a-ce6be8cec878" />

I had to follow the layout guide because i was doing the layout,
because i finished off virtuially all of the schematic!

Currently the board looks something like this

<img width="1407" height="916" alt="image" src="https://github.com/user-attachments/assets/46c9508a-b079-43f4-aa7e-4b4f818cee45" />

A lot of the routing and layout was done in smaller discrete chunks and then smooshed into the bits of the PCB layout that were allready decided, im not sure if it was the right approach.
That approach caused me to do a lot of 
- laying out and routing a block
- moving the block into place
- moving arround parts in the block to make it fit better
- moving the block into a second better place
- more moving arround parts in the block to make it fill gaps and fit better

which felt pretty inefficient, but on the other hand I dont think the whole board could have been routed at once given the shear ammount of components and design guidelines imposed

One of the big decisions I was toying with was if i should split the PCB into two PCBs.
- a pcb for the high power stuff
- a pcb for the MCU, USB in, Fan ports 

I know if I make it all one board it will be over 100x100mm which will make it far more expensive from someone like JLC PCB
If i make it two boards then I need to deal with joining them together with some sort of connector.

Ultimatly i decided to go against the split board idea simply because i didnt want to add any extra complexity.

The PCBs wont be the cheapest though since i need 2Oz surface copper weight because of the high currents.
They also look like they will be arround 100x130mm
So i recon it will be arround $80 for 5PCBs

The buck converters did take up a good 4-6 hours, I kept realising there was another thing i could do to better optimise the footprint or adhesion to the guidline

This was the first revisision
<img width="1152" height="1047" alt="image" src="https://github.com/user-attachments/assets/22365be1-942a-4c3f-a7db-b3afbfedac46" />

But when you have multiple of those, the shape makes fitting them in quite akward, so i moved arround the caps to make it squarer
<img width="1465" height="752" alt="image" src="https://github.com/user-attachments/assets/c8ef0e11-5509-4fe7-91e8-f7a720304d8d" />

But that still results in dead space, so i wiggled arround some passives to try and gain more room
<img width="972" height="555" alt="image" src="https://github.com/user-attachments/assets/ddd25536-d881-4c13-861e-0c43640b11cc" />
And that worked pretty well!!!

which means our list of things to do now looks like this
- [x] pick an MCU
- [x] thermistor op amps
- [x] sort out how im doing to do selectable voltage for the fans
- [x] pick mosfets for the fans
- [x] start on the schematic
- [ ] finish off schematics
- [X] tidy up schematics
- [ ] bom optimisation
- [ ] routing

Still some routing left, 
There will be BOM optimisation to do as i keep changing 0402 to 0602s for the sake of being able to run traces under them and also sanity when soldering.
If i want a couple of neopixel ports then i will need to add them to the schematic properly

I should also add 
- [ ] A diagram with a proper diagram of the pinout
but that should wait for after ive finished the layout as current im still playing arround with how to organise things.


# 22/06/2026 the last bit of routing is proving to be a bit of a pain 5 hours
<img width="2621" height="1926" alt="image" src="https://github.com/user-attachments/assets/fd0fcb5e-5f98-4941-8933-e14a312b039f" />
<img width="1761" height="1024" alt="image" src="https://github.com/user-attachments/assets/ebc74924-b3e2-46c1-82f6-ac51ae07c2e4" />
trying to route everything is just becomeing more and more of a pain because the traces often connect to the wrong side and in a place that just doesnt make sense.

I recon the soluation to this is abandoing a pin numbering scheme that makes sense, making a list of all pins with ADC and just picking ones that minimise the ammount of crossing over i need.
The crossing over is really annoying as the vias just prove to be in the way again and again.

PA0-7
PB0, PB1
PC0-5
PF3-10

are the 24 GPIO pins i can use to get an ADC connection.

Given there are 24 and I need to use 20 of them, I dont think its going to give me a ton of flexibility to stop traces needing to travel to the bottom of the chip instead of the top, but knowing which i can play with should still help in avoiding some of the crossing over.

Ive chucked a blue box next to each pin i can juggle arorund.
<img width="726" height="973" alt="image" src="https://github.com/user-attachments/assets/3e10502e-2ec8-470e-9ee8-2b5771835a15" />

I will also probably do the same for the PWM outputs
<img width="808" height="942" alt="image" src="https://github.com/user-attachments/assets/665904b7-2c32-4aa0-8544-4d63d0ab1229" />

I ended up getting a list of all the pins, and a list of which pins have an ADC in, and which have a PWM out.
Then taking away the ADC in from the PWM out so i dont accidentally use a pin i want.

I also removed pins where the TIM channel was used by multiple pins, leaving me with that selection of boxes

The part i have routed are much neater, there are still all of the fan PWM inputs to route, so its too early to say how its truely turned out.

I would also throw in a bit of DCR here and there to make DRC more bareable at the end.
It may be a lot of errors, but it was a lot higher :sob: 
<img width="674" height="519" alt="image" src="https://github.com/user-attachments/assets/cbe1db48-1d82-4c3f-a94e-f9f64690ce97" />

Anyway, still lots more routing to do! 


# finishing off the PCB 23/06/2026 -6 hours

This session has been a lot of cleaning things up, such as the routing for everything connecting to the MCU

previously it looked like this
<img width="2621" height="1926" alt="image (5)" src="https://github.com/user-attachments/assets/83d042a0-2106-4252-95c9-78c575fa876c" />
That mess was only going to get worse, I hadnt even get routed half of the fans or any of the heaters.


With everything routed to the MCU it looks like
<img width="2091" height="1512" alt="image (6)" src="https://github.com/user-attachments/assets/f175cbbe-b8d9-4fe1-9fe5-7b9771e167ab" />
which I think is a LOT cleaner

I also spent a while trying to work my way through all of the DRC issues
<img width="689" height="818" alt="image" src="https://github.com/user-attachments/assets/3433e3d1-0404-42aa-824f-2248a934116b" />
that is the highest number of DRC errors ive ever seen.

Some of the issues were harder to sovle and ive still not solved them, just ignored them as I think there is a good chance that its just KiCAD glitching out

such as this scenario
<img width="916" height="82" alt="image (8)" src="https://github.com/user-attachments/assets/69bc076c-cc2d-4399-8871-cb11edab4f04" />
<img width="495" height="417" alt="image (7)" src="https://github.com/user-attachments/assets/4ef8478d-256d-471d-856c-342013bd6bc6" />
<img width="304" height="942" alt="image (9)" src="https://github.com/user-attachments/assets/add99a92-7faf-427f-a35f-04811ce70016" />

the anular for that hole should be 0.125mm , but kicad thinks its 0.0999999 no matter what i change the diameter to.
I needed to change the anual to something larger so whatever PCB fab i pick can make it, and that still didnt fix it.
If my PCB fab can do it, then im happy, kicad can go sulk about it some more or something.

Anyway there were a LOT of DRC issues, it took a while.

My next step is going to be trying to make a bom file with all of LCSC part numbers.

#24/06/2026 AHHHH - arround 4-6 hours

most of this chunkkk of time has been spent picking components on LCSC
<img width="287" height="1056" alt="image" src="https://github.com/user-attachments/assets/80be1f2c-fe6e-4916-a246-8b5c68d1ff7a" />
The bom starts out looking like that, and then I needed to find the component in the circuit and check for any special requirements

For example with the resistors used as a voltage divider for the thermistors i wanted them to be fairly accurate. 
But for some other things like where the resistors were simply there for the purpose of limmitting current I would quite happily pick +-10% if it existed and was cheaper

The caps were a bit tricker, as for some of them , picking a higher voltage rating would make them a lot pricier
for example the 1210 footprint 4.7uF caps needed for the 3.3V voltage regulator,
If you want then in 5V , they cost arround $0.05 each, but if they are exposed to the incomming 24-30V , they need to be rated to something higher like 36V
which then makes them cost $0.23 each

picking the voltage rarting for a cap was also interesting in situations like when the cap is used in a bunch of places, when it was a negligable ammount more i would go for 36V rated caps as that would be fine no matter the voltage input

There was also a bunch of noticing stuff like "uh oh, why is that 0402 cap 10uF" and then proceeding to realised that needs to be bigger to actaully have that much capacitance

or when i realised that the reistors i used to limmit the inrush current into the mosfet gates werent enough, it would cause an inrush higher than what the MCU GPIO pins were rated for.
Which once again led downa rabbit hole of changing out components, re creating the BOM and copying over the stuff that was allready done.

There were also some really annopying things i encountered, such as the voltage regulator i used for 3.3V and my chosen MCU both being out of stock on LCSC.
After some research on LCSC i found an automotive variant of the vreg that was a little more, but a pin for pin replacement.

The MCU was a bit trickier, but ultimatly i found a version with a higher temp rating , other than it was identical to the original MCU.
Well, the price was also more, but changing the MCU would require a chunk more work and re routing, and its not like I chose an MCU that is EOL, so the project should still be recreatable in the future.

That IC bom from the start now looks like this
<img width="720" height="1349" alt="image" src="https://github.com/user-attachments/assets/0644d57e-5a4b-457e-a1b2-a855d6928300" />
An LCSC part num next to every component.


# Start of the build!!!

you will probably notice that the build was uploaded all in one go, thats because when I was assembling the PCB I would have my laptop next to me for KiCad etc, and as wonderful as github is, loosing your progress because you accidentally close the chrome tab before you commit the changes to the .md is reallyyyy annoying , so I journalled everything into Obsidian notes, and then copy pasted to here!



# 06/07/2026 PART ARRIVING!!! 0.5 hours

wow the terminals are big, today was spent sorting the components into piles ready for as soon as the PCBs arrive!

if you're wondering what piles i sort them into
- resistors
	- by value footprint, 
	- if multiple of same value, smallest package first
- capacitors
	- by value footprint, 
	- if multiple of same value, smallest package first
- special SMD stuff
	- stuff like op amps, micro conctrollers, mosfets etc
- through hole stuff



<img width="3000" height="4000" alt="20260706_135630" src="https://github.com/user-attachments/assets/a3ad94e2-4e6f-4d60-b017-04d6974f0f8a" />

# 08/07/2026 PCBs ARRIVED TIME TO SOLDER!!! - 4h

<img width="3000" height="3362" alt="20260708_174204" src="https://github.com/user-attachments/assets/f9725724-e3f9-46df-b610-940ec5e07210" />
<img width="3000" height="4000" alt="20260708_174311" src="https://github.com/user-attachments/assets/d1bb2609-41dd-4a58-989b-43fd4d6d7579" />



I got a decent stencil application, but i think  i accidentally went over the same area twice on the right hand side of the board, so there was a lot of paste there

<img width="4000" height="3000" alt="20260708_211736" src="https://github.com/user-attachments/assets/bb230935-bdfc-4fcd-bc1f-961ddb6de838" />

also made a good start onto placing components!!!

<img width="3000" height="4000" alt="20260708_224349" src="https://github.com/user-attachments/assets/d4ffe230-3277-4960-85e8-1d60ac94c35f" />

oh., and i regret using 0402, i can do it, its just that 0603 is soo muchhh easier, i can do 0603 with my fingers, 0402 and 0201 are just ,needlessly fiddly for a board like this.

# 09/07/2026 still placing new parts on - 3h
so, it says this is a new day in the journal, but urm, its past midnight!!!
since i don't want all the flux in the paste to dry up and make my life harder, I kept placing components!!
and wow there are a lot of components!

<img width="4000" height="3000" alt="20260709_024031" src="https://github.com/user-attachments/assets/d1e40804-6244-4fec-be8b-6ac80ee84944" />


i dont know why i didnt get a photo before i reflowed
but here is a photo post reflow of me trying to cool the board!!

<img width="4000" height="3000" alt="20260709_221442" src="https://github.com/user-attachments/assets/e88773d8-fce7-4ce7-90a8-3fdd47d8f32c" />



once your usb wires start looking like this, you know the project is going to get more intersting.

<img width="3000" height="4000" alt="20260709_225045" src="https://github.com/user-attachments/assets/54abdd5f-a958-45a5-aa3c-dc7b9a29dab3" />


so it turned out that i picked the wrong type of USB-C connector footprint in KiCad, which made for an interesting discovery when trying to solder on the USB C port...


# 10/07/2026 USB + trying to get power on the 3v3 rail - 7 hours

I finished bodging the USB thing, so that should be sorted, not great, but okay.

<img width="3000" height="4000" alt="20260710_235956" src="https://github.com/user-attachments/assets/e424558a-13df-4269-a583-bd0a73e7c839" />


have been trying to identify why im not getting 3.3v
<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/bbae894e-03ed-462e-8ae5-1fe9cf39dbe2" />
<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/9b20e843-572d-44c5-93d8-c8da9461d6f5" />

photos of the area don't make it look awful 

ive got a janky thing instead of the jumper, i dont want to solder the through hole pins for the selection source for the 3.3v reg, as otherwise i wont get as good contact with the hotplate when reflowing, so ive done this
<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/dec2e26b-9075-4a0b-994a-5a0939c4c0d4" />

i still need to remove it, but its not as hard

5v looks clean ish, but im not sure whats causing that nose

<img width="4000" height="3000" alt="20260710_172340" src="https://github.com/user-attachments/assets/e4eb86c6-7f46-4dc8-90a7-9b17020cf0f2" />



and then my 3.3v rail is looking like this

<img width="4000" height="3000" alt="20260710_172649" src="https://github.com/user-attachments/assets/cee046bd-00dc-4ea2-8bae-e310e9d83a17" />


its supposed to be operating at around 2.2MHz so i dont expect anything spicy at this scale, but given im at 1.12V im guessing ive either got a mistake on my resitsors, or the freq selection stuff.

<img width="4000" height="3000" alt="20260710_173040" src="https://github.com/user-attachments/assets/96e05ac8-a17a-4597-a6d4-910966537d84" />

and now im just getting noise

noise that keeps changing
<img width="4000" height="3000" alt="20260710_173043" src="https://github.com/user-attachments/assets/ec028aa8-9686-4f9a-8f88-7ad799d4c3b7" />

anddd i worked out i left a pin floating that should have been grounded...

<img width="3000" height="4000" alt="20260710_180816" src="https://github.com/user-attachments/assets/d6ce3887-0aa1-4249-88c3-3978bdef1798" />


so now that's bodged with a wire under it that i can join to gnd somewhere

so much time spent trying to work out exactly whats gone wrong :sob: 

# 11/07/2026 more trying to fix the 3.3V regulator - 6 hours

I've really been having an experience trying to fix the 3.3V reg, I think the long wire i tried to use before was too long and acting as an atenna at those frequencies or just something was wrong.
Soooo here are my various attempts including
- shorter wire
- shorter wire again
- scrapping of coppuer under the footprint to try and get a bridge between the pin and the copper
- another attempt at that
- even more attempts at that
<img width="1290" height="860" alt="_DSC5476" src="https://github.com/user-attachments/assets/3b70903b-a93d-4860-80ac-c29cbc3af9c1" />
<img width="1379" height="919" alt="_DSC5472" src="https://github.com/user-attachments/assets/af41a1cd-00a5-4e6e-b12a-d202767b90bf" />
<img width="1628" height="1085" alt="_DSC5475" src="https://github.com/user-attachments/assets/38e9c3bb-97f3-4805-87ed-6912d35411cc" />
<img width="837" height="558" alt="_DSC5468" src="https://github.com/user-attachments/assets/af577ef0-431d-4174-a26b-84af2bc71d58" />
<img width="1510" height="1007" alt="_DSC5465" src="https://github.com/user-attachments/assets/894cd194-c236-49b2-b560-f9a776bfb470" />
<img width="1006" height="671" alt="_DSC5449" src="https://github.com/user-attachments/assets/03b917dc-ba52-40f6-9c82-068cde831681" />

the 3.3V reg still doesn't work...

# 22/09/2026 soldering up another board - 3 hours 

Given how much reflowing I did on the original almost fully populated PCB, I thought to give myself the best chance of not just having random dead ICs, I should assemble a board where Ipopulate near the minimum components to test if it works properly.

<img width="2048" height="1366" alt="_DSC6055" src="https://github.com/user-attachments/assets/bc7f1999-0002-44e2-b09a-a73c0a1af459" />

<img width="2048" height="1366" alt="image" src="https://github.com/user-attachments/assets/bf14c7e4-a199-4035-9821-15ee504cd9c7" />



# 23/09/2026  placing the last parts onto the 2nd board and reflowing - 3 hours

i finally got all the parts onto the second board and reflowed all of it. Annoyingly since i waited so long the flux in the past dried out making the sticking of components down to the PCB much harder.

<img width="3000" height="4000" alt="20260924_233933" src="https://github.com/user-attachments/assets/9ca32be3-0d23-45df-80dc-76a9ff2d4607" />

its not as clean as the other
<img width="3000" height="4000" alt="20260926_023415" src="https://github.com/user-attachments/assets/7fe41706-808f-4ab9-8f50-25106d6f8900" />


but took less time to get populated as I skipped a bunch of stuff, and stuff that was harder to solder like the big inductors that sucked to get soldered.
# 24/09/2026 stm32 shaped 2.5W heater - 8 hours

today has been full of "experiences" when trying to get the PCB working

first thing i did that messed me up was soldering on a terminal 90degrees out of alignment...

Getting them to solder down was INCREDIBLY difficult , and getting one out was even harder,
When you have tons of vias, massive uninterrupted power planes, and extra thermal mass and conductivity from having of 2oz external copper, and 1oz internal copper , it make the experience even more "fun". I think it took a good 4min of dual wielding a hot air gun and a soldering iron to get it out. And "getting it out" meant getting it to drop around 2mm, so i could use side cutter to get it out because it did NOT want to come out.


here is a photo of the poor state that PCB was in :(
<img width="3000" height="4000" alt="20260924_214917" src="https://github.com/user-attachments/assets/18c057a3-2484-4b5e-b291-fdaab0e793b9" />
<img width="3000" height="4000" alt="20260924_215209" src="https://github.com/user-attachments/assets/097baad9-e10d-4ea5-8ebe-7cbbfb1a71b5" />


But in the end i got it out!!!

here is a photo of how the PCB looks now

<img width="3000" height="4000" alt="20260924_220310 2" src="https://github.com/user-attachments/assets/bb00bd47-c8da-4def-afb7-f102c8366f22" />

which got me back to having a populated PCB!!


#### This is the part of todays journal where I explain the title being "stm32 shaped 2.5W heater"

So at this point I have
- a virtually fully populated PCB with a potentially heat damaged STM32
- a functionally populated board that shouldn't have any heat damaged components

So I get some things together
<img width="4000" height="3000" alt="20260924_230330" src="https://github.com/user-attachments/assets/04783589-306c-4a73-949a-924ca0beed3a" />

- multi meter because the owon P4305 has an amperage accuracy of arround +-10ma, if a healthy STM32 pulls 5-10mA, and a dead one pulls 0mA, then i cant tell the difference without the multimeter
- modern linear PSU that i trust to actually output the right voltage
	- using this for the 3.3V since ive had so many issues with it and its killed so much of my time
- 2nd PSU to do the 12-24V input


I hook everything up as you'd expect to the semi populated board as i thought id have a better chance getting something out of that one. I turn on the 3.3V psu and instantly hit the overcurrent protection, what on earth is pulling 1.5A of 3.3V?

So I grab a multimeter and measure the resistance between GND and the 3.3V rail, ~1Ohm, not normal

some extra attentive checking for shorted pads on the stm32 later and nothing looks wrong, but ive touched some pads up 
<img width="3190" height="1859" alt="_DSC6074" src="https://github.com/user-attachments/assets/0f51a042-9666-4646-9458-7751781a7e20" />
<img width="3381" height="1838" alt="_DSC6077" src="https://github.com/user-attachments/assets/b387ef94-4400-4713-9b1c-4b338e88a6b8" />
<img width="3511" height="2712" alt="_DSC6076" src="https://github.com/user-attachments/assets/beac310e-e988-474b-826c-60936b2bf0f7" />
<img width="3069" height="1898" alt="_DSC6075" src="https://github.com/user-attachments/assets/0e72c598-495e-4780-b926-c3cb0e3a649c" />

Try 2 , ***beeeep***, overcurrent protection again...
but nothing is catching fire, nothing is going bang etc, so given its a semi populated PCB, i thought id turn the overcurrent protection off, and just run up the current limit until i could feel a component getting warm. 

100mA, nothing,
200mA nothing, 
1.5A limit, YUP THE STM32 IS GETTING WARM FAST

The heat was coming from the package so i presume id probably managed to damage it with heat.

It being the fault of the STM32 was confirmed when i desoldered it an my GND-3.3V resistance dropped to 5.6kOhms

a replacement STM32 got that current down from 1.5A to ~7mA, which makes sense 

<img width="4000" height="3000" alt="Pasted image 20260926011845" src="https://github.com/user-attachments/assets/c1f0e498-71ab-4569-8975-e2cd428588a3" />



The board I thought would have issues only pulled ~5mA also in the healthy range, so hopefully i should have two boards that just need a flash then should work!


# 25/09/2026 actually getting it working and getting a demo video - 3 hours

well this makes for some fun trouble shooting
<img width="932" height="70" alt="image" src="https://github.com/user-attachments/assets/c0166983-2247-4dba-8959-a7cac7dfa000" />

Pulls the right amount of power as if its a working stm32, but doesn't want to talk to me
<img width="4000" height="3000" alt="20260925_022702" src="https://github.com/user-attachments/assets/b8b67d68-b0a3-45a6-a254-06e42e4d4436" />


Also a fun time to realize that breaking out the SWD pins is normally something that people do to help with troubleshooting.

its also certainly a setup im working with
<img width="4000" height="3000" alt="20260925_215227" src="https://github.com/user-attachments/assets/98320a24-08eb-495d-846c-b8c9e541114c" />

So now i could have any of the following issues:
- issue with my pcb design (unlikely)
- issue with my USB cable "solution" (likely)
- issue with my power supply (unlikely)
- something else ??

Anddddd its communicating!!!

it fell into the  "issue with my pcb design", not as unlikely as I thought
Boot0 was properly set up with a pull down resistor and a switch to pull it up
But Boot1 was left floating, it should have been pulled down...
It was NOT broken out
soldering on the STM32s legs is trickyyyyy...


<img width="844" height="560" alt="image" src="https://github.com/user-attachments/assets/e4db79e0-b427-4123-9311-09857f1e6876" />

fan go spinny spinny  
*insert happy William noises*



# lessons ive learnt from this
- DOUBLE CHECK YOUR FOOTPRINTS
- THERMAL RELIFS ARE IMPORTANT TO MAKE THINGS SOLDERABLE WHEN YOUR PCB HAS 2OZ COPPER
- add holes so I can use a stand to hold the PCB
- make sure to break out the SWD pins an STM32, makes debugging easier
- be suspicious of stuff that is left floating in your schematic
