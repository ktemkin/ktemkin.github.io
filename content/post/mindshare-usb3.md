---
title: "Corrections to: USB 3.0 Technology: Comprehensive Guide to SuperSpeed USB"
date: 2020-10-04T17:10:55-06:00
draft: false
tags: ['usb3', 'book corrections']
---

The MindShare book [_USB 3.0 Technology: Comprehensive Guide to SuperSpeed USB_](https://www.mindshare.com/Books/Titles/USB_3.0_Technology) (ISBN-13: 978-0983646518) is a pretty decent reference for implementers of USB 3.0 / Gen1;
but is published with a few factual mistakes -- places where their tables incorrectly copy the standard. This notebook post is here to gather the mistakes I've found, so I can report them the publisher all at once.

Issues thus far (page numbers for first edition):

- MindShare is already aware of a collection of Errata, captured [here](https://www.mindshare.com/images/MindShare_USB3.0_Errata_2-21-2014.pdf).
- **p460, Figure 20-27**: The table flips bits 16 and 17; bit 16 indicates support for acting as a 
_Downstream-facing_ participant; and 17 indicates support for acting as an _Upstream-facing_ one.
- **p300, Figure 13-11**: The `Type` field the figure's `LDN` should have bits 7 and 8 set to `1`; the book shows
the values for a `LUP`.
- **p301, Figure 13-12**: The `Type` field the figure's `LUP` should have bits 7 and 8 set to `0`; the book shows
the values for a `LDN`.
