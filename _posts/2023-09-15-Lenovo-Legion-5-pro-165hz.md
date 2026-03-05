---
layout: post
title: Lenovo Legion 5 Pro 16ACH6H — Getting 165 Hz on Linux
lead: Get 165 Hz working in hybrid GPU mode on Linux
---

**TLDR;** Use a custom EDID block and load it at startup.
{: .message }

In hybrid GPU mode on Linux, the display is driven through the iGPU, which often fails to advertise the panel's full 165 Hz capability. The fix is to acquire the correct EDID from the dGPU and force-load it at boot.

- Boot into **Hybrid GPU Mode**
- Use `xrandr` or `wlr-randr` to find your display ID — it is usually `eDP-1`
- Reboot into **Discrete GPU Mode**
- Open `nvidia-settings` → `GPU 0 (Nvidia GeForce RTX 3060 Laptop GPU)` → `DP-4 (CSO)` → click **Acquire EDID**
- Place the acquired EDID file at `/usr/lib/firmware/edid/edid.bin`
- Add the following to your kernel command line:
  ```
  drm.edid_firmware=eDP-1:edid/edid.bin
  ```
  If your display ID in hybrid mode is not `eDP-1`, adjust accordingly.
- Include the EDID file in your initramfs so it is available early in boot:

```bash
# /etc/mkinitcpio.conf
# ...
FILES=(/usr/lib/firmware/edid/edid.bin)
# ...
```

- Rebuild your initramfs and reboot:

```bash
sudo mkinitcpio -P
```