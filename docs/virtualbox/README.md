# VirtualBox

- [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)


- [https://superuser.com/questions/109485/virtualbox-to-use-dual-monitors](https://superuser.com/questions/109485/virtualbox-to-use-dual-monitors)

VirtualBox 3.2.1 supports multiple guest monitors. The documentation was not clear on how to enable this.

Basic Setup
  1. Power off your virtual machine if it's on.
  2. From the main VirtualBox window, select your VM and choose “Settings”.
  3. Choose “Display”.
  4. Below “Video Memory” is “Monitor Count”. Slide it to 2, and adjust your video memory if VirtualBox complains.
  5. Start your guest and perform the standard method for your guest OS to Extend the desktop onto a second monitor. (Guest Additions need to be installed.)

A second “Oracle VM VirtualBox” window will appear with the second display. You can resize it however you want.
The VirtualBox “View” menu will have an entry for each “Virtual Screen”. All but the first can also be enabled/disabled from here. This seems to only work after step 5.

Seamless/Fullscreen
  - Enter Seamless or Fullscreen. I'll assume your HostKey is the default “RightCtrl”.
  - If the screens are on the wrong displays, hit RightCtrl+Home.
  - From the View Menu, choose “Virtual Display 1” and set it to the Host display you want. The other displays will shuffle around to accommodate this. If you have more than two virtual displays, repeat with “Virtual Display 2” and so on.

Headless
  - Set the number of monitors with VBoxManage modifyvm "vm name" --monitorcount X
  - Enable multiple vrdp connections with VBoxManage modifyvm "VM name" --vrdpmulticon on
  - Use VBoxHeadless to launch as normal.
  - Connect to monitor 1 with rdesktop -d \@1 ip-address-of-host and connect to monitor 2 with rdesktop -d \@2 ip-address-of-host. This is explained in lomaxx's answer. (You might be able to use @ instead of \@, depending on your shell.)
