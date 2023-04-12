+++ 
draft = false
date = 2023-04-11T19:57:46-07:00
title = "Formula Electric Autonomous"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

I joined the Berkeley Formula Electric Autonomous team, so I'll share what we did in Spring 2023:

There are 3 main parts of the autonomous pipeline:
- Perception
- SLAM
- Planning and Control


__Perception__ takes camera and LiDAR data and outputs where obstacles are detected and where they are.
__SLAM__ takes obstacle data from perception and predicts where the car is based on where obstacles are.
__Planning and Control__ takes the location of the car and location of obstacles, and determines the best path to follow.

I joined the Planning and Control team so I will elaborate on PNC:

# Planning and Control (PNC)

PNC can be divided into two tasks:
- Path finding
- Model Predictive control

Path finding finds a path from point A to B
Model Predictive control optimizes the path so that we are driving close to the racing line (optimal path for racing).


## Path Finding

### Why not A*

We did consider algorithms like A* by using delauney triangulization to discritize the space, and then searching through for a path but triangulization did not guarentee the best path would be in the search space so we abondoned the idea.

Here is an example I tested out.

This is an example race track:

![example track](/img/exampletrack.PNG)

Run delauney Triangulization:

![delauney](/img/delauney.PNG)

Remove the middle:

![remove middle](/img/removemiddle.PNG)

Run A*around the track, and apply interpolation.

![final track](/img/finalTrack.PNG)

Notice how if triangulization returned the red line stead of the blue line overlapping the red line, the optimal path would not even be in the search space.

We have no guarentee about the shape of the track, so this uncertainty is not acceptable.

### RRT*

Here is our first version:

{{< youtube id="mXnNdAj1WYo" >}}

RRT stands for Rapidly-Exploring Random Tree.
It is used in non-Descrete search spaces to find paths. It is usually tested in mazes like this:

{{< youtube id="YKiQTJpPFkA" >}}

Here is a good video explaining RRT*:

{{< youtube id="Ob3BIJkQJEw" >}}

We wrote the code in Python, to be later translated into C for performance. 
Unit testing helped a lot in splitting up work and reaching goals.

[Here is the documentation for our RRT algorithm](/posts/formulaelectricdocs/rrt/)

It is heavily based on [this paper](https://drive.google.com/file/d/1isQxfyFiHrueqK4rpEJMvF1BJfzxMYbw/view), 

We still have optimizations for RRT*, like 
- limiting the space we randomly sample points from to in front of the car
- punishing steep angle changes

### MPC

The subteam just finished RRT* so we have not implimented MPC yet, but we are learning the math behind it.

Some papers we are reading right now include:
[EECS127 course reader](https://eecs127.github.io/assets/notes/eecs127_reader.pdf)
[This Tutorial](https://arxiv.org/pdf/2109.11986.pdf)





















3 parts:

perception
SLAM
planning and control
- Path finding
- MPC
