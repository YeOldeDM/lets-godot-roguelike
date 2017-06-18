<!--
.. title: Step 4: Enter The DunGen
.. slug: step-4-dungeongen
.. date: 2017-06-18 04:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

In this step, we will be creating a random dungeon generator for our game. Things like this can be incredibly complex and involve all kinds of advanced concepts. But we will be making something that is pretty basic and easy to work with, and still produces interesting results.  

## DunGen
Create a new AutoLoad "DunGen" at `res://global/DunGen.gd`. Unless stated otherwise, all code being written in this step will be in this file.

### Concepts
This step will be covering a few new concepts we haven't run into yet in our project. So let's go over them real quick:  

#### RNG  
Random Number Generation. The meat & potatos of any respectable roguelike. RNG is the programmatic equivelant to rolling dice, flipping a coin, drawing a number from a hat, or whatever. Most of the random numbers in our game will come in the flavor of picking a random integer between two other integers (pick a number between 1 and 10), or flipping a coin ( a 50% chance to do A, or do B ).  

#### 2D Array  
An array is a list of stuff. An item in a list can be just about anything...even another list! A list of lists gives us a way to express data in a grid, or a `two-dimensional array`. 

#### Rect2  
With Godot, we have a built-in class called Rect2, which defines the position and dimensions of a simple rectangle. In this step, we'll be using the power of Rect2 to define our dungeon's rooms, which will be wonderfully rectangular in shape!  




### Digital Dice, and How To Roll Them
So far, we've managed to build a lot of Roguelike stuff without having to use any random numbers. Not anymore!  
Since Godot doesn't provide a built-in solution for the kind of RNG we need, we want a helper function:  

```python
# Return a random int between 'n' and 'm'
func rnd( n,m ):
	# in case our args are out of order..
	var low = min( n,m )
	var high = max( n,m )
	# Return random int from low to high
	return  randi() % int( high - low + 1 )  + low
```  


### Mother Method Most Mysterious
Our script should work in such a way that we only need to call one function of DunGen in order for it to do its thing. DunGen in turn will work entirely within itself, and return a package of data representing our generated dungeon. Keeping our code seperated like this prevents us from turning our game's structure into spaghetti. We like big chunky meat-ball code, not sloppy noodle code.  
Another big benefit of working like this, is that it will make it much easier for us to bring this code into other projects where we would want a random dungeon generator, and easily integrate it into that project. Recycling code is a good thing!  

We want some parameters we can feed to Generate. By providing default values to these arguments in our function declaration, we can call `Generate()` without any arguments, or only *some of the arguments*.  
Vector2 map_size  
Vector2 room_size  
int room_count
var floor_id  
var wall_id  

We also need a variable to store our map grid, and an array of Rect2s representing our rooms. We'll need to share this data between the different functions we'll be writing in our script, so we want to keep these variables in our script's global scope. Declare these variables at the top of the script, above `rnd()`:  

```python
extends Node


# Global vars
var map = []
var rooms = []


# Return a random int between 'n' and 'm'
func rnd( n,m ):
	..
```

Now, below `rnd()`, we'll begin by defining the `Generate()` function, giving it the parameters we defined above as its arguments, and giving them all default values:  

```python
# Generate the Datamap
func Generate( map_size=Vector2(120,100), room_count=50, room_size=Vector2(5,16), wall_id=0, floor_id=1 ):
	randomize()
	# Initialize an empty map
	var start_pos = Vector2()
	self.map = []
	self.rooms = []
	
	# Build a blank map
	for x in range( map_size.x ):
		var col = []
		for y in range( map_size.y ):
			col.append( wall_id )
		self.map.append( col )
  
  # Return the map
  return self.map
```

This will create a blank map, in the form of a 2D array, which is filled with the value we've defined for `wall_id`. Other functions in our script will be "carving" rooms and halls into this solid map by assigning the value of `floor_id` to the appropriate places in the map array.  



### Rooms
Now that we have something to carve in to, we want to be able to generate rooms. The rooms in our dungeons will all be rectangles, defined as Rect2. We'll create two new functions to handle our room generation. Place these under `rnd()` and above `Generate()`:  

```python
# Define a rectangle of room_size dimension,
# at a position within the map rect
func create_room( map_size, room_size ):
	# Roll room width/height
	var w = rnd( room_size.x, room_size.y )
	var h = rnd( room_size.x, room_size.y )
	# Roll room X/Y origin 
	var x = rnd(0, map_size.x - w-1)
	var y = rnd(0, map_size.y - h-1)
	# Return a Rectangle
	return Rect2( x, y, w, h )
```

Create_room will use our `rnd()` function along with the arguments passed to the function to roll up a random rectangle which should fit within the boundries of the map.  

```python
# Set floor_id to cells in a Room rect
func carve_room( room, floor_id ):
	for x in range( room.size.x - 2 ):
		for y in range( room.size.y - 2 ):
			self.map[room.pos.x+x+1][room.pos.y+y+1] = floor_id
```

Carve_room will take a `room` Rect2 that was created by `create_room()` and use it to carve a rectangle of `floor_id` values into the map data. We're doing this in such a way that we're leaving a 1-cell border or walls around our room rectangles. This ensure that rooms that happen to land next to each other will still be seperated by a wall.  

Now, we can have our `Generate()` function use these functions to generate some rooms. We will try `room_count` times, generating a room and checking if it intersects any existing rooms. If a new room does intersect an old room, that room is rejected and the function tries again.    

Add this code to the end of `Generate()`, but above the last `return self.map` line (make sure that line stays at the end of the function):  

```python
	# Generate Rooms
	for r in range( room_count ):
		var new_room = create_room( map_size, room_size )
		
		var passed = true
		if not self.rooms.empty():
			for room in self.rooms:
				if new_room.intersects( room ):
					passed = false
		if passed:
			self.rooms.append( new_room )
```
The `Rect2` class has a built-in `intersects( other_rect )` method which makes this task easy on us.  
Once we've created our array of Rooms, we can iterate through those and carve them into the map:  

```python
	# Carve Rooms
	for i in range( self.rooms.size() - 1 ):
		var room = self.rooms[i]
		carve_room( room, floor_id )
```



#### Room Center
We're going to use the centers of our room rectangles to use as the start and end points of our connecting hallways. The `Rect2` class doesn't have a built-in function to find its center, so we'll have to write one. This can live up near the top of the script, just underneath `rnd()`:  

```python
# Get the center of a rectangle
func rect_center( rect ):
	var x = ceil( rect.size.width / 2.0 )
	var y = ceil( rect.size.height / 2.0 )
	return Vector2( rect.pos.x + x, rect.pos.y + y )
```

### Halls
Just like we did with Rooms, we're going to define a couple (three, actually) functions to help us carve hallways into our map. This block of code can go between `carve_room()` and `Generate()`:  

```python
# Set floor_id to cells along a vertical line
func carve_h_hall( x1, x2, y, floor_id ):
	for x in range( min(x1,x2), max(x1,x2) + 1 ):
		self.map[x][y] = floor_id


# Set floor_id to cells along a horizontal line
func carve_v_hall( y1, y2, x, floor_id ):
	for y in range( min(y1,y2), max(y1,y2) + 1 ):
		self.map[x][y] = floor_id


func carve_hall( A, B, floor_id ):
#	# flip a coin..
	if randi()%2 == 0:
#		# Carve horizontal then vertical
		carve_h_hall( A.x, B.x, A.y, floor_id )
		carve_v_hall( A.y, B.y, B.x, floor_id )
	else:
#		# or Carve vertical then horizontal
		carve_v_hall( A.y, B.y, A.x, floor_id )
		carve_h_hall( A.x, B.x, B.y, floor_id )
```

As we carve rooms into our map, we also want to carve hallways that connect those rooms together.  
After carving the first room, we get the center point between the new room, and the previously-created room. We then call `carve_hall()` using those two points.  
When we carve the first room, we'll designate its center as our player start_pos, instead of carving a hall. Change the bottom part of `Generate()` to look like this:  

```python
# Carve Rooms
	for i in range( self.rooms.size() - 1 ):
		var room = self.rooms[i]
		carve_room( room, floor_id )
		if i > 0:
			# Carve halls between rooms
			var new_center = rect_center( room )
			var prev_center = rect_center( self.rooms[i-1] )
			
			carve_hall( prev_center, new_center, floor_id )
		else:
			# Set the Dungeon start position
			start_pos = rect_center( room )
```

Since we also want to return the start_pos variable, we'll change our return call to return a dictionary. This way we can pass both peices of data (and more when we decide we need it) in one tidy package:  

```python
	#	# Return map dictionary
	return {
		"map":			self.map,
		"start_pos":	start_pos,
		}
```

Here is the complete DunGen.gd script:  

```python
extends Node


# Global vars
var map = []
var rooms = []


# Return a random int between 'n' and 'm'
func rnd( n,m ):
	# in case our args are out of order..
	var low = min( n,m )
	var high = max( n,m )
	# Return random int from low to high
	return  randi() % int( high - low + 1 )  + low


# Get the center of a rectangle
func rect_center( rect ):
	var x = ceil( rect.size.width / 2.0 )
	var y = ceil( rect.size.height / 2.0 )
	return Vector2( rect.pos.x + x, rect.pos.y + y )


# Define a rectangle of room_size dimension,
# at a position within the map rect
func create_room( map_size, room_size ):
	# Roll room width/height
	var w = rnd( room_size.x, room_size.y )
	var h = rnd( room_size.x, room_size.y )
	# Roll room X/Y origin 
	var x = rnd(0, map_size.x - w-1)
	var y = rnd(0, map_size.y - h-1)
	# Return a Rectangle
	return Rect2( x, y, w, h )


# Set floor_id to cells in a Room rect
func carve_room( room, floor_id ):
	for x in range( room.size.x - 2 ):
		for y in range( room.size.y - 2 ):
			self.map[room.pos.x+x+1][room.pos.y+y+1] = floor_id


# Set floor_id to cells along a vertical line
func carve_h_hall( x1, x2, y, floor_id ):
	for x in range( min(x1,x2), max(x1,x2) + 1 ):
		self.map[x][y] = floor_id


# Set floor_id to cells along a horizontal line
func carve_v_hall( y1, y2, x, floor_id ):
	for y in range( min(y1,y2), max(y1,y2) + 1 ):
		self.map[x][y] = floor_id


func carve_hall( A, B, floor_id ):
#	# flip a coin..
	if randi()%2 == 0:
#		# Carve horizontal then vertical
		carve_h_hall( A.x, B.x, A.y, floor_id )
		carve_v_hall( A.y, B.y, B.x, floor_id )
	else:
#		# or Carve vertical then horizontal
		carve_v_hall( A.y, B.y, A.x, floor_id )
		carve_h_hall( A.x, B.x, B.y, floor_id )


# Generate the Datamap
func Generate( map_size=Vector2(120,100), room_count=50, room_size=Vector2(5,16), wall_id=0, floor_id=1 ):
	randomize()
	# Initialize an empty map
	var start_pos = Vector2()
	self.map = []
	self.rooms = []
	
	# Build a blank map
	for x in range( map_size.x ):
		var col = []
		for y in range( map_size.y ):
			col.append( wall_id )
		self.map.append( col )
	
	# Generate Rooms
	for r in range( room_count ):
		var new_room = create_room( map_size, room_size )
		
		var passed = true
		if not self.rooms.empty():
			for room in self.rooms:
				if new_room.intersects( room ):
					passed = false
		if passed:
			self.rooms.append( new_room )
	
	# Carve Rooms
	for i in range( self.rooms.size() - 1 ):
		var room = self.rooms[i]
		carve_room( room, floor_id )
		if i > 0:
			# Carve halls between rooms
			var new_center = rect_center( room )
			var prev_center = rect_center( self.rooms[i-1] )
			
			carve_hall( prev_center, new_center, floor_id )
		else:
			# Set the Dungeon start position
			start_pos = rect_center( room )

	#	# Return map dictionary
	return {
		"map":			self.map,
		"start_pos":	start_pos,
		}
```

It's important this script correct, and there are a few spots in the code that are easy to get mixed up (especially if you're a little dyslexic like me!). So if you want to avoid future breakage, go through your code if you need to and double/triple check your work. 

### Putting It All Together 
Our Map script will use this package of data to create the game's world and spawn the player at some logical starting point.  

Because it makes our work easier, we're going to encapsulate thes functionality into its own function and call it `draw_map()`:  

```python
# Draw map cells from map 2DArray
func draw_map( map ):
	for x in range( map.size() - 1 ):
		for y in range( map[x].size() - 1 ):
			set_cell( x,y, map[x][y] )
```
Just to be clear, this is being added to `Map.gd`. We are done working with `DunGen.gd` for now..  
After all the scripting we just did on the other script, this should feel like a breeze! All this is doing is taking our 2D array and setting the map's cells to match the values in the array.  


## The Art of Building Without Building
We can generate dungeons now. We can package that map data and give it to our Map. Our Map can take this data and draw a real map based on that. The last step is to bring these together. This is also pretty easy to do. Change the Map script's `_ready` function. We also have a `start_pos` stored in our data variable also, so let's use that to spawn the player and set its position:  

```python
# Init
func _ready():
	var data = DungeonGen.Generate()
	draw_map( data.map )

	# Spawn Player
	var player = RPG.make_thing( "player/player" )
	spawn( player, data.start_pos )
```  
### Camera
Bam! Run your game, and there is a pretty good chance that your player will spawn on some point on the map that's outside our viewport. This is no good, but easy to fix!  
We need a way to have our game follow the player around, keeping him in the center of our view. Godot makes this easy enough it almost hurts.  
Hop over to your Database scene, where you have your Player object. Select this node (the instance in Database, not the Thing base scene!), and add a new `Camera2D` node to that. A camera is made so it will follow whatever node it's parented to. Before the camera will actually be used, you need to enable its "Current" property. Play the game now, and you should see your player at center screen, no matter where you end up spawning in your map.  
A couple little tweaks before we are done with this. When we added the camera to the player node, it defaults to track the player's origin, which if you remember is the upper-left corner. We want to track the player's center, so set the camera's Position (its farther down the inspector list) to be half of what our cell size is ( 16,16 if you're following along ). We also want to disable Drag Margins. You may or may not want to enable Smoothing; try it both on and off, and with various smoothing values, and see what looks best for you. Personally, I think it looks best with smoothing enabled, and with a fairly fast smoothing rate of 15 or so. You can also play with your camera's Zoom settings here. If those 32x32 tiles are looking too tiny to you, try setting the Camera's Zoom factors to 0.5, making the pixels in its view "twice as large".  

## Conclusion
Get excited! You can now run around and explore your procedurally-generated dungeon! You can play around with your generator's parameters and get lots of different results!  
That's going to be it for this step. In the next step, we'll expand on this and have our code fill our dungeon's rooms with Things. We'll also modify our Map so it paints random variations of a set of tiles, so our dungeon doesn't look so blah!  
 
