---
layout: post
title:  "Turning off Macbook Display when using External Display"
date:   2014-05-27 13:27:25
categories: macbook
---


If you want to disable the internal monitor of your Macbook, without closing the lid in a Terminal, type :

```
sudo nvram boot-args="iog=0x0"
```

Then, restart your Macbook connected to the display. While your Macbook is rebooting, close the lid.

If you want to reverse these changes, type in a Terminal:

```
sudo nvram -d boot-args
```
