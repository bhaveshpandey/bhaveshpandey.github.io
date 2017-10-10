---
layout: post
title: Volumetric Deformations.
subtitle: Volumetric deformations without sampling to points and back.
date: 2017-10-09
tags:
- houdini
- procedural
- vex
- volumetrics
---

Often we are faced with scenarios where we need to deform volumetric simulations/data.
One of the most common solutions I have seen involves:

- sample the volumetric data to a point cloud.
- deform the point cloud.
- splat point clouds to volume.

This approach suffers from quite a few disadvantages.

- Its painfully slow.
- The round trip conversions to and from point cloud causes severe resampling
of voxel data.

Volume:             F(x,y,z)

Deformed Volume:    G(x',y',z')

We need to find/create an inversible function which maps (x,y,z) to (x',y',z') and also
(x',y',z') to (x,y,z).

This mapping allows us to sample volumes into the Deformed domain.
