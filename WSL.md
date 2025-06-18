# what is WSL?

Windows Subsystem for Linux (WSL) is a feature of Windows that allows you to run a Linux environment on your Windows machine, without the need for a separate virtual machine or dual booting. WSL is designed to provide a seamless and productive experience for developers who want to use both Windows and Linux at the same time.

# install WSL on windows 11

*run command* `wsl --install`

*the command above will install* ***WSL** *and an ubuntu on your windows automatically then you should restart the system and then search Ubuntu in start menu, and start it*

*this had no openssh-server i used* ***`sudo apt update`*** *then* ***`sudo apt install openssh-server`*** *to install it and then* ***`sudo systemctl enable ssh`*** *and* ***`sudo systemctl start ssh`*** *to run the service.*

# WSL commands asked from DeepSeek

`wsl --install -d <DistributionName>` *to install another distro*\
`wsl --unregister <DistributionName>` *to uninstall an installed distro*\
`wsl --list --verbose` *to list installed distros*\
`wsl --list --online` *to list what distros you can install*\
`wsl --set-default <DistributionName>` *to set default distro to run when run* ***wsl*** *command*\
***by default : The last installed distro usually becomes the default when running wsl without arguments.***\
`wsl --terminate <DistributionName>` *to poweroff a distro*\
`wsl --shutdown` *to shutdown all running distros*\
`wsl --list --running` *to list all running distros*


## By default, when you install a WSL distribution (like Debian) using `wsl --install -d Debian`, it gets installed in:
`C:\Users\<YourUsername>\AppData\Local\Packages\<DistroPackage>`

***But you can choose a different drive/location using one of these methods:***

### Method 1: Export & Import to a Custom Location (Recommended)

*First, install Debian normally (if not already installed):*\
`wsl --install -d Debian`\
*Export the distro to a .tar file (backup):*\
`wsl --export Debian C:\temp\debian.tar`\
*Unregister the old Debian (deletes from default location):*\
`wsl --unregister Debian`\
*Re-import Debian to your desired drive (e.g., D:\wsl\debian):*\
`wsl --import Debian D:\wsl\debian C:\temp\debian.tar --version 2`
* *--version 2 ensures WSL 2 (recommended for performance).*
* *Replace* `D:\wsl\debian` *with your preferred path.*
*Set default user (optional, if you want your old user back):*\
`debian.exe config --default-user YourUsername`

### Method 2: Manually Install from Scratch (Advanced)

*Download the Debian rootfs* `.tar.gz` *from:*
* *Debian WSL GitHub (unofficial)*
* *Or Debian’s official site.*

*Place the* `.tar.gz` *file where you want it (e.g.,* `D:\wsl\debian\debian-rootfs.tar.gz`*).*\

*Import it directly to your chosen location:*\
`wsl --import Debian D:\wsl\debian D:\wsl\debian\debian-rootfs.tar.gz --version 2`\
*Launch Debian and set a username:*\
`wsl -d Debian`
