# macbook-pro-4-1-linux
a collection of tricks for installing linux on early 2007 macbook pros pre unibody. Some of these things may apply to similar hardware

##Aquiring and installing 340 NVIDIA drivers

#Ubuntu

add ppa:

```
sudo add-apt-repository ppa:kelebek333/nvidia-legacy
sudo apt update
```

https://launchpad.net/~kelebek333/+archive/ubuntu/nvidia-legacy

#Debian

*untested*

add a file in /etc/apt/preferences with

> Package: *
> Pin: release a=unstable
> Pin-Priority: 100

add a file in /etc/apt/sources.list.d with

deb http://http.us.debian.org/debian/ testing non-free contrib main
