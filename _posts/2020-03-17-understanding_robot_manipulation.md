---
layout: post
description: Summary of various aspects of robot manipulation problem
categories: [markdown]
title: Understanding Robot Manipulation
---


## Problems with Manipulation

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

###  Torque-controlled robots
It is possible to actuate a robot using electric motors that require only a small gear reduction (e.g.  10:1) where the frictional forces are negligible.
We've seen progress in high-torque outrunner and frameless motors bringing in a new generation of low-cost, "quasi-direct-drive" robots: e.g. MIT Cheetah [2], Berkeley Blue, and Halodi Eve. Hydraulic actuators provide another solution for generating large torques without large transmissions.Another approach to torque control is to keep the large gear-ratio motors, but add sensors to directly measure the torque at the joint side of the actuator. This is the approach used by the Kuka iiwa robot that we use in the example throughout this text; the iiwa actuators have strain gauges integrated into the transmission.

## Geometric Pose Estimation

Real depth sensors, of course, are far from ideal -- and errors in depth returns are not simple Gaussian noise, but rather are dependent on the lighting conditions, the surface normals of the object, and the visual material properties of the object, among other things. The color channels are an approximation, too. Even our high-end renderers can only do as well as the geometries, materials and lighting sources that we add to our scene, and it is very hard to capture all of the nuances of a real environment. Real sensor data, and the gaps between modeled sensors and real sensors should be examined. 

### Occlusions and partial views

Cameras don't get to see everything! They only get line of sight. If you want to get points from the back of the mustard bottle, you'll need to move the camera. This is primary challenge with cameras
It is quite common to mount a depth camera on the robot's wrist in order to help with the partial views.But it doesn't completely resolve the problem. All of our depth sensors have both a maximum range and a minimum range. 

### Representations for geometry
The data returned by a depth camera takes the form of an image, where each pixel value is a single number that represents the distance between the camera and the nearest object in the environment along the pixel direction. If we combine this with the basic information about the camera's intrinsic parameters then we can transform this depth image representation into a collection of 3D points.
The conversion of a depth image into a point cloud does lose some information -- specifically the information about the ray that was cast from the camera to arrive at that point. In addition to declaring "there is geometry at this point", the depth image combined with the camera pose also implies that "there is no geometry in the straight line path between camera and this point".

### Point cloud registration with known correspondences
We have a known object somewhere in the robot's workspace, we've obtained a point cloud from our depth cameras. How do we estimate the pose of the object. One very natural and well-studied formulation of this problem comes from the literature on point cloud registration.Given two point clouds, point cloud registration aims to find a rigid transform that optimally aligns the two point clouds. For our purposes, that suggests that our "model" for the object should also take the form of a point cloud

### Iterative Closest Point (ICP)
How do we get the correspondences? In fact, if we were given the pose of the object, then figuring out the correspondences is actually easy: the corresponding point on the model is just the nearest neighbor / closest point to the scene point when they are transformed into a common frame.
This observation suggests a natural iterative scheme, where we start with some initial guess of the object pose and compute the correspondences via closest points, then use those correspondences to update the estimated pose. This is the idea behind iterative closest point (ICP) algorithm.


### Point cloud segmentation
Not all outliers are due to bad returns from the depth camera, very often they are due to other objects in the scene. Even in the most basic case, our object of interest will be resting on a table or in a bin. If we know the geometry of the table/bin a priori, then we can subtract away points from that region. We can use some algorithm to "segment" the scene in a number of possible objects, and run registration independently on each segment. There are numerous geometric approaches to segmentation


## Data driven methods for Bin Picking 
Bin picking has potentially very important applications in industries such as logistics, and there are significantly more refined versions of this problem. For example, we might need to pick only objects from a specific class, and/or place the objects in known position 

> Generating Random Cluttered Scenes
If our goal is to test a diversity of bin picking situations, then the first task is to figure out how to generate diverse simulations. How should we populate the bin full of objects? 
1. Grasp selection
2. Point cloud pre-processing
3. Estimating normals and local curvature
4. Evaluating a candidate grasp
5. Generating grasp candidates
