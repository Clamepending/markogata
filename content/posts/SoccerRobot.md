+++ 
draft = true
date = 2022-12-13T12:58:39-08:00
title = "Robot Soccer Lightweight"
description = ""
slug = ""
authors = []    
tags = ["project"]
categories = []
externalLink = ""
series = []
+++

Sections:
0 Team Orion
1 Inspiration



# 0 Team Orion
In high school I founded Team Orion with 4 of my friends to build autonomous soccer robots. We were National champions in 2019, 2021, and 2022. We participated in the [Lightweight League](INSERT LINK) in 2019 and 2021, and the [Open League](INSERT LINK) in 2022. In the international 2021 competition we represented the United States at the virtual RoboCup2021 Worldwide competition and won the [best video](video link) award.

![2021 Robot](/img/2021Robot.jpg)

{{< youtube id="nJy8fdx0ffE" >}}


We had no mentors. I was in charge of the electrical engineering including PCB design/assembly/testing, Keiji Imai mechanical engineering, Niklas Austermann computer vision, and Danil Korenykh software. Although we had roles, we each contributed outside our areas. For example, I was also involved in the software development and strategy.

[Keiji has an amazing post](https://kogappa.com/projects/rcj_lw/) about the mechanical engineering involved.

# 1 Inspirations (go to section 2 if you want details of our robot)
The following is a list of ideas from teams at the 2019 world cup that we implimented thein 2021. We were the first team to fit all of the following features into our robot under the weight limit (1100g). 
##### I explain the features in the next section

### M&A (australia) - 1st place
{{< youtube id="OaW7RLGctCw" >}}


##### strengths:
speed and reliability

##### features Orion used from the team:
circular line detection PCB and koig algorithm
Maxon DCX19 overvolting
role switching
Orbit function
vision based goalie and striker
Thermoformed mirror (360 vision)


### SkyCrew (JAPAN) - 2nd place
{{< youtube id="nJCUswn8RT8" >}}

##### strengths:
reliability of hardware and software

##### features Orion used from the team:
circular line detection PCB and koig algorithm
role switching
vision based goalie and striker
Thermoformed mirror (360 vision)
kicker

### LEGEND (JAPAN) - 3rd place
{{< youtube id="Pq4eFdwJaSw" >}}

##### strengths:
Speed and kicker

##### features Orion used from the team:
role switching (although their goalie was mostly broken)
vision based goalie and striker
Thermoformed mirror (360 vision)
Dribbler
Kicker
Angled motors


# Features of Orions Robots

{{< youtube id="EiC6TM8G3RY" >}}
### Main PCB
buck converter
lowpass filter
role switching

### Line PCB
circular line detection PCB and koig algorithm
Kicker


### Dribbler
We used a drone motor and esc to spin a custom made silicone roller that makes the ball spin towards the robot. For details, read [Keiji's post](https://kogappa.com/projects/rcj_lw/) 

### Thermoformed mirror (360 vision)
Niklas was mainly in charge of the mirrors and vision system. He heated up a vinyl mirror and pressed it onto a wooden or plastic mold. When the mirror cooled down, it would be in the desired conical shape. Positioning the camera pointing upwards into the mirror allows 360 degree vision with only 1 camera. 

### software strategy
Orbit function
Angled motors vector calcuation
vectorprojection and koig algorithm
Maxon DCX19 overvolting

See our 2021 [Technical paper](https://robocupjuniortc.github.io/soccer-2021/pdfs/TDPs/LWL_Orion.pdf) for more.

### Recommendations for students interested in RoboCup Junior:
Join [Princeton Soccer Robots](LINK) if you can
RoboCupJunior Discord server
RCJ SSL server (be on good behavior)
Great scott youtube (i binge watched)
Digikey Kicad tutorial
twitter



fill in links
check name spelling
The story of PSR
Zircon