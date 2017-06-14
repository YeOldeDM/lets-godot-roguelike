<!--
.. title: Step 3: Enter The Thing
.. slug: step-3-things
.. date: 2017-06-11 04:00:00 UTC
.. type: text
-->

[extendsvader]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/extendsvader.png 
[databasescene]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/databasescene.png
[dathing]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/dathing.png
[globalcloud]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/globalcloud.png
[mapxy]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/mapxy.png
[step3fin]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/step3fin.png
[fuuu]: http://www.officialpsds.com/images/thumbs/FUUUUU-2-psd81145.png
[editchildren]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/editchildren.png


In the last step, we added the all-important collision system to our game. Our player is feeling kind of lonely though, so in this step we will introduce the system we will be using to populate our dungeon with everything that isn't the dungeon itself.


### The Thing

![dathing]  

#### Generalization
Our game would become a lot more interesting to play if we fill it with lots of interesting things to interact with.  
We will want to build our system in such a way that we're able to create complex objects out of a collection of simple general parts. 
This way, we can make a lot of content with just a little work, and ensure that common behavior is consistent among all objects.  

Let's start by defining our most basic type of object and give it a descriptive name. I'll dub this object a `Thing`. 

Everything inside the dungeon beside its walls and floors is going to be a `Thing`. We'll begin bringing our new Thing to life by creating a scene with some nodes and a script, much like we did for the Player. Let's begin!  

#### Player Refactor
Actually, we're going to create our Thing by cloning our Player scene. It has all the same nodes we'll need, mostly set up the way we want them. The easiest way to clone a scene is to `File > Save As..` and save the scene in a different location under a different filename.   
Open your `res://core/Player/Player.tscn` if it's not already, and re-save the scene as `res://things/Thing.tscn`.  
I said the Player scene was mostly set up the way we want for our Thing. There are only a couple changes you need to make:  
First, rename top node to "Thing". A simple change, but an easy one to look over. You also want to clear the script assigned to the Thing top node, and create a new one. If you scroll to the bottom of your scripted node in the inspector, you can find the Script parameter, where you can click the drop-down menu and clear the script. You can also give the Thing's Icon a suitable texture. In the system we'll create around Things, an object which is just "A Thing" and not something which extends Thing (we'll get to this next) is essencially a "Prop", a dumb object whose only purpose is to occupy its position on the map. So find a suitable graphic to serve this purpose. I chose a generic stone alter at `/dc-dngn/altars/dngn_altar.png`. This scene will be a kind of template, everything that is created out of it will get its own Icon texture, so your choice in this texture is really no big deal.  

We popped a fresh script into our Thing, but we haven't done anything with it yet. Before we do, let's think for a minute about what our script needs.  
Our most basic Thing script requires only two things so far: a way to get its map position, and a way to set its map position.  

Actually, this is code we've already written. It's in the `Player.gd` script as `get_map_pos()` and `set_map_pos( cell )`. Take this portion of the Player script: 

```python
# Map node
onready var map = get_parent()

# Get our position in Map Coordinates
func get_map_pos():
	return map.world_to_map( get_pos() )

# Set our position to Map Coordinates
func set_map_pos( cell ):
	set_pos( map.map_to_world( cell ) )
```  
Copy this code and paste it into the Thing script. Once you do that you can delete the same code from the Player script. It's good practice to preserve any code you move around like this at its original source, until you're sure the original code doesn't belong there anymore. Re-writing bad code to make it good code feels great. Re-writing good code that gets lost just sucks.  


Now, your Player script should only contain its `_ready` and `_input` functions and their contents, as well as our custom `step()` function. If you tried playing your game at this moment, all your movement code should be broken, of course. We just ripped a bunch of code out of our script, and now the remaining code is trying to call functions that no longer exist! Why the hell would we ever do that??  
Remember that you moved all that code into another script. It's almost as if we're setting ourselves up to have multiple scripts assigned to a single node. But as a rule, *a node in the godot engine can only have one script assigned to it*. So it must be non-possible for our Player node to utilize both its Player script and a Thing script, right? Not quite!  

##### Extends
There has been one line in all our scripts that we've basically been ignoring up to this point, but it's a very important line (the most important, one could argue). The `extends` keyword tells your script which functionality it has access to, and is almost always identical to the type of node the script has been created for. The script on a Node `extends Node`. A TextureFrame node's script `extends TextureFrame`. You can think of these as extending your script to use the functions of some higher code. Even though we never defined a `get_pos()` or `set_pos()` function for our player, those functions still do things, because our player extends the methods of Node2D, which *does* define a `get_pos()` and `set_pos()`.  

By using a pinch of Gogo Magic, we can tell our player script to extend one of our own scripts, instead of `Node2D`.

Roll up your sleeves and say the magic words: change the first line of Player.gd so it `extends "res://things/Thing.gd"`!  

Play your game now, and your player works again. Tada!  

Player will inherit all members and methods of Thing. It also has its own members and methods that are unique to the Player.  
All other types of things will also inherit Thing, or inherit a script which eventually inherits Thing.  


![][extendsvader]  


### The Database
*(Special Thanks to camelCase for introducing this concept. I hope they don't mind if I borrow it!)*

Now that we've got The Most Important Thing re-created, we can start thinking about all the other Things we'll eventually want in our game. We need some kind of central database of Things where we can create new Things and easily maintain Things we've already created. There are many ways we could tackle this issue. We could use a `data-driven` approach and create a big dictionary of Thing data in a global script. We could take a `file-driven` approach and store Things in some kind of data file we could read from and pass to a constructor function. We're not going to do anything like this though. We are going to take, what I'm going to call, a `scene-driven` approach. We're going to construct and store all our Things in a database scene; much like we were using a scene to create our map's tileset!  

Time to create a new Scene and create its top node as `Node` "Database". Save the scene as `res://things/Database.tscn`.  

Create "category" `Node`s to organize your Things. Can really be set up however you like. This system is stupid flexible.  

We're going to re-make our Player object. This part is important to follow closely. But if All Goes Well, we shouldn't have to do this kind of thing too often in our project, as we're striving to Get It Right The First Time.   

Using your file explorer, copy `res://core/Player/Player.gd` to `res://things/Player.gd`. All we need is a copy of the script, the scene is going to be made fresh from scratch.  

Start with bringing in an instance of `res://thing/Thing.tscn` to your Database. Put it under a "player" category and rename the node from "Thing" to "player". Now, go to the inspector, and down at the bottom is a Script parameter. Use this to change the script this node uses from its default `Thing.gd` to the `res://things/Player.gd` we just copied. **Triple-check** that you're selecting `res://things/Player.gd` and not the original `res://core/Player/Player.gd` we've been working with so far! You will be having a very bad time by the end of this step if you mess this part up.  

By default, when you bring in an instance of a scene into another scene, it treats this scene as another node of the tree, and doesn't let you get to the instanced scene's children. We want to be able to get to our Thing's Icon node and change its texture, so we need to fix that. Click the icon next to the scene that looks like a movie clapper thing (I don't know what to call it) and enable the "Editable Children" option. You can now expand your instanced scene within the scenetree and access its now-orange children.  

![editchildren]  
*This allows you to hack into your instanced scene and do things to its insides*  

Lets add a script to the top Database node. All this script will do is take a string path argument, and return a duplicate of the node that path points to (as long as the path is valid).  

```python
extends Node

# Return an instance of a node in that database at 'path'
func spawn( path ):
	# Find our Thing before we try returning anything
	var thing = get_node( path )
	if thing:
		return thing.duplicate()
	# Print an error message if thing doesn't drop out
	print( "Cannot find a Thing at: " + path )
```  
Now if we want to add a player to our game, we would access this node, and take what `spawn( "player/player" )` returns. That's easy!  


![][databasescene]  
*The Database scene. This will be the virtual Toybox from which we will grab out game pieces to place on our game board.*  






### The Global

Much of Godot logic structure involves having this sort of web of nodes which all communicate with each other. Designing and maintaining these communication systems can be quite a complicated task.  Sometimes, though, you just want to be able to call a function or access a variable from any point in your scene tree. This is where `global scripts` come into play. These can also be called `autoload scripts` or `singletons`. The best way to demonstrate how these work is to just make one and start using it. It's really simple!  

We start by creating a new script. Unlike the scripts we've created so far, this one is not being added to any node in a scene. To do this, go to the Script Tab and select `File > New` and create a new file at `res://global/RPG.gd`. Any scripts which don't extend any particular kind of node or another script should extend `Node`.   

For a quick test, let's define a constant that we can access from another script:  
```python
extends Node

const GREETING = "Hello RPG!"
```
Everything else in this script can be deleted. We have one more step before we are able to use this funky rogue script (no pun intended). Open your Project Settings and find the `AutoLoad` tab. From there, you can select your RPG.gd file. You can also give it a name which should be "RPG" by default. Click Add and your script will be added to the list. Your RPG global script is now imbued into the ether of your game.  
Pick any script now in your game that will be running once you hit Play. In that script's `_ready()` function, add the line:  
`print( RPG.GREETING )`  
Your Output should now greet our new beneficial RPG overlord.  


All global functionality will be done here, and called with ease as `RPG.[x]`, where [x] is any function or member contained within the RPG script. In the future we'll use this script for other universal functions such as rolling dice and all that fun stuff.  

![globalcloud]  

Now let's add something real to RPG.gd (keep the `const GREETING`, we can probably use this later). The first thing we want our global script to do is hold an instance of our database scene, access its script, and use that to return an instance of a Thing from the database (which will then be added to our Map). For now, the only place we need to do this thing is from our Map script, so it could make sense to just write this into Map.gd. But the Ghost of Prototypes Past inform us that we will want to be able to do this from other places as well.  

First, we want RPG to load an instance of the Database scene and store that in a variable. We also want to do this during the script's `_ready()` function so we'll use the `onready` keyword:  

```python
extends Node

const GREETING = "Hello RPG!"

onready var _db = preload( "res://things/Database.tscn" ).instance()
```  
Now that our global script has this data, it can access the scene as if it were part of the scenetree. But instead of using `get_node()` to get to this scene, we just access the `_db` member. Now we just need a simple function to relay a call to spawn an item from the Database and return the result:  

```python
extends Node

const GREETING = "Hello RPG!"

onready var _db = preload( "res://things/Database.tscn" ).instance()


func make_thing( path ):
	return _db.spawn( path )
```
With this, we can get an instance of any Thing in our Database, from anywhere in the universe, as long as we know its path. Lets use this new tool to spawn the player and some other things into our map programmatically.  

Why don't we just make the Database scene itself an AutoLoad? This was the original approach to this system.  
You can assign scenes as singletons in Godot, so in theory we could eliminate the RPG middleman here. The only problem with this is that if the Database scene is loaded as a singleton it will actually add that scene as a node in the tree, which means it will draw all the Things that are placed in that scene over the normal game. We don't need this mess, so we sidestep it by just instancing it and not adding it to the tree. Since we already require a global functions script for other uses, this is a small compromise.  

### Spawn Points
Go to the Map scene, and delete anything that's in the Map. We're going to have our Map script spawn and place our Things with functions. Add this function to the map script, above all other functions:  

```python
# Spawn what path from Database, set position to where
func spawn( what, where ):
	# Add the Thing to the scene and set its pos
	add_child( what )
	what.set_map_pos( where )
```  
Select the Map (TileMap) node, and use the special header it displays to get an X Y position you want to use as the player start position. For my example, my starting position is x10, y10, but your will probably be different.  

![mapxy]  
*This will likely be the one and only time we're going to be hard-coding positions like this. Procedural Generation will do this job for us real soon*  

We want to spawn the player on the map's `_ready` function:  
```python
func _ready():
	# Test objects
	var player = RPG.make_thing( "player/player" )
	spawn( player, Vector2( 10,10 ) )
```

Hit Play and test your player. It should move around and behave just as it always has. If not, you might have to back up and re-do some things.  

Create at least one "Prop" Thing in your Database. A Prop should be a stationary object which has no function (like the stone altar). They should be under some category like "Props". Do this the same way you did for the new Player, but don't touch its script. We don't want our props to move around when we hit movement keys. That would be weird. Make your prop's children editable and bring in a graphic from your source folder to act as the prop's Icon. I chose a completely generic stone alter but you do you.  

Find a couple more spots on your map like you did with your player (these spots should be on floor tiles!). Repeat the process you did with the player to spawn your props.  

```python
func _ready():
	# Test objects
	var player = RPG.make_thing( "player/player" )
	var alter1 = RPG.make_thing( "props/altar" )
	var alter2 = RPG.make_thing( "props/altar" )
  
	spawn( alter1, Vector2( 8,7 ) )
	spawn( alter2, Vector2( 14,7 ) )
  spawn( player, Vector2( 10,10 ) )
```
If we were designing a game which had hand-crafted levels and all the monsters, items and other things were placed by hand, we'd have little need for a system like this. But this is a roguelike, and everything is built by magic robots, so this approach is required.  


### Don't Tread On Me (or Do)
Our Prop Things look awful big and blocky, and should be something which blocks our player's movement. It would also be cool for things like monsters and doors to block our progress the same way walls do. We don't want every Thing to block movement though. Things like potions, swords and pit traps should be something we can step over. Because we want this to be a variable property of Thing, let's add a new variable to the Thing script called `blocks_movement`:  

```python
# Map node
onready var map = get_parent()


export(bool) var blocks_movement = false
```
The `export` keyword exposes this variable to the editor. In dummy captain speak, this means that we can set this variable in the inspector like we do all its other built-in parameters (like position, rotation, scale, and so on). The `(bool)` directly after it is a "hint" to the editor to treat it as a booleen switch, and not a string lineedit, or a number spinbox, or a color selector.  

Go to Database and select one of your nodes which uses Thing. You will see the param checkbox for `blocks_movement` in the inspector.   
Set Altar and Player to block movement.  
Create a potion Thing. Set it to not block movement. Notice that setting the parameter only affects the node you set it on. All these nodes may be sharing a common script, but they're each using a unique copy of that script, with its own values.  

Now if we want to know if a Thing blocks our movement, we can call `if node.blocks_movement:`. We'll create a new function in the map script that can check a cell for blocking Things. To do this, we'll want to go through all Things in the map and check their map position against the Vector2 argument we will give it. If we want to do that, we want a way to logically group all these Things together into an array we can loop (or iterate) through. Godot has a very handy `Groups` tool which will help us here. We can add any node to a group by calling `add_to_group("group_name")` at any point. To get an array of all nodes in a group, we can call (from anywhere in our scene) `get_tree().get_nodes_in_group("group_name")`.  

We can do this as we spawn things into the map. Go to the `spawn()` function in the map script and add this:  

```python
	# All things go to things group
	what.add_to_group("things")
	# Add blocking things to a blockers group
	if what.blocks_movement:
		what.add_to_group("blockers")
```

We'll want to keep all things in a general "things" group. We've also created a special group for blocking objects called "blockers". 

Now our conditions for a non-blocked cell have become slightly more complicated. Just checking whether a map cell is a floor or not isn't going to cut it anymore. We want to keep the `is_floor()` function as we will still have use for it, but we want a new function which does a slightly tighter check on a cell. Call this new function `is_cell_blocked( cell )`:  

```python
# Return TRUE if cell is blocked by anything
func is_cell_blocked( cell ):
	for thing in get_blockers():
		if thing.get_map_pos() == cell:
			return true
	# if no blockers here, check for walls
	return !is_floor( cell )
```  

The exclamation point "!" in `!is_floor( cell )` is an operational `not` symbol. You can write this line as `return not is_floor( cell )` and get the same result, if you're not into cryptic code. Otherwise it's just a little more concise to use !. Since in this function we want to return `TRUE` if the cell is blocked, we want to return the inverse of the result of `is_floor`, which returns `TRUE` if the cell is *not* blocked by a wall.  

You might notice (especially if you try running with this code) that the function `get_blockers()` is totally fake and doesn't exist. *That's because I forgot about it!* Let's make that now, somewhere near the top of your map functions:  

```python
# Return Array of all Things
func get_things():
	return get_tree().get_nodes_in_group("things")

# Return Array of all Movement-Blocking Things
func get_blockers():
	return get_tree().get_nodes_in_group("blockers")
```  

For the sake of it, we also wrote a similar function to `get_things()`. 

Now `Player.step()` can check for "not blocked cells" instead of just "floor" cells. In Player.gd modify the `step()` function so it looks like:  

```python
# Step one cell in a direction
func step( dir ):
	# Clamp vector to 1 cell in any direction
	dir.x = clamp( dir.x, -1, 1 )
	dir.y = clamp( dir.y, -1, 1 )
	
	# Calculate new cell
	var new_cell = get_map_pos() + dir
  
  if map.is_cell_blocked( new_cell ):
    print( "Ow! You hit a wall!" )
  else:
    set_map_pos( new_cell )
```

Now, when your player attempts to step into a cell occupied by a blocking Prop, they will bump off of it as if they hit a wall. 

Spawn some Altars and Potions in your map. You should be blocked by Altars, but be able to walk over Potions.  

Have Map spawn Player last, so it's drawn over other Things. We'll build a proper Z-sort thing later...  


### Watch Where You're Going!
Let's add one more feature before we wrap up this step. We eventually are going to want to know about the Things we bump into, especially once Monsters and Combat start coming into play! We'll begin implementing this functionality now, by having our output announce to us the name of what we bump into, or tell us we're hitting a wall if we are in fact hitting a wall.  

We could use a Thing's node name (via `get_name()`) as our thing's in-game display name, but that is generally a Bad Idea. Rather, we want to create another variable of Thing which will store this name as a string. We also make this an export var so we can set it to each instance in our Database.  

```python
# Map node
onready var map = get_parent()

export(String, MULTILINE) var name = ""

export(bool) var blocks_movement = false
```

As you can guess, the `String` hint tells godot we want to edit this as a string. the `MULTILINE` hint creates an extra button in the inspector text field for this parameter, which will pop up a large multi-line window you can use to input text. This is often easier to work with than the tiny line of space you normally get there. Especially for us who have small monitors and little screen real estate.  

Now we're going to abandon the `is_cell_blocked()` function we just created (don't delete it, we'll want it later!) and create a new function. This will be real similar to `is_cell_blocked()`, but returns a reference to the node that is blocking a cell (or null if it's an "empty" floor), and is called `get_collider( cell )`:  

```python
# Return a blocking Thing, this Map, or null at cell
func get_collider( cell ):
	for thing in get_blockers():
		if thing.get_map_pos() == cell:
			# Return the blocking Thing at this map pos
			return thing
	# Else return me if hitting a wall, or null if hitting air
	return self if not is_floor( cell ) else null
```

The last line in this function is called a `ternary operator`. With this, we are basically declaring a variable and setting it based on an if-else statement, and returning it all in one line. Doing this "the long way" would look like:  

```python
var collider = null
if is_floor( cell ):
  collider = self
return collider
```  

Neither way is Right or Wrong, so use the way you feel most comfortable with. Ternary operators don't show up often in this code, but I thought I would take the opportunity to demonstrate one here.  

Now that we have that, we can modify our `step()` code yet again. This time, we'll construct a dynamic string that will be printed when we try walking into blocked cells that will give us info on what we're bumping into. Here's the entire final iteration of `step()` for this step (I promise!):  

```python
# Step one cell in a direction
func step( dir ):
	# Clamp vector to 1 cell in any direction
	dir.x = clamp( dir.x, -1, 1 )
	dir.y = clamp( dir.y, -1, 1 )
	
	# Calculate new cell
	var new_cell = get_map_pos() + dir
	
	# Check for colliders at new cell
	var collider = map.get_collider( new_cell )
	if collider == map:
		print( self.name + " hits the wall with a thud!" )
	elif collider != null:
		print( "%s punches the %s in the face!" % [self.name, collider.name] )
	else:
		set_map_pos( new_cell )
```  
Now our game is telling us who is hitting what. In the case of us hitting a Thing, we harmlessly punch it in the face.  
For the last print statement in that code we are using `string formatting`. If this looks alien to you, [take a quick break and read this](http://docs.godotengine.org/en/stable/learning/scripting/gdscript/gdscript_format_string.html#multiple-placeholders). We'll probably be using string formats with some regularity down the road.  


Ensure your new Player is working, then **delete the old Player (the whole `res://core/Player` folder)**.  Nothing sucks worse than when you have duplicate files and get into a situation where you're working on one file but testing its results on another, and clawing your eyes out trying to figure out why your changes have no effect!  
Close your project and re-open it, and try playing it. If you get any broken dependency errors now, that means you were probably still using the old player script instead of the one you put in `res://things/`. Which also means that you've probably made all your new changes to the file you just deleted. See what I mean?!  
![fuuu]  
*Here's what we do when our $#!T bricks on us. Don't worry, it happens to the best of us. Stay Strong!*  
 

[here](https://github.com/YeOldeDM/realms-of-todog/tree/28465f2ad40ea38aabd71a67876b7f464e8870bd) you can download a snapshot of the project at this current step, if you'd like something to compare to your own project. 

### Conclusion
We are beginning to put some real meat on our game's bones already. With what we have, we could create as many Things as we like and put them wherever we want in our dungeon. We're not even close to done yet, though! In the next step, we will be throwing ourselves in a totally different direction, and get algorithmic as we create our game's glorious Random Dungeon Generator. I know! I'm excited too!  [Let's do this!!!](../step-4-dungeongen.html)  

![step3fin]  
*The final result at the end of this step. We've built ourselves a virtual punching bag to help us channel our frustration. Namaste!*  


