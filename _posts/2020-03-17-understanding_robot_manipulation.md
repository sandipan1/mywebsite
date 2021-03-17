---
toc: true
layout: post
description: Summary of various aspects of robot manipulation problem
categories: [markdown][robotics]
title: Understanding Robot Manipulation
---


The problems in manipulation require significant interactions between perception, planning and control
> Geometric perception : to understand local geometric of the objects and environment 
> Semantic perception:  to understand what opportunities for manipulation are present in the scene


### Manipulation is more than pick and place :

One application of robot manipulation is "pick and place" .This has been done in factories with carefully crafted parts. 
In recent years these systems use deep learning for perception and work well with more diverse set of objects **especially if the location/placement is not very accurate **

[Southamton Hand Assessment Procedure](http://www.shap.ecs.soton.ac.uk/about-usage.php?task=coins) way to emperically evaluate prosthetics hand 
This might be a good benchmark for robot manipulation 

### Open-world manipulation 
The idea that the world has infinity variability is referred to as "open world" problem .There is a chance that diversity of manipulation in the open world might actually make the problem easier 

### ROS 
With the progress of simulation it is possible to meaningfully study manipulation problem without needing a physical robot. But simulation is not enough .Another challenge is the complexity of autonomy components(perception, planning, control) . ROS(Robot Operating system) help experts from different areas to share their expertise in modular components. The components simply agree on the messages that they will send and receive on the network 

### Position controlled robots 
Given desired joint trajectory the robot executes it with relatively high precision.

Most robots are position controlled and not torque controlled. 

For lightweight arms electric motors are used. 

$ \tau_{motor} = k_ti $

  k_t  is motor torque constant.
However this relationship doesn't hold true when if the gear ratio is high(say >>10) . To achieve reasonable cost and weight , small electric motors with large gear reductions are choosen and this comes with number of dynamic effects that are difficult to model - including backlash, vibration and friction


#### Position control

How to overcome the challenge of not having a good model of transission dynamics ?

If we add sensors to the output side of the transmission, then we can use high gain feedback to directly regulate the joint instead of the motor. 
There is a ==non decreasing== relationship between the current input to the motor and the torque at the joint. 

Position sensor (encoder/potentiometer) are inexpensive, accurate and robust. They can accurate measure joint position and joint velocity by differentiation
> Joint accelerations are generally more noisy and less suitable for tight feedback loops.

A desired trajectory $$ q^d(t) $$ can be tracked using PID control

$ \tau = k_q(q^d - q) + k_d(q^d - q) + k_i \int (q^d - q)dt $

$ k_p, k_d, k_i $ are position, velocity and integral gains. 
> PID gains for each joint are chosen independently and are constant (not gain-scheduled)


