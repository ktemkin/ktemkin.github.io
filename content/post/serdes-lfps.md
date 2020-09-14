---
title: "USB3 LFPS Using ECP5 SerDes I/O"
date: 2020-09-01T11:38:10-06:00
draft: false
tags: ['serdes', 'ecp5', 'usb3', 'lfps']
---

Lately, I've been developing a USB3 stack, and a USB3 frontend using the ECP5 SerDes. The ECP5 SerDes seems _almost_ 
perfect for USB3, which shares many of its implementation details with PCIe. However, there are a couple of hitches. 

The more difficult to deal with is support forUSB3's _Low Frequency Periodic Signaling (LFPS)_. LFPS carries small bursts
of information via a slow, out-of-band signal, meant to be used before the high-speed USB3 link is established:

{{< figure src="/post-media/serdes-lfps/lfps.png" title="from the USB3.2 specification; Figure 6-32" 
    alt="depiction of LFPS bursts over time; including small square wave bursts separated by periods of idle">}}

LFPS data is conveyed by short bursts of square-wave data, followed by periods of electrical idle. The information
carried by each burst is encoded in the total length of each square-wave burst, and in the time period that elapses
between each burst.

#### Transmitting LFPS Bursts

Transmitting these signals with a SerDes is relatively straightforward. Assuming one can bypass the SerDes' typical
encoding, one can fill the SerDes' buffers with long strings of `1`'s and `0`'s, resulting in square waves that can
be recognized by the other side of the USB3 link.

#### Receiving LFPS Bursts, Probably

_Receiving_ these signals, however, can be a real challenge, as they're not in the form the SerDes expects. In addition
to being unencoded and significantly lower frequency, they also lack the frequent transitions necessary for the SerDes
to recover a clock signal.

The [`usb3_pipe` design by Enjoy Digital](https://twitter.com/enjoy_digital/usb3_pipe) works around this problem in an
interesting way: instead of directly detecting the LFPS square waves; the `usb3_pipe` design detects only the periods of
electrical idle; and assumes that the short gaps between periods of electrical idle must be filled with valid LFPS square
waves. From the gaps between periods of electrical idle, it infers (_probably_) the lengths of each burst.

On most FPGAs, this works surprisingly well; and is more than sufficient to detect the basic LFPS pulses necessary to
bootstrap USB3 communcations. On the ECP5, this isn't the case.

The ECP5's _Loss of Signal (LOS)_ detection is documented as follows:

{{< figure src="/post-media/serdes-lfps/los.png" title="Table 78 in TN1621-18; the ECP5 SerDes manual" 
    alt="table indicating that the parameters for the ECP5 SerDes' Loss-of-Signal detection is completely undocumented">}}

In addition to being undocumented to the point where reverse engineering is required for use, the ECP5's _Loss-of-Signal_
detection gives the appearance of being rather overzealous -- routinely detecting loss-of-signal during normal use, no matter
which of these `TBD` values are applied. As a result, using `usb3_pipe`'s LFPS-detection method seems difficult to use reliably
on the ECP5.

#### Receiving LFPS Bursts, Definitely

Fortunately, it seems there's another answer: the ECP5's _SerDes Reference Manual_ alludes to a technique for generating
and capturing simple out-of-band signals:

> There is an input per channel RXD_LDR, low data rate single-ended input from the RX buffer to the FPGA core. In the  core  a  low-speed  Clock  Data  Recovery  (CDR)  block  or  a  Data  Recovery  Unit  (DRU)  can  be  built  using  soft  logic. A channel register bit, RXD_LDR_EN, can enable this data path. If enabled by another register bit, a signal from the FPGA can also enable this in LFE5UM/LFE5UM5G.

> In  the  transmit  direction,  it  is  also  possible  to  use  a  serializer  built  in  soft  logic  in  the  FPGA  core  and  use  the  TXD_LDR pin to send data into the SERDES. It will be muxed in at a point just before the de-emphasis logic near where the regular high-speed SERDES path is muxed with the boundary scan path. This is shown conceptually in Figure 19. The low data rate path can be selected by setting a channel register bit, TX_LDR_EN.

Unfortunately, the reference manual does not document these features further; so I was forced to check the SerDes code generated
by _Lattice Diamond_'s PCIe IP in order to identify how to use these features. Fortunately, the relevant inputs and parameters are easily identified in the generated Verilog:

```
// <snip>
.CH0_LDR_RX2CORE(n101),
.CH1_LDR_RX2CORE(n112),
.CH0_FFC_LDR_CORE2TX_EN(1'b0), 
.CH1_FFC_LDR_CORE2TX_EN(1'b0),
.CH0_LDR_CORE2TX(1'b0), 
.CH1_LDR_CORE2TX(1'b0)

// <snip>
defparam DCU0_inst.CH0_LDR_RX2CORE_SEL = "0b0";
defparam DCU0_inst.CH0_LDR_CORE2TX_SEL = "0b0";
defparam DCU0_inst.CH1_LDR_RX2CORE_SEL = "0b0";
defparam DCU0_inst.CH1_LDR_CORE2TX_SEL = "0b0";
```

With a little bit of experimenting, I was able to come up with the following general configuration for reciept,
which works nicely for capturing LFPS bursts.

```
// Output signal from the module, in Verilog syntax.
// Carries the logic value currently seen at the SerDes inputs, 
// translated to a single-ended value.
.CH0_LDR_RX2CORE(rx_gpio),

// Parameter; enables driving the `rx_gpio` signal.
defparam DCU0_inst.CH0_LDR_RX2CORE_SEL = "0b1";
```

As a bonus, this also shows a path that allows us to drive the SerDes transmitter as though it were a simple GPIO output;
allowing us to send our LFPS signaling without workarounds like squishing long strings of `1`'s or `0`s into the SerDes.
A little more experimenting reveals the following configuration:


```
// Input signals to the module. The CORE2TX_EN signal determines
// when the output is driven by our out-of-band "gpio" -- where `1`
// means "use the out of band signal". With this input set, the 
// CORE2TX signal drives the Tx output buffer directly. When it's 
// not set, the SerDes operates normally.
.CH0_FFC_LDR_CORE2TX_EN(tx_gpio_enable), 
.CH0_LDR_CORE2TX(tx_gpio_value)

// Note that unlike the previous parameter, we want this _SEL to be 0.
// This value appears to defers control to the CORE2TX_EN signal 
// (and likely to a bit in the SerDes registers).
defparam DCU0_inst.CH0_LDR_CORE2TX_SEL = "0b0";
```

With these set up, the SerDes seems to send and receive our out-of-band LFPS signals quite readily:

{{< figure src="/post-media/serdes-lfps/serdes_out_of_band_lfps.png" 
    title="Scope capture of LFPS bursts being transmitted and received" 
    alt="oscilloscope display showing correct-appearing LFPS bursts" >}}

In this diagram:

- The top (yellow) signal shows the tail end of an LFPS burst generated using the SerDes out-of-band I/O; and
- The bottom (blue) signal shows the value of the received out-of-band "GPIO". In this case, I've looped back the
  SerDes Tx port to its Rx; which means that bottom signal is the time-delayed version of the top.

In the end, this is a signficantly nicer way of transmitting and receiving LFPS!
