---
title: "Directional Coupler"
layout: post
mathjax: true
---

Directional couplers are a common example of a 4-port device. Here we will design and construct a coupled line directional coupler. As the name suggests, this type of coupler consists of no more than two transmission lines traveling close enough together to electromagnetically transfer energy to each other. 

## Specification
Before looking at anything, I'll leave this diagram of a coupled line directional coupler (courtesy of Pozar's) numbered with conventional port numberings. 

<div align="center">
  <img src="\assets\directional_coupler_ports.png" width="400"/>
</div>

The main parameter we're interested in when designing a directional coupler is how much energy it couples -- its coupling. Coupling is the ratio of the input power and output power from the coupled port, expressed in decibels. Confusingly, some authors will use the opposite convention, i.e. making coupling a negative quantity. Fortunately though, there's no ambiguities between the two since you can't couple more energy than you put in. That is, if someone makes reference to a negative coupling factor, then you can freely invert the sign to adjust the convention used, and vice verse. Of course that's assuming they aren't claiming to have discovered free energy. 

For our coupler, we want a coupling that corresponds to something that can be feasibly constructed. Coupling more power requires close spacing between lines, which is harder to manufacture. On the other hand, too weak coupling may be difficult to measure. After some fiddling, 20dB seemed to be a good balance between the two.

A big caveat is that microstrip is not ideal for coupled line couplers. The assumptions used for deriving the design equations rely on things that only hold for stripline. Essentially, since stripline is a homogeneous medium, the propagation velocities for both the even and odd modes are the same. This is very much not true for microstrip. Where this hurts us the most is directivity -- the isolation between the coupled and isolated ports -- and poorer broadband performance. 

## Design
Designing a coupled line directional coupler is straightforward. First we need to determine the even and odd mode impedances using this pair of equations

$$ Z_{0e} = Z_0 \sqrt{\frac{1+C}{1-C}} \quad\text{and}\quad Z_{0o} = Z_0 \sqrt{\frac{1-C}{1+C}}$$

where $$Z_{0e}$$ is the even mode impedance and $$Z_{0o}$$ is the odd mode impedance. $$C$$ is the coupling factor given as a voltage ratio. For a 20dB coupler, this is

$$C = 10^{-20/20} = 0.1$$

Substituting $$C$$ and $$Z_0$$ into above equations, we find $$Z_{0e} = 55.28\Omega$$ and $$Z_{0o} = 45.23\Omega$$. There are equations to determine the line spacing needed from here, but instead we'll just use the calculator in QUCS. 

<div align="center">
  <img src="\assets\coupled_line_calc_qucs.png" width="600"/>
</div>

Note that we have set the length to a quarter wavelength since this will provide the maximum coupling. Also note that the line width is consistent with what we determined for a $$50\Omega$$ line. That's all we need, so let's put it together in QUCS. 

## Simulation
Here's the QUCS schematic. 

<div align="center">
  <img src="\assets\coupled_line_schematic.png" width="600"/>
</div>

The main characteristics we want to check are:
* Coupling: as previously discussed, the ratio of input power to coupled power
* Isolation: the ratio of the input power to the output on the isolated port
* Directivity: the isolation between the coupled and uncoupled ports

I'm not particularly interested in the insertion loss since it will be fairly small with our coupling and we're simulating a lossless line. Since we'll be measuring S-parameters, we'd like to know what S-parameters these correspond to. The appropriate S-parameters are shown below. Although I'm not explicitly writing it, these are indeed assumed to be in decibels.

* Coupling: $$S_{31}$$
* Isolation: $$S_{41}$$
* Directivity: $$S_{31} - S_{41}$$


Note that directivity cannot be directly measured with a normal two port setup. This is why we have it expressed as the difference of coupling and isolation. Directly measuring directivity would require power to be fed into the input port while making a measurement between the coupled and isolated ports, among other things. More formally, directivity is the ratio of the output power on the coupled port to the output power on the isolated port. Thus,

$$D = 20\log(P_3/P_4) = S_{31} - S_{41}$$

where $$P_3$$ is the coupled power and $$P_4$$ is the isolated power. Also note that with S-parameters, coupling and isolation will be negative quantities despite being defined as positive ones. For convenience, I'll be leaving these as is. 

Running an S-parameter sweep on the above quantities (in dB), we get the following graph. Note that I've only simulated 1 to 2GHz. Outside of this range, the performance of the coupler starts to severely degrade. As such, I'll also only be measuring from 1 to 2GHz.

<div align="center">
  <img src="\assets\coupled_line_sim.png"/>
</div>

As we can see, we indeed have 20dB coupling at 1.5GHz as desired. However, this does start to increasingly drop off at both higher and lower frequencies, with it being particularly bad outside of the 1-2GHz range shown here. We have about 25dB isolation at 1.5GHz, and while we'd like to see much better, it certainly isn't terrible. More problematically, the isolation worsens as frequency increases. Finally, the directivity at 1.5GHz is 5dB, which is abysmal. This is to be expected though given our topology, as we previously discussed. Although the performance isn't spectacular, we've still met our minimum spec of 20dB coupling at 1.5GHz. 

## Implementation

Below is my implementation. I opted for the U-shaped configuration since it prevents excess coupling while keeping the geometry simple. Also note that I have mitred out the corners to prevent reflections, as is common practice with planar transmission lines. 

<div align="center">
  <img src="\assets\coupler_circuit.png" width="500"/>
</div>

Probing the coupling ($$S_{31}$$), we see that our coupler is coupling about 23dB. Not bad, but a bit lower than we'd hope. However, the ripple over our operating range appears to be about 1dB, just as we simulated. I suspect some of the lower coupling can be attributed to loss over the through line, which I measured to be around -0.4dB based on a $$S_{21}$$ measurement. Another possibility is over-mitring of some corners blocking or reflecting some power. Regardless, the solution to lower than desired coupling would be to move the coupled lines closer together. This would require rebuilding the entire circuit though, which I didn't have time to do. 

Coupling graph:

<div align="center">
  <img src="\assets\coupler_s31_graph.png"/>
</div>

Interestingly, our isolation ($$S_{41}$$) is a bit better than we predicted, with a measured 27dB vs. the simulated 25dB. Additionally, we only have about 1dB ripple over 1-2GHz compared to the steady 5dB decrease we simulated. Oddly, the isolation fluctuates over our frequency range as well. I don't really have a good explanation for this.

Isolation graph:

<div align="center">
  <img src="\assets\coupler_s41_graph.png"/>
</div>

As mentioned before, we can't directly measure directivity. Rather, we find it by taking the difference of the coupling and isolation. I plotted this in Excel, as shown below.

<div align="center">
  <img src="\assets\coupled_line_directivity.png"/>
</div>

At 1.5GHz, we have about 4.5dB directivity, which is only 0.5dB lower than simulated. Again this is abysmal, but to be expected for microstrip. I suspect the lower directivity is mainly attributable to loss over the line, about 0.4dB as mentioned before. The curve inherits the oddities of the isolation plot, namely the mild improvement after 1.5GHz, peaking at ~1.65GHz and dropping off afterwards. Regardless, the variation over our frequency range is about 3dB, less than the ~5dB predicted.

Overall, I'd say my design is successful. At 1.5GHz, it has very close to our desired coupling while having nearly the directivity we expect and better isolation. If anything else, this is unquestionably a directional coupler. 