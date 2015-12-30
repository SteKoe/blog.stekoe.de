---
title: "Did you know? Battery Status of external devices are always approximated!"
date: 2013-06-12
tags:
---
While I was playing around with iStatPro, a monitoring tool for Mac OS X, I figured out that the battery status was displayed totally wrong. Mac OS X showed me a percentage of 53% remaining battery power, but iStatPro reported 32%. How comes?

After some investigation I figured out that it is not easy to calculate the remaining power of a battery when you don't know what type of battery is used. What does that mean? Each battery type (NiMh, Alkaline, ...) has its own "discharge curve" which leads to problems when implementing algorithms to calculate the current battery level. Some battery types like Alkaline have a very linear curve whereas the voltage of a NiMh battery drops dramatically at the start and then does not change much until the very end when it fells down drastically. In addition to that some battery types react on even little changes of temperature which may also lead to false information.