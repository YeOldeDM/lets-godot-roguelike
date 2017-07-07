<!--
.. title: Step 4: Tile Families
.. slug: step-5-tilefamilies
.. date: 2017-06-26 05:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

Last time, we created all the tools we need to give our game a randomly-generated dungeon. In this step, we're going to jazz up our map a little to include random varients of commonly-themed tiles. We are also going to download and utilize a plugin to help us get through a lot of the grunt work we'll be putting ourselves up against. After the last step, this one should feel quick and easy, so let's get right to it!  

## We Need a Bigger Boat, and by Boat I mean Tileset
The first thing we want to do is expand upon our current tileset.  
So far we've only been using two tiles; one for floors and one for walls.  
Open `res://core/Map/Tiles_edit.tscn`. Delete tile nodes under the base node.  


### Get All The Graphix!
Now we want to take all the wall and floor tile graphics from our *source folder* and copy them into our projects `res://graphics/dun/` folder. Under `/wall` and `/floor`.  
There are some "unique" graphics we're not interested in. Don't include those.  
This will be a lot of assets. Don't panic!  

## Tile! Set! Help! Errr!!
Making tilesets is easy and straightforward. But tedius if you have lots of tiles...  
Go to the **AssetLib** tab at the top of the screen. You can find all kinds of different plugins for your projects here!  
Search for **"Tilesethelper"**. Download and install from Godot. Or, [here's the link](). Don't forget to enable it once installed.  

This is how to use Tilesethelper to import all our Wall tiles. Give these LightOccluders. We'll want them soon.  
Do the same to import all floor tiles. All they need is the Sprites...  

## Big Chunky Global Family
Make a new Singleton `res://global/TileFamily.gd`. Prepare to crunch.  
Start with some group *constants*. These will be arrays corresponding to the index ranges covered by each group.  
A Family consists of one wall group and one floor group. Define these as var array[floor, wall] for different pairs of constants.  

## Repainting The Map
Instead of linking map data directly with map index, we need a layer of complextion.  
Declare a const in Map for MAP_FAMILY and ROOM_FAMILY.  

In `Map.paint_cell()` we want a new argument, `family=MAP_FAMILY`.  
In the `paint_cell()` function, we get a random index between the two numbers in the family array.  

### Paint Rooms
We can paint rooms in a different family from our hallways, to give the rooms more character.  
After painting the map, we can iterate through rooms and re-paint them with a different family.  
`paint_rect()` does this for us.  

### Random Room Flavors
We can randomize the family of each room. Do it!  




## Conclusion  
Body.  
*Invitation to [the next step!](link)*
