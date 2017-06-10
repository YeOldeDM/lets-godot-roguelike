<!--
.. title: Complete Roguelike Tutorial using Godot
.. slug: articles
.. date: 2017-06-1 01:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

Introduction
=====

In this series of articles, we will be going through the process of creating a simple roguelike, using the Godot Game Engine.  

### Who is this tutorial for?
Who this tutorial is made for, is those who have aquired the first base skills required to make a game, but lack the knowlege of how to
put those skills to use to craft a "full game". Games are big and complex things, and the prospect of doing such a thing can seem like 
an almost impossible task.  Hopefully, this will serve as a guide to help those accomplish such a task by using a method of Rapid Iterative 
Design, Failing Faster, and Keeping It Simple.

This tutorial will be geared toward those who already have at least a basic experience with Godot and how to navigate its UI.

This project will also be very scripting-heavy, and will assume you have at least a basic knowlege of general programming, although no
experience with *gdscript* (Godot's scripting language, which is similar to *Python*) should be required in order for you to follow along. I will 
be trying to touch on concepts as we encounter them in the project, but going into such things in detail would make this thing way too long.
There are a ton of other resources out there that cover these things, which are written by people who are certainly much smarter than me.  

Here are a few resources containing many tutorials and how-tos covering different aspects of the Godot engine. Any aspect of the engine we work with in this tutorial is most likely covered more in-depth somewhere within these links:  

[Godot 101](https://www.youtube.com/playlist?list=PLsk-HSGFjnaFISfGRTXxp65FXOa9UkYc5) by KidsCanCode  

[30 Tutorials In 30 Days](https://www.youtube.com/playlist?list=PLhqJJNjsQ7KEr_YlibZ3SBuzfw9xwGduK) and [Learn the Godot Engine and Create a Platform Game](https://www.youtube.com/playlist?list=PLhqJJNjsQ7KEbSXHacP9eD37xyoPJz9gm) by GDquest  

[Godot Game Engine Tutorial Series](http://www.gamefromscratch.com/page/Godot-Game-Engine-tutorial-series.aspx) by Gamefromscratch (these have become pretty outdated, but still worth looking at)  

[Godot Docs Step-by-Step](http://docs.godotengine.org/en/stable/learning/step_by_step/index.html)  

### Why Godot?
Here's a list, chief:  
-- Godot is FREE. As in, free free  
-- It's super-small and lightweight (~30mb), it'll run on a potato  
-- It's cross-platform, and supports many devices  
-- Awesome dedicated 2D engine (hard to find in a 3D-heavy world)  

### Why a roguelike?
Because I want to :)  But also, it is a genre which is easily approachable to the newcomer to game dev, while also being a chance to create
a "big game". Because of the procedural nature of such a game, very large worlds can be created with a minimal amount of time being put into
asset creation and world design (which can be a huge issue if you're a one-member team). The rigid, grid-based world mechanics along with a turn-based time control lets us work within an environment
which is, while limited, much easier to control and develop on.  The Roguelike is a prime candidate for doing a lot with a little.  

![old game screen](https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/oldgame.png)  
*A screenshot from an older prototype of this project. Our final product here should be an improvement on this.*  

### What do I need to start this thing?
First off, you'll need the newest stable version of the [Godot Engine](https://godotengine.org/). This tutorial will be written using ver 2.1.3.
It is highly recommended that you also download the Demo Projects as well as the Export Templates, though neither is required 
for this tutorial.  

### This project will also be using the graphics set from **Dungeon Crawl Stone Soup** which can be found [here](https://opengameart.org/content/dungeon-crawl-32x32-tiles).
You can, of course, use whatever graphics you wish (as well as create your own), but if you really want to follow along you will want to download a copy of this and extract it somewhere handy.  There will also be a few 'custom' graphics I have made for the project, for the occasional things we'll need that the graphics pack is lacking. Those will be made available as Free For Any Use.  
Copies of the project will also be available for download and use for reference. Links to a snapshot of the project as it is at the end of each step is available at the end of each article.  

The Steps
=====

[Step 1: Learning To Crawl](../step-1-setup.html)  
[Step 2: Learning To Walk](../step-2-collision.html)  
[Step 3: Enter The Thing](../step-3-things.html)  


This tutorial is currently a work in progress. New articles will be added to this list as they are made, so stay tuned!  



# Credits
This tutorial series is based upon and inspired by [this tutorial by Jotaf](http://www.roguebasin.com/index.php?title=Complete_Roguelike_Tutorial,_using_python%2Blibtcod)  
I'd also like to shout out special thanks to Verbalshadow, Zygliplix, ChuuniMage, and everyone else who has helped make this thing happen.  

