---
layout:     post
title:      Using Opencv with Cinder using Cmake on OSX
# subtitle:   I hate xcode!
tags:
    - cinder
    - opencv
    - cmake
---

While rummaging through my hard disks to do the most dreary of tasks **disk cleanup**, I found
some photos which were good but were suffering with some amounts of perspective distortion.
There are softwares out there which do help in correcting the perspective distortion to certain
degrees but what fun is that!

So I began with the barebone project testing/prototyping the code and found myself in need of
a way to handle user input quickly to be able to define straight edges in the input image.
While opencv does provide a minimal gui functionality, I did want to use something which was
a bit more robust and would allow me to build ui fairly quickly.

**Quick question**: How do you ask a Vim user to use xcode?

**Answer**:         You dont!

One of the more important requirement for me was: Avoid using **xcode**. I hate xcode!
Coming from a Linux background, I primarily use Vim and this being software I am writing
for personal use, I definitely dont want to be suffering from using xcode or eclipse or some other shit.

After cloning the massive github repo for OpernFrameworks, I came to realisation its more tightly integrated
with xcode on OSX (which is quite a shame). Comparitively, using it on Linux is a remarkably easy.

Having used [OpenFrameworks](http://openframeworks.cc) in the past, I gave it a quick try.

Enter [Cinder](https://libcinder.org). Cinder is a c++ based creative coding framework.
