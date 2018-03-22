---
layout:     post
#date:       2017-11-24
title:      Building Cinder projects with Cmake on OSX
subtitle:   while using different directory to keep cinder projects.
tags:
    - cmake
    - cinder
---

A while back, I started writing some image processing code using opencv and
had to setup something to handle user inputs relatively quickly.
The first thought was to just use the highgui functionality native to opencv.
However, I found it to be fairly cumbersome to handle events.

So I decided to use [Cinder](https://libcinder.org) for this. I really dislike the
to keep my projects in the same directory as Cinder and wanted to setup something
which would allow me to separate my projects from the framework's directory.
Another preference for me is using Cmake for building my projects as opposed to
XCode which Cinder is very closely tied to.

Thankfully Cinder also ships a Cmake file which allows you to set your projects
up a bit more flexibly.
Here is what I had to do to get it to build my project.

```cmake

cmake_minimum_required(VERSION 3.0 fatal_error)

```
