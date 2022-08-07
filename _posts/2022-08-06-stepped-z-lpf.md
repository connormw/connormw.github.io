---
title: "Stepped Impedance Low-pass Filter"
layout: post
mathjax: true
---

Stepped impedance filters are one of the most common microwave components. Yet, I think they're one of the most interesting ones. It's amazing to me how strategically arranging precise lengths of transmission line is so effective. Here I'll discuss my specification, my calculations, and my implementation and measurements. I'm not going to go into great detail on the theory behind why this works, mostly because I don't fully understand it myself yet.  

### Specification
Since I won't be using this for anything in particular, my main priority was to make specifications that will result in something I can feasibly build and measure. The main parameters to specify are the response type, cutoff frequency, the attenuation at a certain frequency above the cutoff. I decided on a Butterworth response, which is a maximally flat passband. My nanoVNA has a max frequency of 3GHz, so I decided that 1.5GHz would be a reasonable cutoff. The attenuation specifies the steepness of the rolloff with steeper rolloffs requiring higher order filters that are more tedious to build. Ultimately, I decided on 20dB attenuation at 2.5GHz because it was within the limits of my VNA and it only requires a 5th order filter. It's subjective, but above 5th order seemed too tedious and below seemed too easy to me. For a Butterworth filter, the order $$N$$ can be calculated as

$$N = \frac{\log{\left(10^{A/10}-1\right)}}{2\log{\left(\omega/\omega_c\right)}}$$

where $$A$$ is the attenuation at $$\omega$$ and $$\omega_c$$ is the cutoff frequency. Decimal values should be rounded up. 

### Design
As the name suggests, the element values for Butterworth filters come from the coefficients Butterworth polynomials. One can manually calculate them, but there's plenty of tables online and in print, so we'll defer to those. For a 5th order Butterworth filter, the normalized prototype values are

$$g_1 = 0.618, \quad g_2 = 1.618, \quad g_3 = 2.000, \quad g_4 = 1.618, \quad g_5 = 0.618$$

Note the symmetry which will simplify calculations and construction. 

Now we must scale these values by our filter impedance ($$50\Omega$$) and cutoff frequency. This is with a pair of equations,

$$C'_k = \frac{C_k}{R_0\omega_c} \quad\textrm{and}\quad L'_k = \frac{L_kR_0}{\omega_c}$$

$$L_1 = 3.28\text{nH}, \quad C_2 = 3.43\text{pF}, \quad L_3 = 10.61\text{nH}, \quad C_4 = 3.43\text{pF}, \quad L_5 = 3.28\text{nH}$$