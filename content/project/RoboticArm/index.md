---
title: Robotic Arm
summary: Design and fabrication of a Resistive touch controlled manipulator
tags:
  - Microcontrollers
  - C
  - CAD
date: '2022-11-22T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: 
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: GitHub Link
    url: https://github.com/LShivaRudra/RoboticArmWithTouchControl/
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---
| Component Name                             | Use                                                                                     | Quantity  |
|--------------------------------------------|-----------------------------------------------------------------------------------------|-----------|
| Buck Converter                             | Used to step down voltage from 12V to 5V as the operating voltage of the servos is 5-6V | 1         |
| Jumper wires                               | For making connections                                                                  | Multiple  |
| Screws and Nuts                            | For Assembly                                                                            | Multiple  |
| Breadboard                                 | For circuit connections                                                                 | 1         |
| 3D printed parts                           | For Assembly                                                                            | Multiple  |
| 3.5" inch ILI9486 TFT Touch Shield LCD Module 480x320 for Arduino Uno | It is the Resistive touch screen module that we used. It comes along with a stylus.     | 1         |
| Tower Pro MG996R servo motor               | It provides higher torque compared to the smaller servos, so used to actuate larger links like base, waist. | 3         |
| Tower Pro SG90 Servo                       | Smaller servos are used to actuate smaller links like wrist, gripper.                   | 3         |
| 12V AC-DC adapter                          | Used to provide power to the servo motors                                               | 1         |
| 9V batteries                               | Initially used to power the servos, later were replaced with the adapter. Can be used to power Arduino. | 1-3       |

The components:

![1.png](/images/RoboticArm/1.png)

![2.png](/images/RoboticArm/2.png)

The circuit diagram is as follows:

![3.png](/images/RoboticArm/3.png)


The final assembly is as follows:

![4.png](/images/RoboticArm/4.png)

The code can be found out in the GitHub Link of this project.