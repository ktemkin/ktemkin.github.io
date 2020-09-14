---
title: "Summary: ECP5 SerDes Out-Of-Band Tx/Rx"
date: 2020-09-14T10:04:21-06:00
draft: false
tags: ['serdes', 'ecp5', 'out-of-band']
---

_This post is a summary of the findings from [my look at using the ECP5 SerDes for USB3 LFPS]({{< relref "serdes-lfps.md" >}}) for quick reference of how to use the ECP5 SerDes semi-documented out-of-band I/O features._

Quick summary:
- In addition to using the ECP5's SerDes pins to drive the SerDes; the SerDes Tx/Rx pins can also be used as Low-Data-Rate (LDR) outputs/inputs while the SerDes is in use. 
- This is intended to allow transmission/receipt of out-of-band signals in addition to normal SerDes
operation; and thus requires a configured SerDes.

#### Documentation

To use this functionality, add the following to the invocation of the SerDES `DCUA` block, replacing `CHX` with your channel number:

name | type | description
:----|:----:|--------------
`CHX_LDR_RX2CORE_SEL` | parameter | Set this to `1` to enable receiving out-of-band data; works in parallel with normal SerDes operation.
`CHX_LDR_RX2CORE` | output | When `CHX_LDR_RX2CORE1_SEL` is set, this carries input value from the SerDes Rx input. Captured post differential input, and thus is single-ended.
||
`CHX_LDR_CORE2TX_SEL` | parameter | Set this to `1` to drive the SerDes Tx output directly; remains in affect until overridden via a _SerDes Client Interface_ register. You most likely want to leave this at `0`.
`CHX_FFC_LDR_CORE2TX_EN` | input | When `CHX_LDR_CORE2TX_SEL` is `0`, this controls whether the SerDes Tx output is driven by the SerDes (`0`) or by the `CHX_LDR_CORE2TX` signal (`1`).
`CHX_LDR_CORE2TX` | input | When this is enabled by either `CHX_LDR_CORE2TX_SEL` or `CHX_FFC_LDR_CORE2TX_EN`, the value provided to this signal is directly routed to the SerDes Tx output.

#### Warning

These descriptions are based on my initial experiments; and should be taken with a grain of salt. :)
