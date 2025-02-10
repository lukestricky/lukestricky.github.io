---
title: "HT1621 Testing Board"
date: 2024-02-16
categories: [PCB]
tags: [PCB, Embedded, System]
author: Luke
---

## Motivation
For one of the projects, I'm building currently is a compass for my sailboat that tracks and displays various metrics. Since it needs to be easily readable in sunlight, I decided to use a reflexive LCD display. I chose the LCD-S401C71TR, a 4-digit LCD that fits my needs. To drive this display, I needed a HT1621, a small SMD driver chip. Instead of just using an existing board with an lcd on it already, I wanted to design a breakout board for the HT1621. This way, I can learn by doing and understand how it works so I can implement it on the PCB for my compass project.
## Design Requirement
This section outlines the design requirements for the breakout board, focusing on logic and LCD drive voltage separation, external voltage level inputs, and compatibility with the LCD-S401C71TR display. The design should also provide flexibility for testing other displays and implementing voltage dividers for the HT1621 controller.

1. **Separate Inputs for Logic and LCD Drive Voltage**
    - The design must include separate input connections for the logic voltage and LCD drive voltage.
    - This separation ensures proper voltage handling and prevents unnecessary logic level shifting when both voltages are the same.
        
2. **External Ports for Voltage Level Inputs**
    - Dedicated external ports must be available for both logic voltage level input and LCD drive voltage level input.
    - The LCD drive voltage level inputs are necessary for scenarios where the logic voltage matches the LCD voltage, eliminating the need for a logic level shifter.
        
3. **Compatibility with LCD-S401C71TR and Expandability**
    - The design must specifically support the LCD-S401C71TR display.        
    - Any unused pins should remain accessible, allowing the possibility of testing other display modules.

4. **External Bridge Pins for Voltage Dividers**
    - External bridge pins should be included to allow switching between different voltage divider configurations for the HT1621 controller.
    - This feature ensures adaptability for various voltage requirements and enhances usability.
## Designing PCD
The design sheet is shown below for the PCB followed by explanations for all design choices and components, and then an image of the rounded PCB. 
![Desktop View](/assets/img/ht1621TestingBoard/Schematic_HT1621.png){: width="500" height="250" }
### HT1621

Reading over the data sheet, the HT1621 requires pull-up resistors on its four communication lines. According to the data sheet, 47.5k resistors are suitable for both 3.3V and 5V operation. Additionally, 100n capacitors should be placed on the voltage pins for stability. A voltage divider between VDD and VLCD is needed to reduce voltage if VDD is higher than VLCD. The data sheet recommends a 15k resistor for stepping down 5V to 3.3V. This voltage divider will be bypassed through external pins in case the logic voltage matches the drive voltage. The SEG and COMM pins must also be connected to the LCD for proper operation.

### Level Shifter

The level shifter requires connections for both 3.3V and 5V, which are the two logic levels it will be shifting. It also requires pull-up resistors; however, the pull-up resistors for the 5V line were disregarded since the HT1621 already has them in place. For the 3.3V lines an equation was used from the data sheet to limit the current into the level shifter:
$$Rpu = (Vpu – 0.35 V) / 0.015 A$$ 
This gave ~200 $\ohm$ for the pull up resistors needed on the 3.3V side of the level shifter. 

### External Connection and Config

To facilitate external connections and configuration, two four-pin headers have been added for 3.3V and 5V data. A four-pin header is also included for power, simplifying connectivity. This includes two ground connections to ensure a common ground when voltage is being supplied from two different sources. Additionally, a two-pin header is provided for extra communication ports from the HT1621 that the LCD does not use. Finally, one three-pin header is included for selecting between the voltage divider and VDD for the HT1621
## Routing
## Using LPKF PCB Machine
The PCB is routed with the intent of being make on a LPKF S104 as I have access to one at school. Some things that need to be done differently than if the intent was to order from JLC PCB are:
1. All vias need be be minimum 0.4mm drill diameter due to tool limitations
2. No clearance less than 0.2mm due to tool limitations
3. Leave clearance of 0.5mm for pours, this is to limit the probability of accidental shorts as well as to use the tools less to limit wear
4. Limited to two-layer boards with through hole plating
![Desktop View](/assets/img/ht1621TestingBoard/PCB_routed.png){: width="500" height="250" }
## Writing Driver
...