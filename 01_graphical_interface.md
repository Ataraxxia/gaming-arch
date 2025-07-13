 # Introduction

This section of the guide will tell you how to add graphical interface to your Arch installation.

__Some of my assumptions and notes__

- I assume you have fairly recent hardware, especially when it comes to GPU's. By that I mean Nvidia RTX 50, 40 or maybe 30 series card. I don't know how things work on older ones nor I care.
- I've tested this tutorial on my desktop PC. Laptops with dual GPU's are places where dragons be. 

## NVIDIA driver

First install prerequsisties:

    pacman -S gcc cmake

If for some reason you omitted kernel headers, make sure it's installed:

    pacman -S linux-headers

Finally, instal the nvidia open driver with a helper package:

    pacman -S nvidia-open-dkms nvidia-utils

If everything went fine, you should be able to execute `nvidia-smi` and it should show you your GPU along with some basic metrics.

    <placeholder>

I'd suggest you reboot after this.

## AMD driver

    <TODO, i've never had an AMD GPU>

## Graphical enviroment

At this point you're free to chose whatever: hyperland, sway, gnome; pick your poison. I will suggest you install KDE Plasma however, because it comes with many sane options preconfigured and it leans heavily toward flatpacks. It also supports Wayland and utilizes your GPU for increased performance. 

Installing plasma can be as simple as (choose all the defaults by pressing ENTER):

    pacman -S plasma

You can launch plasma manually by executing:

    /usr/lib/plasma-dbus-run-session-if-needed /usr/bin/startplasma-wayland

If you plan on omitting the next step, make a script out of it and make it executable:

    vim ~/start_plasma.sh
        /usr/lib/plasma-dbus-run-session-if-needed /usr/bin/startplasma-wayland
    
    chmod +x ~/start_plasma.sh

    ./start_plasma.sh # This will start the UI

### (Optional) Login manager

By default plasma does not come with a login manager, so that means you will be greeted by TTY login console and will need to start your graphical enviroment manually. Personaly I like it, because I found login managers to be potentialy wierd point of failure when you try to customize them.

Preffered login manager for plasma is the SDDM, install it and enable:

    pacman -S sddm
    systemctl enable --now sddm.service


## Final stretch

Plasma has very little typical packages preinstalled. By that I mean webbrowser, terminal emulator or file browser. You can install them via CLI but Plasma provides a flatpack interface that can install pretty much anything you might think of.

I'd suggest you install:

    konsole - terminal emulator
    firefox - web browser
    dolphin - file manager

Now, when your KDE Plasma is installed and launched, i'd suggest you verify that your GPU is properly utilised, execute:

    nvidia-smi

It should show you your desktop processes utilizing your GPU.