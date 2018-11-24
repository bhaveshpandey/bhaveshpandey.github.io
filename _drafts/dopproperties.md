---
layout: post
title: Dop Properties.
subtitle: An introduction to data driven SOP based DOP simulations.
date: 2018-11-19
tags:
- houdini
- procedural
- python
- dynamics
- sops
- dops
---

## Motivation

Almost all the VFX faciliites I have worked with, have wrapped up or want to wrap up Houdini's DOP (Dynamic operators) context into SOP (Surface operators) based tools. There are a variety of motivations behind this. Some want to limit the useage of the more expensive Master (now named FX) licenses; while some want to abstract away all the additional complexities which come with DOPs for a more inexperienced workforce; some want to reduce the setup/iteration times of the effects.

Ultimately, the major driving force usually is making the Dynamics pipeline as much cost effective as possible using minimal number of FX or Master licenses for trivial effects which usually dont require bespoke setups.

Most of these solutions are monolithic DOP setups wrapped into a SOP context digital asset which offers a quick solution to a specific problem at hand but does a pathetic job when things start to diverge from standard requirements. This is a major shortcoming of monolithic digital assets and really pains me to see more of these at many facilities.

Seeing no good solutions at hand, I decided to make a better toolset and workflow aimed at solving this problem, and **dopproperties** is the solution I came up with.

The operators shipped with **dopproperties** are designed to be used as a replacement to their counterparts in DOPs and the UI reflects this by keeping the parameters, terminologies and layout as similar to the coressponding dop operators.


## Features

* Modular.

* Highly extensible.

* Complete control over DOP data tree and dop object properties.

* SOP driven.

* Orthogonality.

* Extensive DOP operator coverage.


## What is *dopproperties*?

At its core **dopproperties** is a framework to develop sops which allow for dynamically modifying the DOP data tree.

**dopproperties** provides SOP based toolset to control dop data tree, effectively controlling the simulations inside DOP context without ever diving inside the context.

It allows users to work with DOP nodes at sop level as shown in the following figures.

![A very simple setup - sop](/img/projects/dopproperties/simple_01_sop.jpg)

There is a basic dop level architecture which enables these SOP level networks to drive the DOP context and fully control the DOP data tree. It simply involves applying a custom solver to our dop network. This solver is responsible for reading all the data applied in the SOP context and control the DOP context.

A simple dop network which will solve the above sop graph looks like this.

![A very simple setup - dop](/img/projects/dopproperties/simple_01_dop.jpg)

Lets look more closely at these graphs and what they mean.


## What dopproperties is for?

## What dopproperties is not meant for?

## How to leverage dopproperties architecture to keep DOP abstractions flexible and open.
