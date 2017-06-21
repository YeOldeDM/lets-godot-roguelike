<!--
.. title: Step 4: Enter The DunGen
.. slug: step-4-dungeongen
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS



In this step, we will be creating a random dungeon generator for our game. We'll be making our generator in the form of a self-contained, "modular" script which will be loaded as a singleton. By creating our dungeon generator this way, you could easily import this script into any other project which requires a similar system, with a minimal amount of integration required.  

Let's begin the step by creating a new script file at `res://global/DunGen.gd`. Let's also add this script to the AutoLoad list in Project Settings now, so we don't forget to do it later!  

## Mother Method Most Mysterious
The Bulk of our DunGen script will consist of one large function we'll call `Generate()`. We'll give this function a bunch of arguments which will define all the variables the generator will need to generate its map, such as the size of the map, the number of rooms to generate, and so on. With that information, our generator should produce and return the data our Map needs to build our dungeon. We'll begin our function with just the bare bones we need to have our function do something. We'll then mature our function in small logical iterations, talking through each of the new things we introduce.  

### Let's Start!
Let's give birth to our new baby function `Generate()`!  Start with this:  
```python
extends Node

# Generate the Datamap
func Generate():
  
  # Randomize
	randomize()
  
  # initialize data
	var map = []
  
  # return data
  return map
```  
We're not doing much yet, but let's break it down. The first thing we do is call this magic golden function `randomize()`, which will cause all random number generation afterward become truely random. Because of some mysterious quirk of Godot, random numbers will be  generated in the same sequence every time you play your game, until `randomize()` is called (or a seed is defined, but we wont get into those). Anyway...
Next, we're defining a new empty array called `map`. This will become a 2D array which will hold the indicies (0 or 1) which will tell our Map which of its tiles should be Walls and which tiles should be Floors. Finally, we're returning the currently-empty and useless array back to the caller.  

### It's Arrays All The Way Down
There are several methods we could use to hold our `map` data. Since our dungeon floorspace will always be a regular rectangle made of of tiles we'll want to refer to by X Y coordinates (Battleship-style), a *multi-dimensional array* will be ideal. To be specific, we'll use a two-dimensional array. If you're not yet familiar enough with this sort of data structure, it might hard to wrap your head around at first (it was for me, at least). But keep at it, make sure your code is correct, and in the end it should Just Work. I think once you see everything working together in the end, the concept will start to click. And once that happens, the idea of a multi-dimensional array is actually pretty simple, if a little abstract.  

Let's expand on our `Generate()` function to have it construct for us one of these fancy 2D arrays for our `map` var. We'll also need to introduce some arguments for our generator to take. We'll give all these arguments *default values*, so we will be able to call `Generate()` without necessarily needing to fill in all the arguments if we don't want to.  

```python
extends Node

# Generate the Datamap
func Generate( map_size=Vector2(80,70), wall_id=0 ):
  
  # Randomize
	randomize()
  
  # initialize data
	var map = []
  
  # Populate map array
	for x in range( map_size.x ):
		var column = []
		for y in range( map_size.y ):
			column.append( wall_id )
		map.append( column )
  
  # return data
  return map
```  
Our default values will also describe to us what type of variable we're expecting for each of our arguments. If you're writing your scripts in Godot's built-in script editor, you should get a nice tooltip to appear when you type a call to this function, telling you what types of vars it wants for each of the arguments, along with their name. This is pure gold for anyone other than the author of the script who is trying to use it, and even super-helpful to us as well. It's easy to forget what your own code is doing, so reminders like this save a lot of time and effort.  
To populate our `map` array, we iterate over the length of `map_size.x`. For each iteration "row", we create a new list "column". We then iterate over the length of `map_size.y` and assign the value of `wall_id` that many times into the column array. The column is then appended to the map array, and the iteration begins again for the new value of `x`. Since we're representing our walls with indicies (map tiles are defined by index), we'll define index `0` as our wall_id. In theory, wall_id could be anything!  

### We Have Rooms To Grow
By now, we've effectively created for ourselves a dungeon consisting of solid rock. Not very interesting. From this solid block, we want to carve out spaces to represent rooms and hallways. To do this, we need to add some more arguments to our function. The order of these *are* important:  

```python
func Generate( map_size=Vector2(80,70), room_count=35, room_size=Vector2(5,16), wall_id=0, floor_id=1 ):
```

We've added three new arguments:  
`room_count` will be the number of times we attempt to generate a room.  
`room_size` defines the minimum and maximum width and length a room should have  
`floor_id` defines the value we want to define as "a floor", just like with `wall_id`  

Let's add the code to generate the rooms:  

```python
  ...
  
  # Populate map array
	for x in range( map_size.x ):
		var column = []
		for y in range( map_size.y ):
			column.append( wall_id )
		map.append( column )
  
  # Generate Rooms
	for r in range( room_count ):
		# Roll Random Room Rect
		# Width & Height
		var w = int( round( rand_range( room_size.x, room_size.y ) ) )
		var h = int( round( rand_range( room_size.x, room_size.y ) ) )
		# Origin (top-left corner)
		var x = int( round( rand_range( 0, map_size.x - w - 1 ) ) )
		var y = int( round( rand_range( 0, map_size.y - h - 1 ) ) )
		# Construct Rect2
		var new_room = Rect2( x, y, w, h )
  
  # return data
  return map
```  

Here we have our first use of random number generation! It might look weird at first to be doing this, but here is why:  
`rand_range(a,b)` returns a random `float` between a and b. We want all these values to be integers. Since these random values will eventually be used to reference an index in an array, it *needs* to be an integer type.  
We could use `int( rand_range(a,b) )` but that would round our floats down to the nearest integer. This would favor low numbers over higher ones, which isn't fair.  
You'd think that `round()` would return an integer, but it doesn't *(needs confirmation?)*. `round(3.14)` will return `3.0` and not `3`. We want `3`. So we need to round our random float, then convert to `int`. Whew!  
**For our game proper, we'll be rolling our own RNG functions which will be much more elegant to work with. But in the spirit of keeping this script independent from the rest of the program, we'll opt for this "brute force" approach. It will probably be the one and only time we'll be doing this `int(round(rand_range()))` madness.**  

Because we want our rooms to fit within the boundries of our map's size, we'll roll our room's width and height first. All we're doing here is getting two random numbers `w`idth and `h`eight, which will be between `room_size.x` and `room_size.y`. To get the position of the room on the map, we roll two more numbers `x` and `y` which are between `0` and `map_size.x` and `.y` respectively. The maximum numbers to roll are then offset by subtracting `w` and `h` respectively. Because our map indicies begin at 0, we also want to make our max numbers one less than our size.  
We end the chunk of code by declaring a new variable `new_room` and assigning it to a `Rect2()` object. Just like we use `Vector2` to define a cell position on our map, we can use `Rect2` to define a rectangular area of cells on our map. We can create a Rect2 object by giving it our `x`, `y`, `w`, and `h` random values.  

Now that we've rolled up the abstract definition of our room rectangle, we need to do something with it. We need to store our `Rect2` rooms so we can use them later. Let's declare a new array `rooms` at the top of the function, just under `var map = []`:  

```python
	# initialize data
	var map = []
	var rooms = []
```  
We also want to make sure our rooms aren't overlapping each other. `Rect2` has a method built into it that makes this task fairly easy. Add this code within the same scope as the "Generate Rooms" block:  

```python
    ...
		# Construct Rect2
		var new_room = Rect2( x, y, w, h )
		
		# Check against existing rooms for intersection
		if !rooms.empty():
			var passed = true
			for other_room in rooms:
				# If we don't intersect any other rooms..
				if new_room.intersects( other_room ):
					# Add to rooms list
					passed = false
			if passed: rooms.append(new_room)
		# Add the first room
		else:	rooms.append(new_room)
```
If we're generating our first room, we have no other rooms to check against for overlap, so it goes straight into the array. For every room after, we create a test. We iterate through every other room we've created so far, and checking for overlap using `new_room.intersects( other_room )`. If any of these calls returns true, that room fails the test. Only rooms which pass the test get invited to the party.  

### Colossal Cubic Cave Carving
Now we have a list of all the non-intersecting rooms our dungeon will contain. It's time for us to use these to carve the floors of those rooms into our `map` data. Here is where our 2D array comes into play:  

```python
    ...
					passed = false
			if passed: rooms.append(new_room)
		# Add the first room
		else:	rooms.append(new_room)
	
	# Process generated rooms
	for i in range( rooms.size() - 1 ):
		var room = rooms[i]
		# Carve room
		for x in range( room.size.width - 2 ):
			for y in range( room.size.height - 2 ):
				map[ room.pos.x + x + 1 ][ room.pos.y + y + 1] = floor_id
```
Much like we did when we created our "blank" map, we're using this double-for loop, iterating through Y through X. This time though, we already have a value assigned to the index we're going for. Instead, we're accessing that x y position in our data grid and assigning it a new value -- our `floor_id`. The math here looks a little wonky, but it's there for a reason. We're carving our rooms in such a way that the outer border of the room is not carved. This ensures that every room has at least one tile of wall surrounding it, otherwise rooms which are right next to each other but not overlapping, would have no barrier between the edge where they meet. This means that you should consider that your `room_size` parameters will be two tiles shorter than the numbers you give it.  
In fact, we can compensate for that in our code, by adding two to the numbers we give our RNG for width and height of the room! Let's back up to that part of the code and give it a tweak:  

```python
		var w = int( round( rand_range( room_size.x + 2, room_size.y + 2 ) ) )
		var h = int( round( rand_range( room_size.x + 2, room_size.y + 2 ) ) )
```  
Now we don't need to worry about it!  A room given the size of `(3,3)` will now have a `3x3` floor, not a `1x1` floor. Take a tile here...add a tile there...the math should all work out in the end.  



### Horrible Hall Hacking
Our rooms, by design, should be completely cut off from each other. Unless we want to make a digging-based dungeon crawler (neat idea, but out of the scope of our project) we need some way to connect these rooms together, so the player can, you know, crawl through them.  
We carve our hallways much the same way we did our rooms. But halls are not special like rooms. We don't need to keep them in any list. We don't care if hallways intersect other hallways, or even other rooms! The halls should be able slash their way freely through our dungeon, giving us an interesting result which is highly variable, but usually doesn't look too "random".  
Within the same scope as "Process generated rooms", add this code. It will be the skeleton of our hall-carving algorithm:  

```python
		if i == 0:
			# First room
			pass
		else:
			# Carve a hall between this room and the last room
			var prev_room = rooms[i-1]
			var A = _center( room )
			var B = _center( prev_room )
      
			# Flip a coin..
			if randi() % 2 == 0:
				#carve vertical -> horizontal hall
				_hline( A.x, B.x, A.y )
				_vline( A.y, B.y, B.x )
			else:
				#carve horizontal -> vertical hall
				_vline( A.y, B.y, A.x )
				_hline( A.x, B.x, B.y )
```  
Now, this code as it is, is broken, so don't try running anything at this point. 



### Private Parts
We'll also have a couple small helper functions, which will be meant to be called *only* through our `Generate()` function. To reinforce this, we'll be writing these as *private functions*, which have their names prefixed with an underscore `_`. We've already dealt with private functions already. Every script we create has a `_ready()` function defined, which is a private function that is meant only to be called internally by the game engine. You should, in most cases, never see a call to `_ready()` written anywhere in any script. Functions which are not private are considered *public functions*:  

```python
# Public Function
func add(a,b):
  return a+b

# Private Function
func _add(a,b):
  return a+b
```
Know that there is really no such thing as privacy in gdscript. There is really nothing stopping you from calling `DunGen._private_func()` from anywhere else in the program; the private designation is more of a suggestion than a hard restriction. If making these helper functions public feels better to you, it will make no difference.  

## Putting DunGen To Work

## Conclusion
