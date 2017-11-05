# pixelbook-crouton
Configuring [crouton](https://github.com/dnschneid/crouton) inside my pixelbook


## How
I did add a crouton install of linux stretch _after_ having attempted galliumos boot from live ISO.
As a result I already did
1. put machine in dev mode.
   1. enable usb boot
   1. enable legacy boot.

I also already had the crouton extension installed
At which point, it is a simple matter of downloading `crouton` from https://goo.gl/fd3zc and then running

without encryption:
```
sudo sh ~/Downloads/crouton -r stretch -t xfce,extension,xiwi -n debian
```

I quickly discovered:
* a few fixes are required to run xfce (graphical interface i chose)
* I don't really want a GUI. I need terminals, with tabs in a GUI, or tmux + emacs mostly.

So, I could have gone with a lighter coreutils only version of stretch.

### issue #1 : XFCE won't start
* exactly described with crouton issue [#2676](https://github.com/dnschneid/crouton/issues/2676)
* workaround is to replace `console` with `anybody` in `/etc/X11/Xwrapper.config`
```
allowed_users=anybody
#allowed_users=console
```
* sudo startxfce4 works
* switching back and forth between full screen xfce and chrome has issue. (F11 does not work). I use chromes F5 (window switcher)

### issue #2 : tmux won't start
* need to install locales and add an UTF8 one.
```
echo "LC_ALL=en_US.UTF-8" >> /etc/environment
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
apt-get install locales
```
* then tmux runs

# STILL UNDER WORK : EMACS IN CHROME SHELL : BINDINGS ARE INTERFERING
I may consider working inside XFCE just because of that

# Docker won't run inside a chroot on ChromeOS
This one is particularly annoying and nasty, as I know for a fact that [GCP's container optimized OS](https://cloud.google.com/container-optimized-os/) is actually based on Chromium OS.
`cgroups` seem to be missing.

That said, for container afficionados, it turns out that _when run as root_, CoreOS `rkt` can run containers inside a crouton `chroot` jail in ChromeOS...

First you need to [install `rkt` as per CoreOS instructions](https://coreos.com/rkt/docs/latest/distributions.html)
Since I do not use `sid` but `stretch` I used [these specific instructions](https://coreos.com/rkt/docs/latest/distributions.html#deb-based]):
```
gpg --recv-key 18AD5014C99EF7E3BA5F6CE950BDD3E0FC8A365E
wget https://github.com/rkt/rkt/releases/download/v1.29.0/rkt_1.29.0-1_amd64.deb
wget https://github.com/rkt/rkt/releases/download/v1.29.0/rkt_1.29.0-1_amd64.deb.asc
gpg --verify rkt_1.29.0-1_amd64.deb.asc
sudo dpkg -i rkt_1.29.0-1_amd64.deb
```
Then it works. You just need to learn `rkt` command line syntax instead of docker.

```
sudo rkt run --net=host docker://hello-world --insecure-options=image
sudo rkt run --net=host  docker://alpine --name=foobar --insecure-options=image --interactive --exec=/bin/sh
```

## Import your cryptographic keys
left as an exercise
And then you should be ready to roll
