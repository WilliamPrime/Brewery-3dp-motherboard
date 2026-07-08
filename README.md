# Brewery-3dp-motherboard
A 3d printer expansion board designed to help with monitoring and managing thermals in a 3d printer

# Key featurs

- supports 12-30V input
-20 buffered thermistor inputs
- 12 fan outputs each with selection between 5V,12V and Vbus
- 12V is dropped down with a buck converter on the board with 12A max
  - if the input voltage is 12V the 12V buck shouldnt be used
- 5V is dropped down with a buck converter on the board with 12A max

- 4 Pwmed outputs meant for heaters or really beefy fans
- 35A max per output, 62A max across all of the ports
  - the heaters are on a seperate fuse and terminal in, allowing you to save power budget for fans or just run things at a different voltage.



So you could you could use the thermistors to do something like 

- 12 for motors
  - would allow for checking in places where you suspect hotspots, but also coldspots.
  - would montior, Extruder, Z, and then 4 motor for an AWD gantry
- temp of outside of pannels
- ambient air temp
- electronics enclosure air temp
- lower printer air temp
- mid printer air temp
- upper printer air temp
- bed edge temp 
- frame temp



# KiCAD photos
<img width="1937" height="1220" alt="image" src="https://github.com/user-attachments/assets/18c0f692-c5e3-4c85-8e75-91d82aeda130" />
<img width="1674" height="1266" alt="image" src="https://github.com/user-attachments/assets/4ae2b7c1-bfaf-4c01-b627-486576411514" />

# Firmware

This board uses an STM32F407ZGT6 MCU, which is supported by mainline klipper and marlin.

# BOM

| Part              | Item cost | shipping+tax cost | Total cost for item(s) | Notes                                 |
|-------------------|-----------|-------------------|------------------------|---------------------------------------|
| LCSC components   | $157.72   | $39.63            | $190.90                | ordering enough to populate the 5PCBs |
| JLC PCB + stencil | $92.09    | $48.06            | $140.15                | No PCBA, im handsoldering, the boards |
|                   |           |                   |                        |                                       |

Total cost $331.05

There is a detailed break down of the parts to be ordered on LCSC [**here**](https://github.com/WilliamPrime/Brewery-3dp-motherboard/blob/main/PROD/Brewery_Alpha_0.01_LCSC%20bom.csv)
