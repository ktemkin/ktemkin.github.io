---
title: "Quick look at the Hanmatek HO102S Portable Oscilloscope"
date: 2023-03-28T11:38:39-06:00
tags: ['equipment', 'oscilloscopes', 'wavegens', 'multimeters']
draft: false
---

I've recently been checking out a few options for inexpensive, DMM-form-factor oscilloscopes -- mostly out of a desire to be able to work on hardware from anywhere with just my tiny laptop, a tiny projector, and some minimal equipment.

Options like the [Analog Discovery 2](https://digilent.com/shop/analog-discovery-2-100ms-s-usb-oscilloscope-logic-analyzer-and-variable-power-supply/) are nice; but their readouts take up screen space; and they're often limited bandwidth -- so I thought I'd try out a cheaper option in a DMM form factor. I wound up picking up a [Hanmatek HO102S](https://www.amazon.com/gp/product/B0BCDMQY4J) -- which isn't from a well-known brand, but _is_ cheap enough that purchasing one seemed like a relatively low-risk proposition.

After playing with it a bit, here are some thoughts.

{{<figure src="/post-media/hanmatek-ho102s/closeup.jpeg" 
    alt="close-up of the multimeter's top half, showing a fairly standard oscilloscope screen">}}

## Quick Takes

This whole article is supposed to be quick takes; but given how much I tend to wind up writing, here's a TL;DR section first.

#### Specifications

- Oscilloscope, function generator, and digital multimeter modes
- Two (2) BNC oscilloscope inputs, 100MHz analog bandwidth and 500MSa/S
- One (1) 50Ω BNC function generator output (optional), with an analog bandwidth of 25MHz and an unspecified output sample rate that appears to be ~100MSa/S.
- The standard four DMM banana plug inputs:
	- a <= 10A port for higher current
	- a <= 200mA port for lower currents
	- a <= 600V voltage/resistance/continuity/capacitance/diode-measure port
	- your standard COM input

#### The Good

- The HS102S is _super_ cheap. A 50MHz variant without the function generator option currently costs about $125 on Amazon, which puts this much more in the realm of accessible to beginners. The more expensive 100MHz + function generator variety only brings the price up to about $240.
- The unit provides a fairly capable oscilloscope, function generator, and multimeter, all in one DMM-sized package.
- The unit seems like a capable multimeter, with a full set of basic features and a capacitance measurement mode.
- Rechargeable battery, and USB-C charging (though not power delivery). <3
- The purchase price of the unit includes a small case, two 100MHz 10:1 / 6MHz 1:1 probes, a set of reasonable multimeter probes, and -- for the version with a function generator -- a BNC-to-alligator cable.
- The case and body feel reasonably rugged -- I wouldn't be afraid to e.g. drop this thing from a desk.
- It's _adorable_.

#### The Bad

- The "front panel" controls are a bit clunky to use -- you'll find yourself repeatedly mashing the directional keys in order to adjust the horizontal or vertical scales and offsets.
- The scope is missing a few very-standard features:
	- No **cursor** controls whatsoever.
	- The scope can only be triggered on rising or falling edges; so e.g. no pulse-based triggering.
	- There's no trigger holdoff; so the scope will struggle to trigger on signals with multiple edges, like UART signals.
	- Only a few measurements are available: frequency, period, amplitude, min, max, peak-to-peak, and average. Other common measurements (rise/fall time, positive/negative width, duty cycle, etc) are missing, which can be doubly difficult given there's no cursor functionality.
- The function generator is capable, but the analog bandwidth is on the lower side, and like many lower-end modern function generators, you'll see some quantization due to low bit depth.
- The function generator lacks sweep functions.
- The multimeter's continuity buzzer doesn't latch, which means it can miss rapid connections. This means you can't e.g. sweep it across a row of TQFP pins until you find the connection you're looking for. :(
- The little unit does the unfortunately-standard "beep at you repeatedly if it thinks you left it on, then turn off", which inevitably will trigger while you're using it, or trying to capture values for a review.
- While the scope does have PC connectivity and companion software, I couldn't get it to work. The device seems to offer Mass Storage or HID-connected modes; but the software seems to expect things to be over a virtual serial port. (Perhaps they've linked the wrong software in their listings?)

## Some Details

Some more detailed/review-style discussions follow; but they're by no means comprehensive.

#### As an oscilloscope

All in all, the device seems like a _fairly_ capable 500MSa/s oscilloscope -- more than enough for most simple / day-to-day visualizations; but lacking a few common oscilloscope features that limit the way you can look at complex waveforms. That said, this is definitely a capable unit for the current price point of around $100-$250 USD, depending on options.

{{<figure src="/post-media/hanmatek-ho102s/simple_waves.png" 
    alt="an image of a reasonable enough triangle and square wave">}}

The screen provides a pretty standard -- and thus familiar -- oscilloscope display; but the controls are set up like a multimeter; so the controls definitely take some getting used to:

{{< figure src="/post-media/hanmatek-ho102s/whole_scope.jpeg" 
    alt="an image of a device in the form factor of a digital multimeter, but with a directional pad and a set of buttons, instead of either a DMM or a scope interface">}}

Awkwardly, you have to use the directional keys in order to adjust all of the horizontal and vertical parameters -- so if you're doing more than hitting Autoset, you're going to have to be mashing a bunch of buttons.

The device is limited to 500MSa/S -- on the low-end acceptable for a device that advertises 100MHz of analog bandwidth. As a quick test, I plugged in a 1.5ns-rise pulse generator, in order to see limits of the device's sample rate and analog bandwidth:

{{< figure src="/post-media/hanmatek-ho102s/pulse.png" 
    alt="an image of a square wave displayed on the device's screen, with some ringing around the edges">}}
    
Even without zooming in to the edge, we can see a bit of ringing on the waveform -- caused here by the fact that the scope only has high-Z inputs, without any option for 50R termination. Omitting that is pretty standard on less-expensive scopes, so that's quite forgivable.

If we zoom in a bit, we can see the ringing in more detail -- which actually helps us get some interesting input with which to see the scope's capabilities.

{{< figure src="/post-media/hanmatek-ho102s/pulse_ringing.png" 
    alt="the same square wave, but zoomed way in on its rising edge; which looks to rise in around 3ms">}}

Unfortunately, this scope has limited measurement capabilities and -- as far as I can tell -- _no cursor support_ -- which definitely makes measuring things like rise time a real pain.

The scope does appear to meet its analog and digital input specifications -- as long as you note that at 100MHz, you're only oversampling by around 5x, as opposed to the more-standard 10x oversampling, so high-frequency images won't be as clear.

#### As a function generator

The unit has a built-in, DAC-based arbitrary waveform generator -- though you won't be making much use of the "arbitrary" part for user waveforms, as there doesn't appear to be a way to enter waveforms of your own.

Instead, you get a fairly basic set of waves: sinewaves, square waves, ramps, simple pulses, and "arbitrary". The arbitrary category has a few useful waveforms: sinc waves, short amplitude sweeps, stairs, and bessel functions.

{{< figure src="/post-media/hanmatek-ho102s/ampalt.png" 
    title="a short amplitude sweep, as produced by the HO102S"  alt="a view of a sine wave that gradually increases in amplitude, then resets back to 0V and starts over">}}
    
The fact that the arbitrary functionality includes a short amplitude sweep hints at a major missing feature of the function generator: there's no capability to run amplitude or frequency sweeps, which means you can't use to e.g. get a quick idea of a circuit's frequency response.

Note that like most frequency generators, the HO102S has an output impedance of 50R, so if you plug it into its own inputs, you'll see ringing due to the impedance mismatch. This makes it even more of a pity that the scope doesn't include a 50R-input mode.


#### As a multimeter

The HO102S also features a pretty basic autoranging, 4.1-digit digital multimeter -- capable of the typical voltage and current measurements, continuity, resistance, diode-checking -- and, surprisingly, reasonably decent capacitor measurement.

{{< figure src="/post-media/hanmatek-ho102s/multimeter.jpeg" 
    title="measuring a little 2.5000 reference with the HO102S"  alt="the device in multimeter mode, advertising a voltage reading of 2.500 volts">}}
    
The primary limitation of the HO102S as a DMM is its relatively low precision, at 4.1 digits -- and while it appeared pretty darned accurate in all tests I did, I'm still suspicious that it may be inaccurate in certain ranges or at different temperatures. (Then again, I'm pretty sure my suspicion is directly driven from a distrust of multi-function units, especially at this price range, so that might just be my bias.)

All in all, the multimeter seems good enough that I wouldn't feel I was missing that much if I were stuck somewhere with it as my only DMM. 

My only real complaint is the lack of a latching continuity buzzer -- which means that you have to hold the probes on the relevant pins for a a fraction of a second before you'll get a beep. This means you can't do the reverse-engineering favorite of rapidly sweeping a probe across a whole row of pins. :( 
