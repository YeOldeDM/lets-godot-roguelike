<!--
.. title: Step 0: So You Want To Make A Roguelike
.. slug: step-0-introduction
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

### Why Godot?
Here's a list, chief:
* Godot is FREE. As in, free free
* It's super-small and lightweight (~30mb), it'll run on a potato
* It's cross-platform, and supports many devices
* Awesome dedicated 2D engine (hard to find in a 3D-heavy world)

### Why a roguelike?
Because I want to :)  But also, it is a genre which is easily approachable to the newcomer to game dev, while also being a chance to create
a "big game". Because of the procedural nature of such a game, very large worlds can be created with a minimal amount of time being put into
asset creation and world design (which can be a huge issue if you're a one-member team). The rigid, grid-based world mechanics along with a turn-based time control lets us work within an environment
which is, while limited, much easier to control and develop on.  The Roguelike is a prime candidate for doing a lot with a little.

### What do I need to start this thing?
First off, you'll need the newest stable version of the [Godot Engine](https://godotengine.org/). This tutorial will be written using ver 2.1.3.
It is highly recommended that you also download the Demo Projects as well as the Export Templates, though neither is required 
for this tutorial.  
This project will also be using the graphics set Dungeon Crawl Stone Soup which can be found [here](https://opengameart.org/content/dungeon-crawl-32x32-tiles). You can, of course, use whatever graphics
you wish (as well as create your own), but if you really want to follow along you will want to download a copy of this and extract it 
somewhere handy.  There will also be a few 'custom' graphics I have made for the project, for the occasional things we'll need that 
the graphics pack is lacking. Those will be made available as Free For Any Use.  
Copies of the project will also be available for download and use for reference. Snapshots of the project at the end of each Step can be found [here](https://github.com/YeOldeDM/realms-of-todog).  

[Click here once you're ready to begin!](../step-1-setup.html)  
