---
layout: post
title: Lenovo Legion 5 pro 16ACH6H 165hz on Linux.
lead: Get 165hz working in hybrid mode Linux
---

**TLDR;**  Use a custom EDID block and load it on startup
{: .message }

- Boot using Hybrid GPU Mode
- Use `xrandr` or `wlr-randr` to figure out the display id. It's mostly `eDP-1`
- Boot using Discrete GPU Mode
- Open `nvidia-settings` -> `GPU 0 -(Nvidia GeForce RTX 3060 Laptop GPU)` -> `DP-4-(CSO)` -> Press `Acquire EDID`
- Now place the `edid` file at `/usr/lib/firmware/edid/edid.bin`
- Add `drm.edid_firmware=eDP-1:edid/edid.bin` to the Kernel Command line
- Note that if your display id in Hybrid mode is not `eDP-1` you will have to change it in the above command
- Now add `/usr/lib/firmware/edid/edid.bin` to your initramfs eg:

```bash
cat /etc/mkinitcpio.conf

#----SNIP----#
FILES=(/lib/firmware/edid/edid.bin)
#----SNIP----#
```