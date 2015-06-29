title: Transmission: Pi
subtitle: A Tasty Bittorent Server
description: I finally get round to making an always on Bittorent client, for getting the latest linux distros I use.
created: 2015-06-30 00:02:49

# Transmission: Pi

So regularly, I find myself needing another install disk for a particular Linux
distribution - be this for a desktop, a server, or a Raspberry Pi. Which means
that I have, at any one time, atleast 2 downloads for various Linux distros on
any one particular machine I use... although it is rarely the one which I want.
This is a big issue at times, depending on the size of the installer - A
generic installer for the Raspberry Pi alone of just Raspbian is 1.03Gb. This
adds up to about 10 - 15 minutes on a decent connection, but could mean an hour
or two somewhere else. So it is probably worth having a Torrent Client
constantly running.

Using a desktop, or server is an option for this, however for something which
is not going to be doing a whole lot of heavy lifting, they seem like a bit
overkill for always on - also the power cost is fairly huge for something like
that too. Enter the Raspberry Pi.

On its own, however there is little use trying to use a Pi for this. Most
people only use an 8Gb card for the main OS, and that isnt much for 1Gb install
images. So an external hard drive is also a must.

The final thing you will want is a wired connection for your Pi, unless you are
completely insane and want a wireless thing... yea lets not go there, you're
already reading this blog!

So, the parts list:

* Raspberry Pi (B, B+, or 2 - one with an ethernet port)
* Memory card with Raspbian installed
* External HDD - I'm using a 1TB 2.5" Samsung USB drive
* Ethernet connection to your home/office network

## Putting it together

So the first thing to do is wire it all together. I'm not going to insult you
by showing you how to do that, however do check you get the USB the right way
up on the 3rd try. You will also want to set up the Pi with ssh access, and
probably your own username and password access. You will also probably want the
external HDD to automatically mount on boot.

### Mount a HDD on boot

To get a drive to automatically mount on Boot, you will want to add it to
`/etc/fstab` so that it can be correctly identified. When you plug your drive
in, it will be assigned a device name similar to `/dev/sda1` which corresponds
to the first partition on the `sda` device. If you have multiple devices, and
also multiple partitions on a device, then this is not guaranteed between
reboots of your machine - one of the other devices may respond quicker the next
time round. What you should use instead is the devices UUID, or unique
identifier, which you can find out using the assigned name. To find out the
assigned name, you can use `fdisk -l`. This will need to be run as sudo or the
root user, and should give an output something like this:

    Disk /dev/mmcblk0: 16.0 GB, 16005464064 bytes
    4 heads, 16 sectors/track, 488448 cylinders, total 31260672 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x0002c262

            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1            8192      122879       57344    c  W95 FAT32 (LBA)
    /dev/mmcblk0p2          122880    31260671    15568896   83  Linux

    Disk /dev/sda: 1000.2 GB, 1000204885504 bytes
    38 heads, 27 sectors/track, 1904020 cylinders, total 1953525167 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000bc67c

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1            2048  1953525166   976761559+  83  Linux

From this you can see the that there are two devices, one with two partitions
(`/dev/mmcblk0`) and another device with one partition (`/dev/sda`). The next
step is to search for the device by ID, looking for the partition you want - in
my case its `/dev/sda1`.

To find the UUID's available, run `ls -la /dev/disk/by-uuid` - you will get an
output similar to the following:

    tbsliver@tux ~ $ ls -la /dev/disk/by-uuid/
    total 0
    drwxr-xr-x 2 root root 100 Jan  1  1970 .
    drwxr-xr-x 6 root root 120 Jan  1  1970 ..
    lrwxrwxrwx 1 root root  15 Jan  1  1970 2654-BFC0 -> ../../mmcblk0p1
    lrwxrwxrwx 1 root root  15 Jan 25 15:58 548da502-ebde-45c0-9ab2-de5e2431ee0b -> ../../mmcblk0p2
    lrwxrwxrwx 1 root root  10 Jan  1  1970 d4f121eb-8b3f-44b9-b886-aa020032adc9 -> ../../sda1

From this i can see that the UUID is `d4f121eb-8b3f-44b9-b886-aa020032adc9` for
that partition. This can then be used in `/etc/fstab`. The next step is to
create a mounting point for it, which I will create at `/media/Tardis` (an
ongoing theme of my HDD's) with the command `sudo mkdir /media/Tardis`. The
final things you need to know are the filesystem used on the device (in my case
its `ext4`). So I will edit mine to look like the following:

    proc            /proc           proc    defaults          0       0
    /dev/mmcblk0p1  /boot           vfat    defaults          0       2
    /dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
    # External Drive
    UUID=d4f121eb-8b3f-44b9-b886-aa020032adc9 /media/Tardis ext4 defaults 0 0

There are many more options you can add to this, so its well worth reading the
man page for fstab - run `man fstab` to read this.

### Installing Transmission

The application we will be using for this is called Transmission. It is quite a
lightweight Linux based Bittorent client, and actually has a non-gui option in
the form of a daemon. So lets install it:

    sudo apt-get update
    sudo apt-get install transmission-daemon

Now we have the basic transmission daemon installed, stop the process so we can
change a few settings. One thing to note about transmission-daemon is that if
you change any settings, then try and to a `restart` it will overwrite your
changes with new ones. You will have to do a `reload` to get it to fully reload
the settings from the file. So the best bet is to just stop it at the moment,
and start it again when we've finished:

    sudo service transmission-daemon stop

Now we want to change a few of the settings. These are stored in
`/etc/transmission-daemon/settings.json`. But first, we should create a
directory to put the downloaded files in. For this, change to the directory you
mounted your external HDD at earlier, and make a few directories, assigning
them the correct user and group:

    sudo mkdir /media/Tardis/torrents
    sudo chown debian-transmission:debian-transmission /media/Tardis/torrents
    sudo mkdir /media/Tardis/torrents/downloading /media/Tardis/torrents/finished
    sudo chown debian-transmission:debian-transmission /media/Tardis/torrents/*

This has now created the folders we will point Transmission at for downloading,
and they will all exist on our external HDD, saving the space on the SD card.
So now, open up the settings file with your favourite editor (mine is vim) -
you will have to do this as root or with sudo, for example `sudo vim
/etc/transmission-daemon/settings.json`. From here, you can change the following lines:

    "download-dir": "/media/Tardis/torrents/finished",
    "incomplete-dir": "/media/Tardis/torrents/downloading",
    "incomplete-dir-enabled": true,
    "rpc-password": "example",
    "rpc-username": "tbsliver",
    "rpc-whitelist": "127.0.0.1,192.168.0.*",

Here you can see we have set the download dir to one we created earlier (the
finished directory), and also set up a seperate incomplete directory (called
`downloading`) for torrents in progress. This saves having a bit of a mess in
the main download directory - You have to enable tihs option, hence the
`incomplete-dir-enabled` option.

The other things to note are the `rpc` options. Note the password is plaintext
- this is only like this until you start transmission again, at which point it
hashes the password in this file. I have also changed the `rpc-whitelist` to
include my local network, otherwise it will deny your conections. The final
thing is the username, which im obviously going to use as mine.

After this is changed, there are a few other settings which may be worth
sorting, however these are easiest done through the web interface. So now you
can start the daemon:

    sudo service transmission-daemon start

And then go to the IP address of the Pi, on port 9091 (by default). For me this
was `192.168.0.130:9091`, which then asked me for the username and password I
set earlier. After that, it should give you a standard Transmission window in
your browser!

### Batten Down The Hatches

The final few steps are just minor belt-and-braces parts, best practices I have
found when using always on Torrent Clients. The first is, ENABLE ENCRYPTION.
Even if all your torrents are legal ones, I find its best to just enable this -
go to the settings window by clicking the button in the bottom left with the
spanner on, and go to the `Peers` tab. In the dropdown menu, select `Require
encryption`.

The other thing I have done is to throttle the client during normal internet
hours, which for my house is between 8AM and 1AM. For that, go to the `Speed`
tab in the settings window, and set a reasonable Upload and Download speed for
during the day - this all depends on your local internet connection. Then tick
the `Scheduled Times` box, and pick a set of times and frequency which suits
you. This way, when most people are using the internet, its not going to wipe
everyones bandwidth out, and when its out of hours then it can just download as
fast as it wants.

## And There We Go!

Well, that was longer than I expected, however its still a useful tutorial on
how to set up a web only Transmission client. There is obviously more which can
be done for this, and if people would like to know more or have questions,
please ask in the Disqus section below! Guest comments are allowed, however
abuse of this feature will mean I break your fingers with a cricket bat. Oh,
and also turn it off. So you've been warned! But comments and criticisms, hints
and tips are all welcome.

Now then, its Stupid-O-Clock, and I could do with sleep. Night!
