---
title: "Automatic Start Sequence V2"
date: 2025-02-18
categories: [Embedded System]
tags: [STM32, c++, Embedded System, PCB]
author: Luke
---

# Automated Sailing Race Timer – Project Report

## Introduction  
Last year, I embarked on designing a PCB based sailing start sequence automation tool, but the final outcome did not meet my expectations. The PCB I designed utilized an STM32 microcontroller in a package that had a ground plane underneath the chip, making it extremely difficult to solder without bridging adjacent pins to ground. Since I did not get the board assembled professionally, I had to solder it by hand, and this package proved to be a major challenge.  

Despite numerous attempts, I struggled to reliably solder the chip without causing shorts. While I managed to turn it on multiple times, crucial pins required for operation were often bridged to ground, making the board non-functional. After repeated failures, the chip eventually broke, and since I had only ordered a single unit, I had nothing working to show for my efforts. Additionally, I was unable to determine if my chosen load switch would have functioned correctly for driving the horn.  

From this experience, I learned several valuable lessons that I aimed to apply in Version 2 of this project:  
1. Avoid using difficult to solder components - Whenever possible, choose packages that are easier to solder.  
2. Order extra parts - Always have enough components to build at least two full boards to account for mistakes.  
3. Test before designing a PCB - Using a breadboard prototype to verify functionality before finalizing a PCB design can prevent wasted effort.  

To improve upon Version 1 and ensure a successful second iteration, I planned the following changes:  
- Use an STM32 package without a ground plane for easier soldering.  
- Order 2 - 3 times the necessary components to allow for backup boards and failures. 
- Only use components I have access to for testing so I can first prototype the circuit on a breadboard. 
- Use an IC for controlling the 7-segment display instead of connecting each segment directly to the STM32, reducing the number of required GPIO pins.

To fully understand what I was trying to achieve, Version 1 of the project can be found here: [PUT LINK HERE].  

---

## The Plan  
For Version 2, I aimed to improve the reliability, usability, and assembly process of the device. The key design elements were as follows:  
- 7-segment display with a driver IC to display the time and different operational modes.  
- State machine with two buttons to control the mode and timer operation.  
- Relay module for switching the horn on and off.  
- STM32 microcontroller in a package without a ground plane to simplify soldering.  
- Single PCB design, consolidating all components for durability and ease of assembly. 
- Watertight case design to protect the device from harsh marine conditions, including a sealed battery compartment.  

---

## The Initial Test  
Before proceeding with PCB design, I tested most of the components and connections on a breadboard. This step was essential to ensure that everything worked correctly before committing to a PCB layout. Once I verified that all components I had were functioning as intended, I proceeded to finalize the schematic and PCB design.

---

## State Machine  
To effectively manage different timer sequences, I implemented a state machine with six modes:

| Mode                 | Tag  |
|--------------------------|---------|
| 2-minute start          | 2MS     |
| 2-minute rolling start  | 2RMS    |
| 3-minute start          | 3MS     |
| 3-minute rolling start  | 3RMS    |
| 5-minute start          | 5MS     |
| Manual horn control     | MHC     |

To control these modes with two buttons, I designed a state diagram where transitions between states were triggered by either elapsed time or button presses.  

*The following diagram illustrates state transitions for the 2-minute start and 2-minute rolling start modes:*  
![Desktop View](/assets/img/AutomaticStartSequenceV2/state-diagram.png){: width="500" height="250" }  

---

## Controlling the Seven-Segment Display  
The 7-segment display board I tested used a TM1637 IC to control a four-digit display. Since my testing with this setup was successful, I decided to use it in the final product.

The TM1637 IC has 20 pins in total, including:

- 2 power pins (VDD and GND).
- 8 segment control pins for the 7 segments and a decimal point.
- 6 common pins for each digit (I only needed 4).
- 2 communication pins (CLK and DIO) for controlling the display.
- 2 additional pins (K1 and K2) for keyboard input, which were not used in this project.

When integrating the TM1637 onto the PCB, the datasheet recommended certain design considerations that I followed:

- 10kΩ pull-up resistors on the CLK and DIO pins to ensure signal integrity.
- 100nF capacitor on the VDD pin for decoupling and stable power delivery.

For the display itself, I used a 5461AS-1 7-segment display, which operates at 5V. This created a potential issue since the STM32 operates at 3.3V logic. However, after some research and testing, I found that using pull-up resistors to 5V on the data lines was sufficient to allow the TM1637 to function correctly with the STM32, eliminating the need for additional level shifting components.

---

## Controlling the Horn  
Since I did not have a working load switch, which was my original plan for controlling the horn in Version 1, I decided to keep things simple and use a single relay module that I already had on hand. This relay module had been successfully tested in a previous Arduino prototype, where it reliably switched the horn on and off. However, one issue arose, the relay coil required a 5V supply to activate, while the STM32 operates at 3.3V logic, making direct control unreliable.

To work around this, I incorporated a BC547 NPN transistor, a bipolar junction transistor (BJT), to interface the STM32 with the relay. Using this setup, the STM32's GPIO pin could effectively switch the relay by controlling the transistor. The connections were as follows: 

- GPIO → 2kΩ resistor → Transistor Base 
- Emitter → Ground - Collector → Relay Signal Pin 
- Relay Signal → 5V through a 2kΩ pull-up resistor

With this configuration, when the GPIO pin was low, the pull-up resistor kept the relay active. When the GPIO pin was high, the transistor pulled the relay signal to ground, deactivating it. This approach ensured reliable switching of the horn while allowing the STM32 to control a 5V relay module using 3.3V logic.

---

## Power Management  
The power requirements for this project included 5V for the relay and display, 3.3V for the STM32, and 12V for the horn. To achieve this, I planned to supply ~12V using three 18650 batteries in series, then step it down to 5V using an LDO, and further step down 5V to 3.3V using another LDO. Although I was unable to test this power setup on a breadboard, which went against my initial goal of prototyping all components first, I followed the datasheets carefully and was confident that the regulators would function correctly in the final design. 

I selected two LDOs that were cost-effective, simple to use, and required only two capacitors each for proper operation. The UA78M05CDCYG3 was chosen to step 12V down to 5V, requiring a 0.33µF capacitor on the input and a 100nF capacitor on the output. The AP2114HA-3.3TRG1 was used to step 5V down to 3.3V, requiring 4.7µF capacitors on both the input and output. To connect the 12V battery supply to the board, I opted for a two-pin screw terminal, ensuring easier assembly and maintenance.


---

## The Schematic  
The schematic below outlines all connections, power distribution, and external interfaces. Additional features include:  
- ST-Link header for programming and debugging.  
- Testable LDOs with 0Ω resistors for easy power isolation.  

![Desktop View](/assets/img/AutomaticStartSequenceV2/schematic.png){: width="700" height="350" }

---

## The PCB  
I fabricated the PCB for this project using the LPKF S104, and several considerations had to be taken into account to ensure a successful design. One of the primary factors was tool availability, which directly influenced the minimum feature sizes I could achieve. The smallest drill bit available was 0.4mm, setting the minimum via drill diameter at 0.4mm. Additionally, the smallest routing bit was 0.2mm, meaning that the minimum clearance between traces and pads was also limited to 0.2mm.

Accuracy was another important consideration. While the LPKF S104 is a precise machine, tool wear over time can lead to slight deviations in routing width, which I have seen in previous projects. To mitigate this, I increased trace and pour clearances to 0.4mm from the default 0.254mm wherever possible to ensure reliable manufacturing. I also incorporated multiple vias wherever feasible to improve connectivity and reduce potential failure points.

To further enhance signal integrity and reduce manufacturing time, I implemented a full ground plane on the top layer and a power plane on the bottom layer, with traces strategically cut out. However, to avoid potential interference, I left the area beneath the 7-segment display driver free of power plane pours, ensuring cleaner signals and minimizing noise in the final circuit.

![Desktop View](/assets/img/AutomaticStartSequenceV2/pcb.png){: width="500" height="250" }

---

## The Case  
For this project, I designed a custom enclosure that was 3D printed, with a laser-cut acrylic top to provide both visibility and protection for the internal components. The case was designed to house the PCB directly above three 18650 batteries, which provide power to the system.

A clear acrylic top panel was incorporated into the design to showcase the PCB and allow the 7-segment display to be visible. The one side of the enclosure has three holes to accommodate two control buttons and an on/off switch, while an other side includes additional hole for battery charging access, and horn connection

To securely hold the 18650 batteries, I designed a custom bracket, attached with threaded inserts, ensuring a reliable fit. The PCB is also secured using threaded inserts.

Given the marine environment where the device will be used, waterproofing was a key consideration. The walls of the enclosure were designed to be 3mm thick and printed with high infill settings and setting were optimized to improve layer adhesion. To further enhance waterproofing:

- A gasket made from TPU was printed, allowing it to be compressed and effectively seal the top panel.
- Silicone sealant was applied around the holes to prevent water ingress.

These design choices ensure that the enclosure is durable, functional, and resistant to water exposure, making it well-suited for use in outdoor and the conditions that a coach boat will bring.

![Desktop View](/assets/img/AutomaticStartSequenceV2/case.gif){: width="500" height="250" }
