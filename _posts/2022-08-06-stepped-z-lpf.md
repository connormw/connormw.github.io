---
title: "Stepped Impedance Low-pass Filter"
layout: post
mathjax: true
---

Stepped impedance filters are one of the most common microwave components. Yet, I think they're one of the most interesting ones. It's amazing to me how strategically arranging precise lengths of transmission line is so effective. Here I'll discuss my specification, my calculations, and my implementation and measurements. I'm not going to go into great detail on the theory behind why this works, mostly because I don't fully understand it myself yet.  

### Specification
Since I won't be using this for anything in particular, my main priority was to make specifications that would result in something I can feasibly build and measure. The main parameters to specify are the response type, cutoff frequency, the attenuation at a certain frequency above the cutoff. I decided on a Butterworth response, which is a maximally flat passband. My nanoVNA has a max frequency of 3GHz, so I decided that 1.5GHz would be a reasonable cutoff. The attenuation specifies the steepness of the rolloff with steeper rolloffs requiring higher order filters that are more tedious to build. Ultimately, I decided on 20dB attenuation at 2.5GHz because it was within the limits of my VNA and it only requires a 5th order filter. It's subjective, but above 5th order seemed too tedious and below seemed too easy to me. For a Butterworth filter, the order $$N$$ can be calculated as

$$N = \frac{\log{\left(10^{A/10}-1\right)}}{2\log{\left(\omega/\omega_c\right)}}$$

where $$A$$ is the attenuation at $$\omega$$ and $$\omega_c$$ is the cutoff frequency. Decimal values should be rounded up. 

### Design
As the name suggests, the element values for Butterworth filters come from the coefficients Butterworth polynomials. One can manually calculate them, but there's plenty of tables online and in print, so we'll defer to those. For a 5th order Butterworth filter, the normalized prototype values are

$$g_1 = 0.618, \quad g_2 = 1.618, \quad g_3 = 2.000, \quad g_4 = 1.618, \quad g_5 = 0.618$$

Note the symmetry which will simplify calculations and construction. 

Now we must scale these values by our filter impedance ($$50\Omega$$) and cutoff frequency. This is done with a pair of equations,

$$C'_k = \frac{C_k}{R_0\omega_c} \quad\textrm{and}\quad L'_k = \frac{L_kR_0}{\omega_c}$$

where $$C'_k$$ and $$L'_k$$ are the scaled values, $$R_0$$ is the filter impedance, and $$C_k$$ or $$L_k$$ correspond to the prototype value $$g_k$$ depending on whether the $$k$$-th element is an inductor or capacitor. We'll be using a ladder topology with series inductors and shunt capacitors. Not only does this produce a low-pass response, it's also easily realizable with transmission lines. This is shown below.

<div align="center">
  <img src="\assets\filter_schematic.png" width="500"/>
</div>

Carrying out the scaling with our circuit in mind, we obtain the following element values. 

$$L'_1 = 3.28\text{nH}, \quad C'_2 = 3.43\text{pF}, \quad L'_3 = 10.61\text{nH}, \quad C'_4 = 3.43\text{pF}, \quad L'_5 = 3.28\text{nH}$$

The key to stepped impedance filters is that an electrically short length of transmission line with a high characteristic impedance is approximately equivalent to a series inductor, and conversely, that an electrically short length of transmission line with a low characteristic impedance is approximately equivalent to a shunt capacitor. 

First, we need to decide upon what we want our high impedance $$Z_H$$ and our low impedance $$Z_L$$ to be. The goal is to maximize the ratio of $$Z_H$$ to $$Z_L$$ while having something actually constructable. For a microstrip line, $$Z_0$$ is inversely proportional to its width in most cases. The maximum width we can have is the width of my copper tape, 50mm. Minimum width is a bit more open-ended, but with my experience and tooling, 1mm is as thin as I can go. Using QUCS's line calculator, we determine that a 50mm microstrip line has $$Z_0 \approx 5.33\Omega$$ and $$Z_0 \approx 85.32\Omega$$ for a 1mm line. Thus we have $$Z_H = 85.32\Omega$$ and $$Z_L = 5.33\Omega$$. 

Next, we must determine the line lengths needed for our required element values. We use a pair of equations, 

$$ \beta l = \frac{LR_0}{Z_H} \quad\text{and}\quad \beta l = \frac{CZ_L}{R_0} $$

obtained from applying the aforementioned line approximations with the previous scaling equations. Note that $$L$$ and $$C$$ are the *normalized* element values, i.e. the $$g_k$$. Applying these, we find that the electrical lengths (in degrees) are

$$\beta l_1 = 20.74\degree, \quad \beta l_2 = 9.88\degree, \quad \beta l_3 = 67.13\degree, \quad \beta l_4 = 9.88\degree, \quad \beta l_5 = 20.74\degree$$

These are mostly fine, but $$\beta l_3$$ does pose a problem. The assumption for "electrically short" is $$\beta l \ll 90\degree$$, and ideally $$\beta l < 45\degree$$. So $$\beta l_3$$ is much longer than we'd hope, but there's really not much we can do about it. The solution would be to use a substrate with a much lower dielectric constant than FR-4, which would increase the wavelengths and make it easier to be electrically short.

To go from electrical to physical length, we need to know the 1.5GHz wavelengths for our $$Z_L$$ and $$Z_H$$ lines. Again using QUCS's line calculator, we find $$\lambda_L \approx 96.36\text{mm}$$ and $$\lambda_H \approx 113.7\text{mm}$$. Thus the required lengths are

$$l_1 = 6.55\text{mm}, \quad l_2 = 2.64\text{mm}, \quad l_3 = 21.20\text{mm}, \quad l_4 = 2.64\text{mm}, \quad l_5 = 6.55\text{mm}$$

With that, we have a complete stepped impedance filter design. Let's move on to simulating it.

### Simulation

Laying out microstrip elements in QUCS is pretty straightforward. First you need to put down a substrate definition with the appropriate parameters. Then you can select the microstrip option from the component library, place them down, and give them our substrate definition and desired length/width. This is my complete schematic. 

<div align="center">
  <img src="\assets\qucs_stepped_impedance.png" width="750"/>
</div>

Note that we are assuming a lossless material as before. Here's the result from simulating the filter's S21. 

<div align="center">
  <img src="\assets\qucs_stepped_impedance_s21.png" width="750"/>
</div>

This is pretty close! We have a nice and flat passband with good roll-off. The only issue is that our cutoff point is around 1.4GHz rather than 1.5GHZ. Our attenuation is also not quite as sharp as we'd hope, with only 19dB at 2.5GHz vs. 20dB, but it's close enough for our purposes. As far as I understand, both of these shortcomings are common to stepped impedance filters compared to their lumped counterparts. The dependence of stepped impedance element values on frequencies poses a challenge. I'm not simulating it, but at higher frequencies, secondary passbands will start to appear as a result, not necessarily periodically though. However, these concerns can be alleviated by keeping everything as short as possible, which is facilitated by using better substrates with lower dielectric constants and thicknesses. 

### Implementation
Now begins the real fun part - actually building and tuning the circuit. After much painstaking cutting and measuring, this is my initial filter.

<div align="center">
  <img src="\assets\stepped_impedance_before_tuning.png" width="500"/>
</div>

Looks good, but let's see what the response looks like on the VNA.

<div align="center">
  <img src="\assets\stepped_impedance_s21_graph.png" width="750"/>
</div>

Yikes... that's certainly not what we expected. The cutoff point (indicated by the marker) is about 900MHz and we're seeing the beginning of a secondary passband at around 2GHz. It took me a while to figure out what's going on here. I actually rebuilt the circuit and the response was identical, so clearly something else was wrong. I believe the issue is that the copper tape itself is introducing huge amounts of parasitic capacitance. In copper tape, the copper layer is very thin and often the adhesive layer is just as thick as it. Essentially the additional separation introduces additional capacitance across the entire surface of the circuit. Indeed this is a very small capacitance, but when we're dealing with picofarads this can be significant enough to cause huge problems. 

So how can we remove extra capacitance? By removing material from the circuit. I focused on trimming down the widths and lengths of the low impedance parts as these are the main capacitive elements in the circuit. This by far produced the most drastic effect on the response. In my fully tuned circuit, the widths are about 25mm and the lengths are about 2mm. After getting the cutoff up to ~1.4GHz, I started lengthening the outer high impedance parts. This had a much smaller effect, and I believe it's since you're also adding slightly more inductance which acts to lower the cutoff frequency. I ended up lengthening either side by ~7mm. Here's my circuit after its makeover.

<div align="center">
  <img src="\assets\stepped_impedance_after_tuning.png" width="500"/>
</div>

And the measured response.

<div align="center">
  <img src="\assets\stepped_impedance_s21_graph_fully_tuned.png"/>
</div>

Much better. The cutoff frequency is almost spot on 1.5GHz now (indicated by marker). Our roll-off is actually now better than expected, with an insertion loss of ~25dB at 2.5GHz. This comes at the expense of having some extra passband loss, however. Additionally, there's some strange rippling as the response rolls off. I really have no idea why this is, but I suspect it's a consequence of drastically adjusting the lengths of elements in my circuit. Regardless though, I would be confident in calling this a 1.5GHz low-pass filter. 