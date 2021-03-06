---
title: 'Engineering a 360° 96MP camera'
date: 2015-08-01
permalink: /posts/2015/08/blog-post-Camera/
tags:
  - CHDK
  - lua
  - parallel computing
---
Engineering a 360° 96MP camera
======

Google sends out their fleet of cars/tricycles/hikers and camels to digitize worlds’ streets and tourist attractions. They even go underwater. So why not go there yourself? Well, I did see some nasty sharks in the pictures and thinking about stinky, dark and confined spaces like pipelines and sewers, it gets even more obvious that I don’t want to go there myself. Let’s better send remote vehicles; Tiny creepy robot-paparazzi.

These days, flying 360° HD ball cameras pop up all over the place. But since the community at CHDK “unleashed the power” of Canon cameras, we can create our own camera array of up to 127 cameras per USB bus. Let’s start:

Step I: Buy 8 12 Canon Powershot D10. 8x12MP=96MP. Ok, it’s not that simple.

Step II:  Setup the SD Card, download the most recent CHDK version and copy it on the card.

Step III: Install libusb (> 0.1.8, Centos 6 works!) and compile ptpcam 2.0 with CHDK extension for your OS.

Step IV: Connect the cameras via USB find out the device names and map each device to a camera.

In linux, the command lsusb lists all connected USB devices and their bus and device names. These change every time a camera is unplugged or restarted, but the serial number is constant. By using lsusb -v and extracting the serial number with a cascade of greps, every camera can be mapped to a single bus:device combination.

Step V: Play around. For example, a camera connected to bus 1, device 3 takes a picture when you send the following commands:

ptpcam --bus=1 --dev=3 ---chdk="mode 1" ## Change to record mode

ptpcam --bus=1 --dev=3 ---chdk="lua shoot()" ## Take a picture

Lua? Yes, Lua. With CHDK, lua scripts can be executed on the camera and we can also send lua commands from the command line. Crucial to synchronizing 8 or more cameras is to simultaneously auto-focus, charge the flash and wait for all cameras to be ready. What do we do when we want the camera to set focus, shutter speed etc. ? We “half-press” and click when the camera is ready. With CHDK, we can simulate any button click, press and release operation. So, let’s half-press until the camera is ready (get_shooting == true), take a picture and release the half-press:

ptpcam --bus=1 --dev=3 --chdk="lua repeat press("shoot_half")" until get_shooting()==true click("shoot_full") release("shoot_half")"

So far so good for one camera, but how do we synchronize 8 or more cameras? For (almost) perfect synchronization you should dive into USB remote control. I want software control and found a way:

Step V: Synchronizing the cameras. Assume that we have more than one camera connected and simulated the half-press on all of them. Each camera might be ready, but one might arrive in this state earlier than the other. How do we tell Camera A that Camera B is ready and vice versa? You’re the head of this operation, so your central computer should decide.
For this, the command putm sends a message to a running script and getm returns a value from a script. So, every camera get’s ready, sends a message and listens to the central unit for the go-ahead. Since I want every camera to take the picture at the same time I used a last little trick to increase synchronization accuracy: The package GNU parallel executes commands in parallel, e.g.:

parallel ptpcam --bus={1} --dev={2} ---chdk={3} ::: bus_array ::: dev_array ::: "mode 1"

This activates record mode on every camera listed in the pre-defined arrays bus_array and dev_array (see Step IV).

Rewind and repeat: Get ready, take a picture, download the last taken picture. With gnu parallel and a lot of testing and optimization I pushed the cycling time of taking a picture and downloading it to 9 seconds at millisecond-accurate synchronization.