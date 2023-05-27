==================
 Ubuntu on Star64
==================

These are my basic notes about how I got Ubuntu running on Pine64's star64 board.

The basics: booting Linux
-------------------------

To begin with, I checked that my device was at least basically functional by installing
`Fishwaldo's pine64 images <https://github.com/Fishwaldo/meta-pine64>`_ of Yocto.
The build I used was version 1.2, from 2023-05-17.

This got me a Yocto install that was quite usable to start with, complete with SSH::

    lengau@hyperion:~/Projects/ubuntu-star64$ ssh root@10.13.0.6
    root@10.13.0.6's password: 
    Last login: Sat May 27 21:14:05 2023 from 10.13.0.36
    root@star64:~# uname -a
    Linux star64 5.15.107 #1 SMP Mon May 15 17:57:25 UTC 2023 riscv64 riscv64 riscv64 GNU/Linux
    root@star64:~# cat /etc/os-release 
    ID=pinix
    NAME="PinIx"
    VERSION="1.2 (pinix)"
    VERSION_ID=1.2
    PRETTY_NAME="PinIx 1.2 (pinix)"
    DISTRO_CODENAME="pinix"
    root@star64:~# 

    
This was absolutely fantastic, as it meant someone else had done the hardest part
(bootstrapping Linux on the board) for me. I booted it from an SD card, as my goal is 
to target the eMMC module for the Ubuntu install.

Thanks to Stéphane Bortzmeyer for `this rundown <https://www.bortzmeyer.org/star64-first-boot.html>`_
of the state of Linux on the Star64. Things have improved a bit in the intervening month,
as PinIx now has a splash screen and will boot into a terminal.
(As I'm not aiming to use this full-time on my Star64, I didn't test any of the graphical
options, but it warms my heart to see plasma images there.)

The board didn't immediately talk to my ethernet switch, so I connected to wifi using
the preinstalled ``nmtui``, but as it turned out that was due to a bad ethernet cable
rather than nonworking ethernet, so I have since got it wired.

A big thanks to Fishwaldo for all his work, as he turned it into a 10 minute process 
for me to get a Linux machine running.

    
Sidequest: Sense-Ors
--------------------

I installed ``lmsensors`` to check how well the 
`Sense-Ors <https://www.youtube.com/watch?v=EM1YYefSmKg>`_ work. Mostly, I just want to
know whether the 30mm heatsink I got is enough to keep the processor from melting itself
while I do builds.

It seems to work out of the box, telling me that the CPU is around 45°C, which seems
reasonable::

    root@star64:~# sensors
    120e0000.tmon-isa-0000
    Adapter: ISA adapter
    temp1:        +46.5 C 


Ubuntu chroot
-------------

Since the Star64 is based on StarFive's chips, I decided to start with the closest
Ubuntu image, so I grabbed the 
``VisionFive image of Ubuntu 22.04 <https://cdimage.ubuntu.com/releases/22.04/release/>`_
on my desktop, decompressed it, and loop-mounted the first partition. I installed 
``rsync`` on the star64 board and rsync'd the files over. I probably could have just
used ``scp``, as it seems the limiting factor here was my SD card. Oh well, not going
to have that for **too** much longer fortunately, as I'll be using the eMMC card I
got for it soon enough::

    lengau@hyperion:~/Projects/star64$ sudo rsync --archive /mnt/ root@10.13.0.25:/srv/ubuntu
    The authenticity of host '10.13.0.25 (10.13.0.25)' can't be established.
    ED25519 key fingerprint is SHA256:Z5mh4/3m4YkUyCOgzSsi0a0hb1OQ4RpLF7IRg9cYI3M.
    This key is not known by any other names
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '10.13.0.25' (ED25519) to the list of known hosts.
    root@10.13.0.25's password: 
    lengau@hyperion:~/Projects/star64$ 
    

Now I just bind-mount the appropriate directories::

    root@star64:~# mount -o bind /dev /srv/ubuntu/dev
    root@star64:~# mount -o bind /proc /srv/ubuntu/proc
    root@star64:~# mount -o bind /sys /srv/ubuntu/sys
    
and hop in::

    root@star64:~# chroot /srv/ubuntu /bin/bash
    root@star64:/# cat /etc/os-release 
    PRETTY_NAME="Ubuntu 22.04.2 LTS"
    NAME="Ubuntu"
    VERSION_ID="22.04"
    VERSION="22.04.2 LTS (Jammy Jellyfish)"
    VERSION_CODENAME=jammy
    ID=ubuntu
    ID_LIKE=debian
    HOME_URL="https://www.ubuntu.com/"
    SUPPORT_URL="https://help.ubuntu.com/"
    BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
    PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
    UBUNTU_CODENAME=jammy
    root@star64:/# 

...and that's it. I (sorta) have Ubuntu running (in a chroot) on my Star64!

Next step: Turn this into an actual bootable Ubuntu install.
