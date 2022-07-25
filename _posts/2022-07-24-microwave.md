---
title: "Microwave Components - Design, Construction, and Validation"
layout: post
mathjax: true
---

### Introduction
In this project, I designed, simulated, and physically implemented a microwave filter, directional coupler, and power divider. Microwave design has fascinated me for a while now, but I didn't really have any idea how to experiment with it on my own. Recently though, I came across this [video](https://www.youtube.com/watch?v=drwGvATLNaw) on YouTube where the uploader implements several microwave filters using copper tape on half-exposed FR-4 board. Additionally, he demonstrates the use of an open-source circuit simulator, [QUCS](http://qucs.sourceforge.net/), to simulate his designs. I was immediately inspired to take up a similar project on my own. For measurements, I used a [nanoVNA](https://nanovna.com/). In all my designs, I targeted 1.5 GHz. 

Before proceeding, I would like to warn that the results, while not terrible, aren't spectacular either. Given the crudeness of my implementation, this is to be expected. Ultimately, cutting copper tape by hand is not exactly a precise operation. I'm also a novice at soldering and some of my connections probably leave much to be desired in addition to contributing to parasitics. Despite this, I thought this project gave me some valuable insight into the process and difficulties of designing microwave components.

### "Hello World" - 50$$\Omega$$ Microstrip Line
Before doing anything else, I decided it would be a good idea to determine the appropriate width for a 50$$\Omega$$ microstrip line. Proceeding without properly matched terminations would just complicate everything afterwards. There's plenty of resources online describing the various approximations and formulas for calculating microstrip impedance, so I won't go into detail here. I just used QUCS's built in calculator to determine what width of copper tape I needed. 

<div align="center">
  <img src="\assets\microstrip_50ohm_calc.png" />
</div>

The values of interest for characteristic impedance calculations are the dielectric constant ($$\varepsilon_r$$), the substrate height (H), and the trace thickness (T). FR-4's $$\varepsilon_r$$ for 1.5 GHz and H are easily found online. While I couldn't get a manufacturer figure for T, 0.036mm seemed to be common for copper tape. The relative permeability for FR-4 is about 1 and $$H_T$$, the distance between the trace and top ground plane, doesn't really matter unless it's small so I left it at its default large value. The other parameters -- trace conductivity, loss tangent, and roughness -- don't impact $$Z_0$$ so I left them as is. As we can see, we want a trace width of about 3mm for a $$50\Omega$$ line.