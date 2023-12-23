---
layout: post
title: " ROS Workshop Setup"
lead: with Virtual Machines
---

**TLDR;** Install ROS in VM. Then export it as OVA file. Distribute this file with the audience. It should take less than 10 mins.
{: .message }

So recently I had to conduct a workshop on ROS+Moveit. The workshop had a big time contain, 1 hour. By following these steps, you can conduct a ROS and Moveit workshop efficiently in just 10 minutes. The pre-configured virtual machine allows participants to focus on the content of the workshop without wasting time setting up the software.

# Procedure
1. Choose a lightweight distro. I choose Lubuntu.
2. A Virtual HDD space of 9GB should be enough for Lubuntu + ROS + MoveIt + Jupyter Notebook
3. Do a minimal install in the VM. Do not install 3rd party drivers.
4. Install guest additions from the `apt` repository.
5. (Optional) Remove Bluetooth related and other not needed packages.
6. Install ROS.
7. Configure the workspace the environment variables.
8. Do yourself a favor by installing ZSH with auto suggestions on the machine.
9. Make sure that all the files needed for the workshop is present in the VM.
10.  Export the VM as a OVA file.
11. Now share it with ppl :)