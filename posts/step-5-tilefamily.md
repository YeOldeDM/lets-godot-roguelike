<!--
.. title: Step 4: Tile Families
.. slug: step-5-tilefamilies
.. date: 2017-06-26 05:00:00 UTC
.. type: text
-->
  
[tilehelper_search]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/5/tilehelper_search.png


# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

Last time, we created all the tools we need to give our game a randomly-generated dungeon. In this step, we're going to jazz up our map a little to include random varients of commonly-themed tiles. We are also going to download and utilize a plugin to help us get through a lot of the grunt work we'll be putting ourselves up against. After the last step, this one should feel quick and easy, so let's get right to it!  

## We Need a Bigger Boat, and by Boat I mean Tileset
What I'm trying to say is we need to expand upon our current tileset. I tried to come up with a witty title for this section, but fudge it! We have a game to make here.
The paltry two tiles we're using have served their purpose well, but it is their time to go.  

So far we've only been using two tiles; one for floors and one for walls. This works well for quickly giving us a visual description of a cell's state (wall or floor). This looks dull though, and our source graphics library gives us a great deal of wall and floor textures to work with. The temptation is too great to supress anymore...  
If you look at the contents of `/dc-dngn/wall/` and `dc-dngn/floor/` you'll notice that many of the tiles are one of a handful of similar-looking textures which are all variations of some common theme; The textures are meant to be used together. A "Texture Family", if you will.

Time to get to work. Open `res://core/Map/Tiles_edit.tscn`. Delete tile sprite nodes under the base node.  


### Get All The Graphix!
Before we continue, we want to take a whole bunch of textures from our source folder and copy them over to our project's graphics sub-folder.  
Take the contents of the source folder `dc-dngn/wall/` and copy it to `res://graphics/dun/wall/`. Do the same with `dc-dngn/floor/` and copy it to `res://graphics/dun/floor`. We really don't need *all* these files though.  

There are some "unique" graphics we're not interested in. Don't include these. Basically, we are only using the graphics which end in a number. We are also excluding the `shadow` and `tree` wall sets, and the `dirt` and `pedestal` sets from floors. We only want the graphics which form neat square textures, and belong in some set of textures.  
*(maybe provide a zip with all the appropriate wall/floor graphics)* 

This will be a lot of assets. Don't panic!  
You probably aren't looking forward to creating all these sprites and assigning their textures individually. Neither am I. Fortunately for us, we can use a tool to do this ditch-digging work for us. Better yet, we don't even have to make it. Someone has already done that for us!  


## Tile! Set! Help! Errr!!
Making tilesets is easy and straightforward. But tedius if you have lots of tiles. The project for this tutorial has 295 tiles!  


Along the top of the editor, where you have the 2D/3D/Script tabs, there is a fourth tab called AssetLib. Go to the **AssetLib** tab at the top of the screen. You can find all kinds of different plugins for your projects here!  

Search for the plugin **"Tilesethelper"**. Download and install from Godot. Or, [here's the link](). Don't forget to enable it once installed.  
You can enable plugins by going to the Plugins tab of your Project Settings. There is also a Plugins button at the top-right of the AssetLib panel that will bring you to the same place.  

![][tilehelper_search]

Go to your tileset edit scene if you're not already there. It should now be an empty `Node` since we deleted the two tiles we made back in Step 1.  
Select the top Node of this scene. With our Tilesethelper plugin enabled, we should now have a new tab in one of our docks called "Tileset". In here we have some options to play with.  
Click the `PNG` icon in the dock. This should bring up a dialog window to let you select one or more images. Navigate to your `res://graphics/dun/wall/` folder and select *all* the graphics there. The PNG page image should now turn into a stack of PNG images, indicating we want to create several tiles from several images.  
It is important that we fill in the Size field in the tileset dock. This will tell the helper what size we want our tiles to be. All our tiles are 32x32, so we want to set `32` as our Size. We don't need to worry about filling in `Name` or `Frame`.  
Below this are three more options we can assign to our tiles. It might seem logical to give our walls Collision, but remember that our own data-driven collision system is already in place. Our "Navigation" system will also be hand-rolled to suit our purposes, so we have no need for the Navigation on tiles. We do however, want to use the mysterious `Occluder`. This is something we wont really need until later on in the project, but it will do us good to set it up now so we don't have to bother re-making our tileset later.  
Later on, we'll be implementing some real-time lighting and so we want our wall tiles to cast shadows, and the Occluder will make this happen. We'll be going into this when we get to it, so if you don't quite understand what this Occluder business is yet, don't worry. Just know that it will look really cool. Just enable the option in the TileSet dock.
When, and only when, all these options are set the way we want, double-check you have the top node of your tileset scene selected, and hit the `Create Tile(s)` button. Say some more magic words and watch the plugin auto-generate a bunch of tiles. Yay!  

Now we can do the same thing for all our floor textures in `res://graphics/dun/floor/`. This time though, we don't want to assign Occluders, so make sure that is disabled. Select all the floor textures, make sure your Size is still 32, and hit `Create Tile(s)` again. BAM! All our floor tiles are appended to the top node of the tileset scene.  

Before we can use any of these tiles, we need to export the scene as a tileset resource. `Click Scene > Convert To.. > Tileset` and replace the tileset you created earlier.

**NOTE: MAKE SURE to disable the "Merge with Existing" option at the bottom of the convert to tileset window. If you don't, it will conserve the original tiles we deleted at the beginning, which will shift all your tiles' indices by 2. That's bad.**  

Now if you go to the Map scene and select the TileMap node there, you should see all the new tiles you added in the dock on the left.  
This is going to totally brick the way our tilemap paints its tiles based on the data we give it, so we need to do some re-factoring to implement our new dynamic set of tiles.  

## Big Chunky Global Family
Make a new Singleton `res://global/TileFamily.gd`. Prepare to crunch. Unfortunately, we don't have a tool to dig this ditch for us.  
If you've used exactly the same textures used in this tutorial, and have turned them all into tile nodes via the tilesethelper plugin, then the indices of your tiles should correlate with [this](). You should be able to copy and paste this into your code without shame.  

**Start with some group *constants*. These will be arrays corresponding to the index ranges covered by each group.  

```python
const WALL_BRICK_BROWN_VINES = [0,3]
const WALL_BRICK_BROWN = [4,11]

const FLOOR_BOG = [161,164]
const FLOOR_COBBLE_BLOOD = [165,176]
```  

**A Family consists of one wall group and one floor group. Define these as var array[floor, wall] for different pairs of constants.  

```python
var FAMILY_BRICK_BROWN = [FLOOR_PEBBLE_BROWN, WALL_BRICK_BROWN]
var FAMILY_BRICK_GREY = [FLOOR_RECT_GREY, WALL_BRICK_GREY]
var FAMILY_BRICK_DARK = [FLOOR_GREY_DIRT, WALL_BRICK_DARK]
```


## Repainting The Map
**Instead of linking map data directly with map index, we need another layer of complexity.

**Declare a const in `Map.gd` for MAP_FAMILY and ROOM_FAMILY.  
```python
extends TileMap

var MAP_FAM = TileFamily.FAMILY_BRICK_BROWN
```

**In `Map.paint_cell()` we want a new argument, `family=MAP_FAMILY`.  
```python
func paint_cell( cell, family=MAP_FAM ):
	var c = self.data.map[cell.x][cell.y]
	...
```

In the `paint_cell()` function, we get a random index between the two numbers in the family array.  
```python
func paint_cell( cell, family=MAP_FAM ):
	var c = self.data.map[cell.x][cell.y]
	var i = -1
	# If cell is wall..
	if c == 0:
		i = RPG.rnd( family[1][0], family[1][1] )
	# If cell is floor..
	elif c == 1:
		i = RPG.rnd( family[0][0], family[0][1] )
	set_cellv( cell, i )
```  
We're refactoring most of this function. First we declare a variable `c` as the value of the cell we're looking at. We also declare a variable `i` to hold the index we want to set that cell to in the tilemap. We then check if the cell is a wall or a floor. For each condition, we have it set `i` to a random index in the tile family constant we give it for the `family` argument.  

### Paint Rooms
**We can paint rooms in a different family from our hallways, to give the rooms more character.  

**After painting the map, we can iterate through rooms and re-paint them with a different family.  

Let's create a new function which will take a Rect2 `rect` argument and a `family`. This will go through the rectangle on the data map and re-paint those tiles to be those from the `family` we give it here:
```python
func paint_rect( rect, family=TileFamily.FAMILY_SANDSTONE ):
	for x in range( rect.pos.x, rect.end.x ):
		for y in range( rect.pos.y, rect.end.y ):
			paint_cell( Vector2(x,y), family )
```
Since our rooms are already defined as Rect2 objects, we can feed those directly into this function. To make this work, we add this code to our `paint_map()` function:  
```python
# Draw map cells from map 2DArray
func paint_map():
	assert self.data != null
	for x in range( self.data.map.size() ):
		for y in range( self.data.map[x].size() ):
			paint_cell( Vector2(x,y) )
	
	for room in self.data.rooms:
		paint_rect( room )

```

### Random Room Flavors
**We can randomize the family of each room.  

**Create an array of room families.  
```python
extends TileMap

var MAP_FAM = TileFamily.FAMILY_BRICK_BROWN

var ROOM_FAMS = [
	TileFamily.FAMILY_STONE_GREY,
	TileFamily.FAMILY_STONE_BROWN,
	TileFamily.FAMILY_BRICK_GREY,
	]
```

**Change the argument we give the `paint_rect()` call to pick a random entry from the family array.  

```python
	for room in self.data.rooms:
		var fam = ROOM_FAMS[ randi() % ROOM_FAMS.size() ]
		paint_rect( room, fam )
```

## Conclusion  
That's it! Play your game now, and you should be presented with a much more visually-appealing dungeon! Our little system is also flexible and scalable just by declaring new `FAMILY_` constants in our `TileFamily` singleton. 

*Invitation to [the next step!](link)*
