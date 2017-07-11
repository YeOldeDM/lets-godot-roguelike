<!--
.. title: Step 4: Tile Families
.. slug: step-5-tilefamilies
.. date: 2017-06-26 05:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

Last time, we created all the tools we need to give our game a randomly-generated dungeon. In this step, we're going to jazz up our map a little to include random varients of commonly-themed tiles. We are also going to download and utilize a plugin to help us get through a lot of the grunt work we'll be putting ourselves up against. After the last step, this one should feel quick and easy, so let's get right to it!  

## We Need a Bigger Boat, and by Boat I mean Tileset
**The first thing we want to do is expand upon our current tileset.  
The paltry two tiles we're using have served their purpose well, but it is their time to go.  

**So far we've only been using two tiles; one for floors and one for walls.  
This works well for quickly giving us a visual description of a cell's state (wall or floor). This looks dull though, and our source graphics library gives us a great deal of wall and floor textures to work with.  
If you look at the contents of `/dc-dngn/wall/` and `dc-dngn/floor/` you'll notice that many of the tiles are one of a handful of similar-looking textures which are all variations of some common theme; The textures are meant to be used together.  

**Open `res://core/Map/Tiles_edit.tscn`. Delete tile sprite nodes under the base node.  


### Get All The Graphix!
**Now we want to take all the wall and floor tile graphics from our *source folder* and copy them into our projects `res://graphics/dun/` folder.  
Take the contents of the source folder `dc-dngn/wall/` and copy it to `res://graphics/dun/wall/`. Do the same with `dc-dngn/floor/` and copy it to `res://graphics/dun/floor`.  

**There are some "unique" graphics we're not interested in. Don't include those.  

**This will be a lot of assets. Don't panic!  
You probably aren't looking forward to creating all these sprites and assigning their textures individually. Neither am I. Fortunately for us, we can use a tool to do this ditch-digging work for us. Better yet, we don't even have to make it. Someone has already done that for us!  


## Tile! Set! Help! Errr!!
**Making tilesets is easy and straightforward. But tedius if you have lots of tiles...  
The project for this tutorial has 295 tiles...  


**Go to the **AssetLib** tab at the top of the screen. You can find all kinds of different plugins for your projects here!  

**Search for **"Tilesethelper"**. Download and install from Godot. Or, [here's the link](). Don't forget to enable it once installed.  

**This is how to use Tilesethelper to import all our Wall tiles. Give these LightOccluders. We'll want them soon.  

**Do the same to import all floor tiles. All they need is the Sprites...  

**It's important to disable "Merge with Existing" option at the bottom of the convert to tileset window. If you don't, it will conserve the original tiles we deleted at the beginning, which will shift all your tiles' indices by 2. That's bad.  

## Big Chunky Global Family
**Make a new Singleton `res://global/TileFamily.gd`. Prepare to crunch. Unfortunately, we don't have a tool to dig this ditch for us.  
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
**Instead of linking map data directly with map index, we need a layer of complextion.  

**Declare a const in Map for MAP_FAMILY and ROOM_FAMILY.  
```python
extends TileMap

var MAP_FAM = TileFamily.FAMILY_BRICK_BROWN
```

**In `Map.paint_cell()` we want a new argument, `family=MAP_FAMILY`.  
```python
func paint_cell( cell, family=MAP_FAM ):
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
