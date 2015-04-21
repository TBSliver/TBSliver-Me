title: Playing With Init
subtitle: How many distros?
description: In which I go over the many things I've just tried on a spare laptop, to see whats available.
created: 2015-04-21 22:28:16

# Paying With Init

So this is less about me playing with init itself, but playing with multiple
different distros which have different init systems. But why, you may ask? Well
if you know me, you probably have also heard about the current `systemd`
debacle, which is currently tearing a lot of communities apart - some want to
use this new shiny, while others are wary of it, as it is (relatively)
untested. I sit on the fence - I dont mind it being on my desktop and laptop
systems, as bootup speed is sometimes actually important, but any of the
servers I currently administer I am putting it nowhere near. This is mainly
due to wanting 'stable' software, which generally means Debian Wheezy, which
is exceedingly tried and tested.

I am also unlikely to try anything new on the currently running servers, mainly
because 'if it aint broke, dont fix it', but also because its best practice to
test any new dependencies first on a staging or testing environment.

## So you dont mind systemd?

No, not really. Its another answer to a hard problem, although as with
anything, there are always decisions made which people dont agree with that
much. The one I am not such a fan of, is the use of binary logging, and also
what I feel to be a slightly more obscure way of managing daemons. Now, dont
get me wrong, haveing a central database of all the running daemons is damn
useful, and makes it much easier to see what is actually running on a server -
although using `top` or `ps` is also possible. And having an easier way of
defining start/stop/restart scripts, in a standard format, is also useful.

However, limiting init scripts to JUST start/stop/restart/reload, is fairly
idiotic. I know of several apps which define extra init points, allowing you to
test your setup - for example, OpenNMS has a CheckJava option, which will make
sure you have actually set up the java environment correctly, and it hasnt
broken in an update. Also, being able to gracefully shut down Apache, as
opposed to the `kill -9` shutdown, means that when using something behind a
load balancer you dont end up with customers sites just dying because the
server they happen to be on currently before the load balancer kicks you over
to another server is rebooting.

## Thats all server stuff though

True, however as I am using it every day for work, I am getting very used to
the way it works. And having to re-write init scripts to work with a new
system, as well as introduce a full migration path for them, is something that
I am really not looking forward to doing, especially with something which seems
to have more power than an init system should. So, I have been looking at other
Distros, to see what is available and how things work - first on a Laptop
machine, so I can play without worrying about my main development machine or
any of my servers. And here is what I've looked at.

## Elementary OS

Yes, ok, its a desktop only distribution, but its been intreaguing me for quite
a while, and with a new release just out, I thought I'd take a quick peak at
it. First thing to notice is, it's definitely based on Ubuntu, with a very nice
interface. However, the machine I installed it on doesnt have much grunt, and
so due to the fact it seems to be a heavily re-skinned Gnome 3 interface, it
ran very jerkily - with no way to turn off the animations among other things.
So, very nice, but not for this machine. Onto the next...

## Devuan

Whats this? Debian without systemd? Well I have been keeping an eye on this for
a while, and last week found out that they had released a pre-alpha installer,
very much based on the current Debian installer. And it worked! Installed all
the way through, got me to an XFCE4 desktop (my prefered DE!) and looked good!
right until it disconnected from the Wi-Fi mid update and refused to
re-connect. So, nice try! I will definitely come back and visit it when it's
had a chance to iron out some of the creases, as I really dont have the spare
time to help with this project all that much. Which brings me onto the next
Distro

## FreeBSD

This is one which I have played on while working on a client provided dev box,
and in all honesty is very interesting. I have yet to install this, but it is
next on the list, and intreagues me due to the way that the package management
can either be done using a simple package manager, or using whats known as the
ports tree - you navigate to the folder corresponding to the program you want
to install, and then `make install clean` in there - and it takes care of the
rest. One other bonus is, the documentation for this Distro is amazing, they
have rock solid support, and you can customise your machines as much as you
need. Also, no systemd, as it doesnt run on any of the \*BSD's.

So thats it for now, I felt like a little rant, and letting people know what
I've been up to! Also this weekend I will be up at MakerFaire Newcastle, on the
HACMan and LAMM stand, so look for me there! I will hopefully be posting some
updates about various things I've been working on soon too.
