---
title: "Microwave Components - Design, Construction, and Validation"
layout: post
mathjax: true
---

In this project, I designed, simulated, and physically implemented a microwave filter, directional coupler, and power divider. Microwave design has fascinated me for a while now, but I didn't really have any idea how to experiment with it on my own. Recently though, I came across this [video](https://www.youtube.com/watch?v=drwGvATLNaw) on YouTube where the uploader implements several microwave filters using copper tape on half-exposed FR-4 board. Additionally, he demonstrates the use of an open-source circuit simulator, [QUCS](http://qucs.sourceforge.net/), to simulate his designs. I was immediately inspired to take up a similar project on my own. For measurements, I used a [nanoVNA](https://nanovna.com/). In all my designs, I targeted 1.5 GHz, largely because of the limitations of my nanoVNA. 

Before proceeding, I would like to warn that the results, while not terrible, aren't spectacular either. Given the crudeness of my implementation, this is to be expected. Ultimately, cutting copper tape by hand is not exactly a precise operation. I'm also a novice at soldering and some of my connections probably leave much to be desired in addition to contributing to parasitics. Despite this, I thought this project gave me some valuable insight into the process and difficulties of designing microwave components.

### Directory
This project is much too long and disjoint for a single page, so I've split each part into separate pages. Links for each are below.
* ["Hello World" - 50$$\Omega$$ Microstrip Line][50ohm]
* [Stepped Impedance Low-pass Filter][lpf]
* [Quarter-wave Stub Notch Filter][notch]
* [Wilkinson Power Divider][wpd]
* [Directional Coupler][dc]

[50ohm]: ../50ohm-line
[lpf]: ../stepped-z-lpf
[notch]: ../notch-filter
[wpd]: ../wilkinson-divider
[dc]: ../directional_coupler