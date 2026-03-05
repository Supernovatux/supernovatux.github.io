---
layout: post
title: "Effortless ROS Workshop Setup with Virtual Machines"
lead: Pre-configure once, distribute to everyone
---

**TLDR;** Install ROS in a VM, export it as an OVA file, and distribute it to attendees. Setup takes less than 10 minutes on their end.
{: .message }

I recently had to conduct a workshop on ROS + MoveIt with a tight time constraint of one hour. By pre-configuring a virtual machine, participants can skip all the setup friction and jump straight into the content.

# Procedure

1. Choose a lightweight distro. I used Lubuntu.
2. 9 GB of virtual disk space is sufficient for Lubuntu + ROS + MoveIt + Jupyter Notebook.
3. Do a minimal install in the VM — skip third-party drivers.
4. Install VirtualBox Guest Additions from the `apt` repository.
5. *(Optional)* Remove Bluetooth and other unneeded packages to keep the image small.
6. Install ROS.
7. Configure the workspace and environment variables.
8. Install ZSH with autosuggestions — participants will thank you.
9. Make sure all workshop files are present in the VM.
10. Export the VM as an OVA file.
11. Share it with your participants.
