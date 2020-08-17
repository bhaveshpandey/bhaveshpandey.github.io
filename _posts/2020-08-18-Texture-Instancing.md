---
layout: post
title: Texture Instancing.
subtitle: Instance textures based on point cloud.
header-img: img/projects/texture_instancing/rtoy_tex_instancing.jpg
date: 2020-08-18
tags:
- houdini
- vex
- texture instancing
- texture bombing
- procedural
- materials
---


## Background

For one of my personal projects, I was in need of generating a large scale terrain (approximately 1500km<sup>2</sup> with point density of 16.67 points/m<sup>2</sup>; more on this in another post). I wanted to experiment with a more flexible and procedural method to texture this terrain and began exploring the **texture instancing** approach. This allowed me to apply local/spatial overrides to textures based on proximity to different geometry in the scene.


## Introduction

Keeping in mind the large scale of the terrain and having a strong aversion to anything manual, I decided to implement <a href="https://developer.download.nvidia.com/books/HTML/gpugems/gpugems_ch20.html" target="_blank"><b>Texture Bombing</b></a> technology using Houdini's <a href="https://www.sidefx.com/docs/houdini/vex/index.html" target="_blank"><b>VEX</b></a> (Vector Expressions) language. The idea was simple, apply a basic procedural shader to the terrain as a base layer and use texture bombing for spatially localised texture overrides. This would allow me to layer visual complexity dynamically. For the rest of the post, I will refer to texture bombing as texture instancing as it makes more sense to me than the original term (albeit instancing technically means something different...but so does bombing).

Texture instancing is a technique which is fairly similar to geometry instancing in principle. The idea is to read specific textures on disk at specific position and project it on specified input geometry. This is achieved by writing the texture data(filepaths, colour tint etc) onto a point cloud which which also determines the position where the texture is projected onto the specified geometry.


For the implementation, I wanted to stress on certain factors in the specification of the toolset, some of those were as follows.

1. Efficiency - tools must be efficient and avoid any redundant computations/instructions.

2. Ease of maintainance - toolset must be easy to maintain. This applies to both the codebase, as well as the design of the tools and workflows resulting from it.

3. Core tech sitting at the center of various tools/visualizers leveraging it - this would allow me to roll out updates efficiently and at the same time reduce code duplication.

4. Seamless transition of the tech in between Houdini's contexts - often times, different variants of tools are developed for different contexts in Houdini (an implementation for the shader, another implementation for SOP context and another for DOPs etc) leading to duplication of work. **3** also helps in mitigating this.

5. Separation of implicit UV space creation from sampling of textures - this would allow me to add more blending modes in future easily.

To accomplish this, I wrote the core technology suite  in CVEX which is then used to create shaders and visualisers. This helped in reducing code rendundancy as both SOP tools and shader tools use the same code. It also enforces visual parity between the SOP visualizations and the shaders used for Karma/Mantra renders.

The tool is very generic, to the point that it can be thought of purely as a number cruncher. It performs following steps and outputs the specified data.

1. Query the point cloud and figure out the proxyimity points.

2. For each point determine the influence it has on a point on the mesh.

3. Compute the implicit UV space using the orientation and scaling factors specified. This will be used to sample textures onto the mesh.

4. Sample the textures and perform blending of textures based on the influence of each point.

5. Output the value.

This can be performed for any kind of data stored in texture maps (colour, displacements, normals, roughness etc).

# Use cases

Some of the common use cases for **Texture Instancing** are:

* Apply textures to large scale landscapes.

* Convenient randomisation and/or layering of textures.

* Procedurally apply textures at rendertime without having to worry about UVs. Imagine a terrain which has specific biome types and for each biome type, you would want to apply specific decals onto the terrain geometry. In such situation, you can easily query which biome type is over the terrain geometry and swap out (or in my case blend in the textures defined in the point cloud) the texture map to specific to the respective biome.

* Applying decal textures dynamically being driven by FX elements (gun shots creating a bullet hole, scratch marks, cracks etc).

* This could also be extended to Creature FX (think of wound/injury details, wrinkles, decals, scales on fish or lizards etc)

* Natural wear and tear details on surfaces using decals (cracks, scratches, chipping etc).


## Tech Demo

Lets have a quick look at a quick technical demonstration of the process and see what the inputs and output might look like.

Following is a very simple polygonal grid mesh. We will be instancing our textures onto this grid.

**Note**: The grid does not have UVs.

![Input grid](/img/projects/texture_instancing/input_grid.png)


Following image shows an example point cloud. Each point of this point cloud, will store the filepath of the texture on disk, position and orientation of the texture projection, scale of the sample texture which will be projected onto the geometry etc.

![Input point cloud](/img/projects/texture_instancing/point_cloud.png)

This particular point cloud has following attributes.

* pscale - used to scale the texture.

* orient - used to define the orientation of the textures. Note that the orientation is same for all the points. We will later have a look at point cloud with noise applied to the orientation.

* id - used to add random offset and debugging colour output to the input mesh.


For this technical demo, we will be using following texture for all the points. It will allow us to assert whether the orientation of the projections is being correctly enforced or not.

**Note**: The image alpha is used to cutout the texture and only project the arrow onto the mesh.

<img src="/img/projects/texture_instancing/arrow.jpg" width="256" height="256" alt="Input texture to be sampled">


Following image shows the output generated by texture instancing process. The texture is applied on the geometry using Tri-Planar projection. Standard instancing behaviour is emulated while computing the projections, meaning the texture will be facing the Z axis defined by the orienation attribute (similar to copying geometry on points using an **orient** attribute).

The contribution of each point is computed using a inverse distance weighting ie. contribution of each point is directly proportional to the proximity from the point being sampled on the mesh. The farther the point is the lower its contribution will be and conversely the closer a point is the higher its contribution is.

Orientation is visualised as 3 axes (XYZ) at each point position.

![Texture instanced onto the input grid](/img/projects/texture_instancing/texture_instanced_no_orient.png)

In this test, we only look up a single point to determine texture which means, we will have a clear distinction between each point's region of contribution (visualised as different colors on the grid).


In contrast to the above example, we now add noise to the orientation. The arrow head correctly orients itself to the specified attribute controlling the orientation.

![Input point cloud](/img/projects/texture_instancing/texture_instance_01_orient.png)


Now lets try this on geometry slightly more exciting than a planar grid. In the following example, we use texture instancing to compute the colour contribution of 4 different textures applied to the input geometry (note that the geometry has no UVs). The 4 different textures are visualized on the top left side of the video.

Another, key point to keep in mind is that there are no UVs on the mesh. The algorithm uses Tri-Planar projection algorithm to generate an implicit UV map which is used for sampling textures.

<iframe width="420" height="236" src="https://www.youtube.com/embed/AnYkQN3xzjo?autoplay=1&loop=1&playlist=AnYkQN3xzjo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> 

As we are now looking up more than one point, the colour contribution of each point is computed and result is set as the new colour on the mesh. Note the blending that happens between different textures is handled using inverse distance weighting, meaning, for each point in the mesh, closest found point in the point cloud will have maximum influence.

Extending this further, we will also hook in our displacement maps specific to each of the 4 colour maps to compute the displacement at each point of the mesh. In the following example we use texture instancing to compute displacement along with colour using 4 different textures. All the other parameters (scale, orientation, blending exponent etc) are same as previous example.

<iframe width="420" height="236" src="https://www.youtube.com/embed/hg-PO1tSHZA?autoplay=1&loop=1&playlist=hg-PO1tSHZA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

If you look closely in the above example, you will see that the silhouette is being broken by displacements which are computed using the various displacement maps as defined by the point cloud.


## Conclusion


Its important to note that each technique comes with its own advantages and disadvantages. Texture instancing in itself is not a magic bullet solution for all your procedural texturing needs! Its a tool with specific set of strengths.

* It allows for a convenient way of adding variation and visual complexity in terms of textures.

* It doesnt require the geometry to have UV maps.

* It is really convenient if the look of the geometry needs to be modified based off a trigger event.

* Really powerful tool to blend together different textures to add complexity.

* Helps in breaking the tiling patterns visible in large scale environments.


However, it also comes with its own weaknesses.

* Its difficult to art direct. For specific aesthetics, your better off painting custom textures.

* The fact that my implementation relies on Tri-Planar Projection (TPP is probably the most suitable algorithm here anyway), means its slower than standard shaders.

* It adds another dependency (point cloud) into the mix. This is not necessarily a drawback but does requrie decent pipeline support to track dependencies.

* The texture scaling might get tricky as we can only affect it using a global multiplier (which would affect all textures) and a per point scale multiplier.

* If displacements of different maps being used are not normalized or within reasonable range or each other, there may be artefacts introduced due to the blending.


<h2>Improvements</h2>


For the second development sprint, there are few more features I would like to add.

One of the things I want to implement next is depth based blending which is quite standard in game engines such as Unreal Engine. An example of this can be seen in this [article](https://www.gamasutra.com/blogs/AndreyMishkinis/20130716/196339/Advanced_Terrain_Texture_Splatting.php).

Additionally, considering large landscapes are quite a common use case for this, I would like to add a planar mode for computing UV space to speed up the computations.

Expose few texture sampling filters (box, gaussian, blackman, sinc etc) as parameters for users.
