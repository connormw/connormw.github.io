---
title: "Quarter-wave Stub Notch Filter"
layout: post
mathjax: true
---

Another common, yet simple filter is a notch filter constructed from a quarter-wave stub. In juxtaposition to the other designs in this project, I'll be using this as an example of eyeballing a rough design and iteratively tuning it in. 

## Design
From the QUCS line calc, we know that $$\lambda/4$$ for 1.5GHz in our substrate is about 27.2mm. Instead of precisely cutting this, however, we'll roughly cut ~30mm and trim down while checking the filter center frequency with my network analyzer. Since we're starting too long, we expect that the notch frequency will be lower than 1.5GHz. 

The theory behind quarter-wave stub filters is simple. In a quarter wavelength long stub, the waves reflected off the open end of the stub are 180$$\degree$$ out of phase from the forward propagating waves. Thus they cancel each other out, making the stub appear as a short circuit at its operating frequency. 

## Implementation

Here's the circuit with the roughly cut stub. 

<div align="center">
  <img src="\assets\notch_circuit.png" width="500"/>
</div>

As expected, the notch in the filter response is around 1.14GHz, much lower than 1.5GHz.

<div align="center">
  <img src="\assets\notch_filter_untuned.png" width="500"/>
</div>

To tune the filter up, we need to shorten the stub iteratively until we reach $$\lambda/4$$ for 1.5GHz. After a couple of iterations, I was able to get pretty much spot on 1.5GHz. 

<div align="center">
  <img src="\assets\notch_s21_graph.png"/>
</div>

We have a deep null of about -30dB at 1.5GHz as desired. There's some loss above 2GHz, but otherwise this filter has good performance and behaves as intended. My guess is that the loss is either from a poor solder joint on the stub or accidentally cutting part of the main line during construction. 