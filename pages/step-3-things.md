<!--
.. title: Step 3: Enter The Thing
.. slug: step-3-things
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

[extendsvader]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/3-things/img/extendsvader.png 
[databasescene]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/3-things/img/databasescene.png


In the last step, we added the all-important collision system to our game. Our player is feeling kind of lonely though, so in this step we will introduce the system we will be using to populate our dungeon with everything that isn't the dungeon itself.  


### The Thing
![Picture of The Thing](url)  
*"The Thing can take on any form"*  
#### Generalization
Our game will become a lot more interesting to play if we fill it with lots of interesting things to interact with.  
We want to build our system in such a way that we're creating complex objects out of a collection of simple general parts, like virtual lego pieces.  
This way, we can make a lot of content with just a little work, and ensure that common behavior is consistent among all objects.  

Let's start by defining our most basic type of object and give it a descriptive name. I'll dub this object a Thing. 

Everything inside the dungeon is a Thing.  

#### Player Refactor
Save Player.tscn as `res://things/Thing.tscn` and rename top node to "Thing"  
`res://things/Thing.gd`  

Our most basic Thing object requires only a couple parameters. These will be parameters that everything in our game will need. They are:  
-- Getting and settings its map position  
-- an Icon visual representation  

This list will grow as our game gets more complex, but this gives us something to work with.  

We want to take all the code we've written in our player script which applies to that list above (actually, just the map position part, as our Icon requires no code yet), and move it to our new Thing script. You should be able to just cut the code from one script and paste it into the other.  

Now, your player script should only contain its `_ready` and `_input` functions and their contents. If you tried playing your game at this moment, all your movement code should be broken, of course. We just ripped a bunch of code out of our script, and now the remaining code is trying to call functions that no longer exist! Why the hell would we ever do that??  
Remember that we copied all that code into another script. It's almost as if we're setting ourselves up to have multiple scripts assigned to a single node. But as a rule, a node in the godot engine can only have one script assigned to it. So it must be non-possible for our Player node to utilize both its Player script and a Thing script, right? Not quite!  

There has been one line in all our scripts we've pretty much been ignoring up to this point, but it's a very important line (the most important, one could argue). The `extends` keyword tells your script which functionality it has access to, and is almost always identical to the type of node the script has been created for. The script on a Node `extends Node`. A TextureFrame node's script `extends TextureFrame`. You can think of these as extending your script to use the functions of another higher script. It's just that these "built-in" types don't have actual scripts you can have normal access to. Even though we never defined a `get_pos()` or `set_pos()` function for our player, those functions still do things, because our player extends the methods of Node2D, which *does* define a `get_pos()` and `set_pos()`.  

By using a pinch of Gogo Magic, we can tell our player script to extend one of our own scripts, instead of `Node2D`.

Roll up your sleeves and say the magic words: change the first line of Player.gd so it `extends "res://things/Thing.gd"`!  

Play your game now, and your player works again. Tada!  

Player will inherit all members and methods of Thing. It also has its own members and methods that are unique to the Player.  
All other types of things will also inherit Thing, or inherit a script which eventually inherits Thing.  


![][extendsvader]  


### The Database
`res://things/Database.tscn`  Top node is `Node` "Database"  

Create "category" `Node`s to organize your Things. Can really be set up however you like. This system is stupid flexible.  

We're going to re-make our Player object.  

Copy `res://core/Player/Player.gd` to `res://things/Player.tscn`  

Start with an instance of Thing.tscn. Change its script to Player.gd (at the location you copied it to).  

You will have to right-click > show children to set the texture of each Thing's Icon. Since these are instances, the scene should "hold" these modifications within the instance.  

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
Now if we want to add a player to our game, we would access this node, and call `spawn( "player/player" )`.  That's easy!  


![][databasescene]  
*The Database scene. It wont really live within our game's scenetree, but will exist in a limbo where we will be able to access its data*  






### The Global
Much of game design logic involves having this sort of web of nodes which all communicate with each other. Designing and maintaining these communication systems can be quite a complicated task.  Sometimes, though, you just want to be able to call a function or access a variable from any point in your scene tree. This is where `global scripts` come into play. These can also be called `autoload scripts` or `singletons`. The best way to demonstrate how these work is to just make one and start using it. It's really simple!  

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


All global functionality will done here, and called with ease as `RPG.[x]`, where [x] is any function or member contained within the RPG script. In the future we'll use this script for other universal functions such as rolling dice and all that fun stuff.  
  
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
With this, we can get an instance of any Thing in our Database, from anywhere in the universe, as long as we know its path. Lets use this new tool to spawn the player and some other things into our map programatically.  

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

We want to spawn the player on the map's `_ready` function:  
```python
func _ready():
	# Test objects
	var player = RPG.make_thing( "player/player" )
	spawn( player, Vector2( 10,10 ) )
```

Hit Play and test your player. It should move around and behave just as it always has. If not, you might have to back up and re-do some things.  

Create at least one "Prop" Thing in your Database. A Prop should be a stationary object which has no function (like the stone altar). They should be under some category like "Props".  

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
The `export` keyword exposes this variable to the editor. In dummy captain speak, this means that we can set this variable in the inspector like we do all its other built-in parameters (like position, rotation, scale, and so on). The `(bool)` directly after it is a "hint" to the editor to treat it as a boleen switch (and not a string lineedit, or a number spinbox, or a color selector).  

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
The exclamaition point "!" in `!is_floor( cell )` is an operational `not` symbol. You can write this line as `return not is_floor( cell )` and get the same result, if you're not into cryptic code. Otherwise it's just a little more concise to use !. Since in this function we want to return `TRUE` if the cell is blocked, we want to return the inverse of the result of `is_floor`, which return `TRUE` if the cell is *not* blocked by a wall.  

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

Now `Thing.step()` can check for "not blocked cells" instead of just "floor" cells. In Thing.gd modify the `step()` function so it looks like:  

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
```
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
*[talk about %s stuff]*  


Ensure your new Player is working, then **delete the old Player (the whole `res://core/Player` folder)**.  Nothing sucks worse than when you have duplicate files and get into a situation where you're working on one file but testing its results on another, and clawing your eyes out trying to figure out why your changes have no effect!  
Close your project and re-open it, and try playing it. If you get any broken dependency errors now, that means you were probably still using the old player script instead of the one you put in `res://things/`. Which also means that you've probably made all your new changes to the file you just deleted. See what I mean?!  



[here](https://github.com/YeOldeDM/realms-of-todog/tree/28465f2ad40ea38aabd71a67876b7f464e8870bd) you can download a snapshot of the project at this current step, if you'd like something to compare to your own project. 

### Conclusion
We are beginning to put some real meat on our game's bones already. With what we have, we could create as many Things as we like and put them wherever we want in our dungeon. In most other game development environments, getting to such a point would take many more hours of careful and dangerous work to construct such a system. This is the power of Godot (and game engines in general!). We're not even close to done yet, though! In the next step, we will be throwing ourselves in a totally different direction, and get algorhytmic as we create our game's precious Random Dungeon Generator. I know! I'm excited too!  [Let's do this!!!](../step-4-dungeongen.html)  

