<!--
.. title: Step 4: Enter The DunGen
.. slug: step-4-dungeongen
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

[dungeonmoon]: 
[2darray]: 
[arghints]: 
[playeraddcamera]: 
[roomwalls]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/roomwalls.png
[roomhalls]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/roomhalls.png 
[startpos]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/startpos.png



In this step, we will be creating a random dungeon generator for our game. We'll be making our generator in the form of a self-contained, "modular" script which will be loaded as a singleton. By creating our dungeon generator this way, you could easily import this script into any other project which requires a similar system, with a minimal amount of integration required.  

![dungeonmoon][dungeonmoon]

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
We're not doing much yet, but let's break it down. The first thing we do is call this magic golden function `randomize()`, which will cause all random number generation afterward become truely random. Because of some mysterious quirk of Godot, random numbers will be  generated in the same sequence every time you play your game, until `randomize()` is called (or a seed is defined, but we wont get into that). Anyway...
Next, we're defining a new empty array called `map`. This will become a 2D array which will hold the indices (0 or 1) which will tell our Map which of its tiles should be Walls and which tiles should be Floors. Finally, we're returning the currently-empty and useless array back to the caller.  

### It's Arrays All The Way Down
There are several methods we could use to hold our `map` data. Since our dungeon floorspace will always be a regular rectangle made of of tiles we'll want to refer to by X Y coordinates (Battleship-style), a *multi-dimensional array* will be ideal. To be specific, we'll use a two-dimensional array. If you're not yet familiar enough with this sort of data structure, it might be hard to wrap your head around at first (it was for me, at least). But keep at it! Make sure your code is correct, and in the end it should Just Work. I think once you see everything working together in the end, the concept will start to click. And once that happens, the idea of a multi-dimensional array is actually pretty simple, if a little abstract.  

![2darray][2darray]

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
Our default values will also describe to us what type of variable we're expecting for each of our arguments. If you're writing your scripts in Godot's built-in script editor, you should get a nice tooltip to appear when you type a call to this function, telling you what types of vars it wants for each of the arguments, along with their name. This is pure gold for anyone other than the author of the script who is trying to use it, and even super-helpful to us as well. It's easy to forget what your own code is doing, so reminders like this save a lot of time and effort in the long run.  

![arghints][arghints]

To populate our `map` array, we iterate over the length of `map_size.x`. For each iteration "row", we create a new list "column". We then iterate over the length of `map_size.y` and assign the value of `wall_id` that many times into the column array. The column is then appended to the map array, and the iteration begins again for the new value of `x`. Since we're representing our walls with indices (map tiles are defined by index), we'll define index `0` as our wall_id.   

### We Have Rooms To Grow
By now, we've effectively created for ourselves a dungeon consisting of solid rock which isn't very interesting. From this solid block, we want to carve out spaces to represent rooms and hallways. To do this, we need to add some more arguments to our function. The order of these *are* important:  

```python
func Generate( map_size=Vector2(80,70), room_count=35, room_size=Vector2(5,16), wall_id=0, floor_id=1 ):
```

We've added three new arguments:  
`room_count` will be the number of times we attempt to generate a room.  
`room_size` defines the minimum and maximum width and length a room should have  
`floor_id` defines the value we want to define as "a floor", just like with `wall_id`  

Let's add the code to generate the rooms:  

```python
func Generate( map_size=Vector2(80,70), room_count=35, room_size=Vector2(5,16), wall_id=0, floor_id=1 ):

    # Populate map array
	for x in range( map_size.x ):
		var column = []
		for y in range( map_size.y ):
			column.append( wall_id )
		map.append( column )
  
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
**For our game proper, we'll be rolling our own RNG functions which will be much more elegant to work with. But in the spirit of keeping this script independent from the rest of the program, we'll opt for this ugly approach. It will probably be the one and only time we'll be doing this `int(round(rand_range()))` madness.**  

Because we want our rooms to fit within the boundaries of our map's size, we'll roll our room's width and height first. All we're doing here is getting two random numbers `w`idth and `h`eight, which will be between `room_size.x` and `room_size.y`. To get the position of the room on the map, we roll two more numbers `x` and `y` which are between `0` and `map_size.x` and `.y` respectively. The maximum numbers to roll are then offset by subtracting `w` and `h` respectively. Because our map indices begin at 0, we also want to make our max numbers one less than our size.  
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
	for i in range( rooms.size() ):
		var room = rooms[i]
		# Carve room
		for x in range( room.size.width - 2 ):
			for y in range( room.size.height - 2 ):
				map[ room.pos.x + x + 1 ][ room.pos.y + y + 1] = floor_id
```
Much like we did when we created our "blank" map, we're using this double-for loop, iterating through Y through X. This time though, we already have a value assigned to the index we're going for. Instead, we're accessing that x y position in our data grid and assigning it a new value -- our `floor_id` argument. The math here looks a little wonky, but it's there for a reason. We're carving our rooms in such a way that the outer border of the room is not carved. This ensures that every room has at least one tile of wall surrounding it, otherwise rooms which are right next to each other but not overlapping, would have no barrier between the edge where they meet. This means that you should consider that your `room_size` parameters will be two tiles shorter than the numbers you give it.  
In fact, we can compensate for that in our code, by adding two to the numbers we give our RNG for width and height of the room! Let's back up to that part of the code and give it a tweak:  

```python
		var w = int( round( rand_range( room_size.x + 2, room_size.y + 2 ) ) )
		var h = int( round( rand_range( room_size.x + 2, room_size.y + 2 ) ) )
```  
Now we don't need to worry about it!  A room given the size of `(3,3)` will now have a `3x3` floor, not a `1x1` floor. Take a tile here...add a tile there...the math should all work out in the end.  
![roomwalls][roomwalls]


### Happy Hall Hacking
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
			var A = center( room )
			var B = center( prev_room )
      
			# Flip a coin..
			if randi() % 2 == 0:
				#carve vertical -> horizontal hall
				hline( A.x, B.x, A.y )
				vline( A.y, B.y, B.x )
			else:
				#carve horizontal -> vertical hall
				vline( A.y, B.y, A.x )
				hline( A.x, B.x, B.y )
```  
Now, this code as it is, is broken, so don't try running anything at this point. This incomplete code will be an outline for our hall-carving process. What we're doing, is getting the center tiles of both the current room, and the previous room. We want to carve a hallway between these two points. The catch though is, these two points could be at any two points on our map. The only way we could connect these with straight lines would be to create something like an L shaped hallway: either go left/right from A and carve up/down to B, or go up/down from A and carve left/right to B.  

We're calling a few functions here that don't yet exist, so let's remedy that.  

Write these functions down at the bottom of our script, below `Generate()`. Generate is our big mother function, so we want to keep her at the top for maximum visibility. The first helper function, `center()` will find the center cell within a Rect2 we'll give it as its argument:  

```python
# Find the global center of a Rect2
func center( rect ):
	var x = ceil(rect.size.width / 2)
	var y = ceil(rect.size.height / 2)
	return rect.pos + Vector2(x,y)
```  
We then have two other helper functions; `hline()` and `vline()`. These will return to us an array of Vector2 map coordinates representing the "line" of our hallway. They both do very similar things, but are different enough to warrant two different functions:  

```python
# Get Vector2 along x1 - x2, y
func hline( x1, x2, y ):
	var line = []
	for x in range( min(x1,x2), max(x1,x2) + 1 ):
		line.append( Vector2(x,y) )
	return line

# Get Vector2 along x, y1 - y2
func vline( y1, y2, x ):
	var line = []
	for y in range( min(y1,y2), max(y1,y2) + 1 ):
		line.append( Vector2(x,y) )
	return line
```  
Now that we have functions that do stuff, let's refactor our broken code so it's no longer broken!  

```python
		...
		else:
			# Carve a hall between this room and the last room
			var prev_room = rooms[i-1]
			var A = center( room )
			var B = center( prev_room )
			
			# Flip a coin..
			if randi() % 2 == 0:
				# carve vertical -> horizontal hall
				for cell in hline( A.x, B.x, A.y ):
					map[cell.x][cell.y] = floor_id
				for cell in vline( A.y, B.y, B.x ):
					map[cell.x][cell.y] = floor_id
			else:
				# carve horizontal -> vertical hall
				for cell in vline( A.y, B.y, A.x ):
					map[cell.x][cell.y] = floor_id
				for cell in hline( A.x, B.x, B.y ):
					map[cell.x][cell.y] = floor_id
```  

![roomhalls][roomhalls]  
*Rooms A, B and C are generated in order. The two different colored "paths" represent either of the two options we give our hall-carver to make a passageway between it and the room preceding it.*  

Now, one piece of information we'd like our generator to make for us will be a starting position for the dungeon. We can keep things simple, and designate the center of the first generated room to be our starting point.  
First, we want to add another local var to our `Generate()` function, up where we declared `map` and `rooms`:  

```python
	# initialize data
	var map = []
	var rooms = []
	var start_pos = Vector2()
```  
Now, where we were skipping over the first room in our hall-carver block, we can instead have it use our new `center()` function to define our `start_pos`:  

```python
		...
		if i == 0:
			# Define the start_pos in the first room
			start_pos = center( room )
		else:
			# Carve a hall between this room and the last room
			var prev_room = rooms[i-1]
			var A = center( room )
			var B = center( prev_room )
```  

![startpos][startpos]  

We're almost done!  

The last operation of our `Generate()` function will return the `map` grid back to the sender. But now we have more information we want to return. We'll do this by packaging all the data into a dictionary and returning that. For now, all we need is `map` and `start_pos`, but later we will also want to access our `rooms` array, so we'll include that in our dictionary also.  The end of Generate will look like this:  

```python
	return {
		"map":		map,
		"rooms":	rooms,
		"start_pos":	start_pos,
		}
```  
With this, our DunGen script is complete, at least for now! From here, it wont take much to have our game utilize our new awesome tool!  
**[Here is the complete DunGen.gd script](https://gist.github.com/YeOldeDM/2cfdf25fb6a34d88c7ec67b385cffada)**.  

## Putting DunGen To Work
Bring yourself now back to your Map.gd script. Down in its `_ready` function, we can have it call our DunGen singleton:  

```python
# Init
func _ready():
	var data = DunGen.Generate()
```  
Since we defined default values for all our generation parameters, we don't need to pass any arguments to the function!  

Now, we need a little function in Map that will take the `map` data we've generated and use it to paint the tiles on our TileMap. We'll call this little guy `draw_map()`, and place it at the bottom of the script, but above `_ready`:    

```python
# Draw map cells from map 2DArray
func draw_map( map ):
	for x in range( map.size() ):
		for y in range( map[x].size() ):
			set_cell( x,y, map[x][y] )
```  

Once we have that, we can call it in `_ready` after we get our data:  

```python
# Init
func _ready():
	var data = DunGen.Generate()
	draw_map( data.map )
```  

The final step is to spawn the player into the map. We already have all this set up, we just need to feed it our `start_pos` data from the generator:  

```python
# Init
func _ready():
	var data = DunGen.Generate()
	draw_map( data.map )

	# Spawn Player
	var player = RPG.make_thing( "player/player" )
	spawn( player, data.start_pos )
```  

### Who Is Camera?!
If you try playing your game now, assuming it doesn't crash, you'll probably notice that you don't see your player on the screen! It will be very likely that we've spawned our player at a position which is currently outside our current viewport. This is because there was a very small (but very important) task I forgot to do in the last step! What we need to do is add a Camera object to our player.  

![playeraddcamera][playeraddcamera]

Open up your Database scene. Select your Player node, and add a new `Camera2D` node to it as a child. Under the camera's properties, enable `Current` and disable both `Drag Margin` checkboxes. You can play with these settings to get the setup you prefer though, of course. Just make sure the Camera is set to "Current", or our game wont use it.  
When we add the Camera, it defaults to center on the origin of the node it's parented to. The origins of our Things are in their upper-left corner, which isn't the true visual center of the object. You can offset this by changing the camera's Pos property to offset it by half our cell size, which would be `(16,16)`.  *The Ghost of Future Prototypes have reported that we will be having to re-create the Things in our database scene soon, so don't spend too much time making your settings here perfect.*  

**Now** try playing your game! Your player should show up at center screen. Move the player around, and the camera follows automagically!  

## Conclusion
Get Excited! We've just given our game the ability to procedurally generate a whole world for us! At this point, you might find yourself spending more time playing your game than you do working on it, and that is a good thing! Try giving your Generate function different arguments and see what results you can get. If you're feeling adventurous, you can try making changes to your generator and experiment with it!  

*In the next step, we'll expand on our map's `draw_map()` function, add a whole mess of new tiles to our dungeon tileset in a painless way, and paint our dungeons up to make them a lot more interesting to look at. Once you're ready to move on, [step 5 is just around the corner!](../step-5-tilefamily.html)*
