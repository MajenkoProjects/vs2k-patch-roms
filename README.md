VAXstation 2000 SCSI Patched ROMs
=================================

The VAXstation 2000 is a fantastic little "shoe box" VAX computer. A little
slow maybe but a great entry to the wider world of DEC's VAX computers.
However it is somewhat lacking in one area: storage. It has one (optional)
internal hard drive (mine came with a 150MB RD54) and one (optional)
external hard drive - both of the MFM variety. That's not a big amount of
storage, and old MFM drives are as you probably know prone to stiction and
bad blocks, if they even work at all. 

The VAXstation 2000 does have a SCSI interface, however it was only intended
to be used with the optional external TZK50 tape drive, and as such has never
had any real support for adding hard drives, and it certainly cannot boot
from a SCSI hard drive even if the operating system supported accessing one.

But all is not lost. An experimental patch set for the VAXstation 2000 was
created (but never officially released) to add full SCSI support to both the
internal ROMs and to VAX/VMS, but only one specific version. Which is what
we have here.

Using the patches is not the easiest thing in the world, since you need
an already working installation of VAX/VMS 5.5-2 (yes, that exact version)
in order to run the patches. And if you don't have a working hard drive
to operate the VAXstation on, how are you going to get a working installation?
The answer is to net-boot it with a VAX cluster. But doing so makes it
rather hard to then make a standalone image. But it can be done with a bit
of shoehorning. Which thankfully we have done for you. In the releases
section you will find a set of BlueSCSI hard drive images that should allow you
to boot a VMS installation (dka200) that is ready to be completed with
your own details, along with the installation source files (dka300) which
the installation will ask you for.

Using it
--------

In short:

1. You need to upgrade your ROMs using the four ROM images (b1-b4) in the ROMs
   directory. You will need four new 64kB EPROMs (27512, 27C512, etc) to burn the images to then
   replace the ROMs on your VAXstation 2000's motherboard.

2. Connect the BlueSCSI to the SCSI bus (important: see belo), making sure to enable termination if
   you don't have the TZK50 tape drive attached or an external terminator.

3. Boot from the dka2 device:

```
>>> boot dka2
```

4. Complete the installation following the on-screen prompts. When it asks for
   the installation media tell it `dka300`.

5. Install the patched binaries into the installed os:

```
$ mount/over=id dka300
%MOUNT-I-MOUNTED, VMS552       mounted on _DKA300:
$ set def dka300:[scsi.bin]
$ @install
```

Now the long version:

Installing the ROMs is pretty straight forward. There's 4 EPROM chips on the motherboard
which need replacing.

![ROMPlacement](https://github.com/user-attachments/assets/eb0417ef-58f3-4042-aaf1-6d31a02361bb)

Note the order, as marked on these ROMs.

The BlueSCSI (which should be a BlueSCSI II Desktop version so you get the proper 50 pin
IDC connector) will not work with the VAXstation out of the box due to the VAXstation
being from the dawn of SCSI and not working in quite the same way as other systems
(the joys of old computers...) so the BlueSCSI will need to be modified in order to work.

The problem here is that the SEL pin and RST pin of the SCSI are weakly tied together inside the
BlueSCSI, so when the RST pin is asserted by the SCSI initiator the SEL pin gets pulled
low, which shouldn't happen. Most things don't care - but the VAXstation does, and when that
happens it just gives up trying to do anything on the SCSI bus ever again.  So we need
to separate those two signals. This needs some careful SMD rework.

1. Remove the resistor R66. This is the resistor that ties the two signals together.
2. Connect the left hand pad (while the text of R66 is in the normal orientation)
   of R66 to the pin GPIO17 of the Raspberry Pi Pico. This
   gives the MCU direct control over the SEL pin.
3. Install modified firmware into the Pi Pico which moves the SEL functionality to
   GPIO 17.
   
<img width="466" height="211" alt="BlueSCSIPatch" src="https://github.com/user-attachments/assets/802cbc62-8243-49b3-af2c-86bdc8daa508" />

This modification uses one of the pins that was used for I2C for the SEL pin, so you will
no longer be able to use the I2C connector on the BlueSCSI.

For the curious
---------------

If you're curious about the steps needed to create the patched VMS images, here's what
we had to do:

1. Create a VAX/VMS 5.5-2 cluster using a SIMH emulated server.
2. Enrol the target VAXstation 2000 into the cluster as a member.
3. Patch the system using the original patch set
4. Verify that SCSI is working and create two blank images on the BlueSCSI (id 2 and 3).
5. Create a standalone backup kit on DKA300
6. Copy the VAX/VMS-5.5-2 installation backup files onto DKA300
7. Copy the SCSI patches to DKA300
8. Boot from DKA300 and install the VAX/VMS base system to DKA200
9. Reboot from the network and manually copy the patched files to the
   correct places on DKA200

As you can see it's not a simple operation.

One last word
-------------

Good luck!
