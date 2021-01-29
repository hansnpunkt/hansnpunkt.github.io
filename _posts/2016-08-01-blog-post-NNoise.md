---
title: 'Using Neural Networks for the removal of bad traces'
date: 2016-08-01
permalink: /posts/2016/08/blog-post-NNoise/
tags:
  - neural network
  - noise removal
  - python
---
Using Neural Networks for the removal of bad traces
======

Bigger, faster, beter. Our world might be moving towards bigger, faster, better and so is the industry surrounding geology. At the same time, more and more companies agree that geophysics is the essential tool to either “connect the dots” and/or find the right location for the next dot/drill hole. In particular, big seismic programs on land and water produce a lot of data.

The standard geophone records about 16.000 samples in 2 seconds. Multiplied by the number of geophones, it quickly adds up to millions of data points. One of the first steps of seismic interpretation is to distinguish between good and bad data points. For this, a clear visualization can help geophysicists to recognize a pattern in a picture rather than in a huge matrix. Accordingly, we plot the matrix, each column/geophone/trace next to each other. In the picture below, I have plotted the data of 72 geophones next to each other with the obspy package for python.

![Section plot](https://github.com/hansnpunkt/hansnpunkt.github.io/blob/master/_posts/images/noisy_seismic_dat.png)

The highlighted traces are evidently different from the others. These are the bad boys. For us humans, it is generally easy to spot them. Yet, it quickly becomes a tedious and time-consuming task when one has to pick through hundreds of hundreds of plots. Now, just as we use the visible patterns, modern computer science constructed new tools to analyze huge data sets.

For example, Neural Networks are one way of training an algorithm on test data and making predictions. As the pictures has visible patters, it also inherent attributes that we can use for characterization. These attributes are the input neurons for the neural network. I used seven input neurons, no hidden layer and one output neuron. The seven input neurons comprise six intra-trace characteristics: e.g. trace offset, average trace frequency, highest peak to 2nd highest peak ratio, seismic quality factor Q and one inter-trace attribute, the maximum cross correlation with the two nearest traces. The output neuron is either 0 in the case of a bad traces, or 1 for good traces.

The picture above shows the application I created. The user loads in a seismic section and is asked to select bad traces. User-selected traces are highlighted in red. In the next step, the algorithm calculates the seven input values for every single trace and adds the class value (1 for good trace, 0 for bad trace). Already, one file is often enough to sufficiently train the Neural Network. Once trained, a new seismic file can be opened and analyzed with the existing Neural Network. Now, bad traces are highlighted and at the same time flagged as such in the header of the file. Interestingly, this even works when not all of the bad traces were initially picked; the trained network then surpasses the users choice by detecting all bad traces. However, the user is always able to select or deselect picked traces. 