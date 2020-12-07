---
layout: post
title: legos - An asset management system.
subtitle: Prototype and early PDG/Houdini integration.
header-img: ../img/projects/legos/legos_pdg_header_mod.jpg
# date: 2020-12-01
tags:
- Asset Management System
- Graph Database
- neo4j
- Cypher
- houdini
- PDG
- Procedural Dependency Graph
---

## Background

This time last year I started writing an Asset Management System(AMS) **legos** for a long term personal project. The system would be central to the pipeline I am developing in tandem to support the production of the project. Having worked at all the major VFX facilities in London and studied their pipelines closely, I realised that a good AMS (including tooling required to integrate it into the pipeline and various DCCs) is key to having a robust and scalable pipeline for VFX production.

Almost all the major studios have their own implementation of an AMS, which sits at the heart of their pipeline. Some are state of the art while others are decent. However, an AMS in not a magic bullet solution, its a tool which requires to be properly designed and intergrated into the pipeline. A lot of places get this part wrong. They design an AMS but its integration into the pipeline and user interactivity is quite lacking; leading to obnoxius workflows which steal away the time artists should be spending on aesthetic decisions to make the work look better.

I wanted to put together all the things I have learnt and implement an AMS, ancilliary tooling such as content browsers and a pipeline which would help me finish my own projects efficiently, run meaningful analytics to figure out which tasks in the production are causing bottlenecks.
Additionally, having the ability to visually see the topology/relationships between the entities in the database is a massive plus. 
The fact that Neo4j is a native graph database, it lends itself quite nicely to the Object Oriented design of **legos**. This is one of the reasons why I chose [Neo4J](https://neo4j.com/product/neo4j-graph-database/) as a primary database.


## Introduction

**legos** is an asset management system built specifically for managing assets generated in the course of a VFX project. It is written completely in Python 3 and leverages Neo4j's graph database technology as its primary database workhorse. **legos** builds upon simple constructs such as Assets, Packages, Relationships to quickly build and track complex dependencies.

Without going into too much detail in the interest of brevity, following are few common terms we will encounter in following sections of this article.

* **Asset** - all the CG content generated during production of a show, is an asset. Asset has several specializations which give additional meaning and context to them. Such as CameraAsset, LightAsset, FileAsset etc. I might use the term Asset, Element, Digital Content interchangeably in following sections.

* **Package** - is technically also an asset. However, more appropriately its a collection of Assets which are closely related or meant to be used in conjunction (as a convention, however if there are any use cases where you need to package different asset types together legos allows for that too). Packages too have several specializations such as LookDevPackage, ShotPackage, CharacterPackage etc.

* **Relationships** - define how the entities interface with each other (these also have special meaning within the database and can allow us to run quite interesting analytics).

* **Revisions** - one of the typical VFX jargons you hear a lot in VFX discussions is Dailies. Revisions are dailies submissions within the **legos** ecosystem. Revisions too have few specializations such as QCRevisionSub, AssetRevisionSub etc (we will only look at QCRevisionSub in this example).

Now that we have few terms out of the way, lets have a look at a typical production use case.

Following is a lookdev test I did using the amazing Skaarj fanart model created by **[Ben Erdt](https://www.artstation.com/artwork/q24my)**.

{% include image.html url="/img/projects/legos/skaarj_forest_env_test_1280.gif" description="Skaarj Look dev, rendered using Houdini's Karma renderer." %}

In a typical VFX pipeline, any CG element which needs to be added to a shot, goes through numerous look development cycles with constant adjustment requests/notes from the client. It is vital that all the information that went into generating an image (or an asset), is tracked in the database to be able to regenerate the approved version, requiring correct version of each an every asset that was used to generate approved content.

Quite a few assets were used to generate the above image. We can deconstruct it into the following ingredients:

* 3D model

* Textures
    * Albedo map
    * Bump map
    * Normal map
    * Roughness map
    * Reflectivity map

* Materials (which will be using above textures)
    * Material/look bindings to the model components (head, hair, armour etc)

* Lights
    * forest light rig
    * Studio light rig (to calibrate the materials in a nuetral look dev environment)
    * HDR for Image based lighting

* Camera

* Render settings (each lookdev iteration can have different render settings which might have huge implications for the look)

In the above example I did not use the Skaarj rig to pose the character or add any animation so there are no dependencies on those components. The screenshot below shows these components and the relationships they have with each other.

{% include image.html url="/img/projects/legos/rv_sub_and_deps.png" description="QC Render and asset dependencies in the database." %}

All of the above content is generated by artists from separate departments, using different/specialized DCCs. These assets are then checked into the database. In the context of above example , the model and textures were created by [Ben Erdt](https://www.artstation.com/benerdt) which basically meant that the data had to be ingested into **legos**. This would then bring the data into the AMS. This process is highlighted by the green box in the image below. Additionally, the textures are converted into ACESCG colourspace and converted to **.rat** files which are optimised for Houdini and support mipmaps.

Look development stages are highlighted in the image below in red. A typical lookdev workflow is to calibrate and/or build the materials in a neutral studio lighting environment while constantly checking the look in the intended lighting environments (forest, indoors, sunset etc). Its not uncommon for each hero character/asset to have a dedicated lookdev environment. In the above example, we do this by using a **LookDevPackage**. This LookDevPackage contains all the assets which decide the look of a CG element, including the model. It also contains lookdev cameras and light rigs which are used while developing the look and is meant to be self sufficient; meaning, if you were to import Skaarj LookDevPackage into a shot, and hit render, you should get the same image we have above.

This can be observed from the image above. All the assets are shown as yellow nodes, LookDevPackage in the node in green. If you look closely, the relationships clearly show that the Skaarj LookDevPackage contains 2 light rigs (a studio lighting env and a forest env), a camera rig (which may contain multiple cameras for closeup or mid range view and different profiles), materials, material bindings, Skaarj model and QC render settings asset.

{% include image.html url="/img/projects/legos/legos_pdg_graph_skaarj.png" description="legos integration with Houdini's PDG. The section of graph within the green box shows texture ingestion into legos and conversion into ACESCG colourspace. The section in blue shows ingestion of model and creation of variants. Section of the graph in red is where the look dev light rigs, camera rigs are created and published into the database. Materials are also generated/built here using the textures and are bound to the model." %}

The table below shows all the assets which were used to render the QC for the Skaarj LookDevPackage.

{% include image.html url="/img/projects/legos/qc_dep_data.png" description="Asset dependencies data. These are all the assets used to produce the QC render. The table shows the asset names, uids, versions, users and time of creation of each of the asset dependencies." %}

## Supported DCCs

At the moment, **legos** is pretty well integrated with Houdini's Task Operators (TOPs). As with everything, there's always scope for improvement, however I believe there's more value to be gained by also integrating with LOPs and implementing a shotbuilder.

## Roadmap

Before the production of the project actually starts, I have a needs-list (if there ever was such a thing!). A minimum viable product that I need.

* Asset Browser - well this is quite critical. There's no point in managing assets/packages if you cant browse them, in some sensible manner. This will be a PyQT based stand alone application which will allow user to browse generated content and also view all the metadata available (in a visually informative way).

* Shot builder for Houdini (and possibly Blender).

* Complete suite of CLI tools - at the moment I have ingestion tools implemented, however its also super handy to write CLI for probing tools.

* Complete Houdini integration
    ** Integration with TOPs
    ** Integration with LOPs

* Basic Blender integration

All of these are quite wide ranging projects and it will be fun to see how much of this I can get done before the deadlines I have actually set!


{% include image.html url="/img/projects/legos/legos_pdg_header.jpg" description="" %}

## Database

I will not pretend to be a database expert, truth be told, this was my first encounter with database modeling. I first tried out MySQL since its fairly straight-forward to use. After a few attempts, I realised I was having a really hard time getting a good grasp of how to best organise the tables in the database for optimal performance. I had spent a lot of time and effort to design the object models for **legos** and deconstructing them into a bunch of tables just didn't feel right.

Spending a few days researching databases, I came across [Neo4j](https://neo4j.com/product/neo4j-graph-database/) a Graph database which was also opensourced (community edition). After looking at a few youtube workshops, I was able to translate the object model I had for **legos** into the database models. Its Object Oriented Design approach really helped me(a complete Database newbie) to model an underlying database for **legos**.

Anyone who has used Neo4j, will probably not be able to stop talking about **Cypher**, the query language for Neo4j. With good reason. It is bloody awesome! Cypher really made querying database enjoyable for me. More on this a bit later.

## Database modeling

legos leverages Neo4j's graph database technology as its primary database. This allows legos to execute complex queries and also run analytics using graph algorithms effortlessly.

Following image shows a very simplified schema of the database model. Though a schema is not required in Neo4j but I find it helpful.

One big challenge I faced was versioning of assets/entities in the database. In a traditional database, it would probably be just another entry in the a bunch of tables. Not so straight forward in Neo4j . and there are techniques to implement versioning within Neo4j with slight modifications to the object models and schemas. This however is beyond the scope of this article.

{% include image.html url="/img/projects/legos/simple_schema.png" description="Simplified schema for demonstration purposes, showing how the various entities in legos are related." %}


{% include image.html url="/img/projects/legos/all_nodes.png" description="View of the graph database showing relationships between several entities in legos. Neo4J is a graph database which actually stores data as graphs and is not a layer sitting on top of a regular database." %}


Ability to run graph algorithms for analysis.

Less rigid due to dynamic relationships (new relationships can be created very easily).

Aligns well with the OOP approach.

