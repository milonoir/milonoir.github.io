---
title: How it's made? part 1
layout: post
lang: en
ref: how-its-made1
date: 2014-10-03 16:20:19 +0100
categories: archive
---

As I [stated before]({% post_url 2014-09-30-hello-world %}), the idea of running my webpage on a *tiny server* at home came about 2 years ago. Before I had this idea, I had browsed the internet for all kinds of free or cheap hosting services. The only thing I knew was that I didn't want to spend too much on it and I didn't want to have too much ads on my site either. This was when I first heard about the *credit card sized*, low consumption computers. Soon it turned out that I could do everything by myself.

#### Hardware

I went with the [Raspberry Pi](http://www.raspberrypi.org/) which was very popular that time. I chose version B, to be more specific, which was more powerful. It contained a 700 MHz ARM CPU accompanied by 512 MB RAM. On the board there were also two USB 2.0 ports, an ETH port, a composite video out, a 1/8" (3.5 mm) jack audio out, an SDHC card reader and a micro USB socket for mains. I note here that the Pi can be powered by a common cell phone charger with 5 V / 700 mA output. If we calculate with these numbers it results approx. 30 kWh *annual* power consumption. I bought the Pi in a kit via eBay, so I got in a transparent plastic case. Since it came from China the color of the PCB is red. Nice! Besides this I also bought a Class 10 SDHC 16 GB card and a charger. Total cost was around 16000-17000 HUF.

![Raspberry Pi](/assets/img/img_raspberrypi.jpg)

#### Operating System

I started with an empty SD card where I had to install some operating system. Today you can order a pre-installed one, but it's not rocket science. I followed the [instructions](http://www.raspberrypi.org/help/noobs-setup/) and then booted the Pi. For the first time I connected it to my TV and plugged in a keyboard and a mouse. The NOOBS (New Out Of The Box Software) offered some OSs to install, I just had to make up my mind. Since I was most familiar with Ubunutu-like Linux systems I chose the very recommended Raspbian, which is a Debian optimized for ARM architectures. After the installation I set up SSH (and VNC, which I rarely use) server and the static IP address (Pi was connected to my home router in the meantime), so I didn't need the TV, keyboard and mouse anymore.

#### Software

I have dug up countless forums and blogs to find the optimal software stack for my purpose. I had two thing to start from: Raspberry Pi and Django (based on Python 2.7). So I had to find fast and lightweight database and webserver for these criteria. SQL? NoSQL? Django offers SQLite3 for test purposes and small projects. In fact, this is far enough for a blog, so I went with it. Using MySQL or Postgre would be overkill. If I were mistaken, I would try MongoDB.

If I say webserver, it's Apache that comes into people's mind first. Perhaps a bit too much, but it's [possible](http://www.raspberrypi.org/documentation/remote-access/web-server/apache.md). I wanted something not that heavy. I had [read through](http://tutos.readthedocs.org/en/latest/source/ndg.html) [some](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/) [tutorials](http://www.apreche.net/complete-single-server-django-stack-tutorial/) and ended up with the Nginx-Gunicorn combo. In this setup the Nginx acts as a HTTP proxy and forwards the incoming requests to a static directory or to some application (Gunicorn). The Gunicorn connects the Nginx and the blog engine via a socket and serves dynamic contents.

The blog engine itself is the Django application wrapped in a virtualenv. The virtual environment segregates the blog from system Python and it consists of Python 2.7, Django 1.6 and third party libraries the blog is dependent on.

In a nutshell, this is how the system was built.
