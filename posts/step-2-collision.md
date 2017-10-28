<!--
.. title: Step 2: Learning To Walk (Into Walls)
.. slug: step-2-collision
.. date: 2017-06-07 03:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

[isfloor]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/isfloor.png

[stepactions]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/stepactions.png "Our new InputMap, with all our new actions just barely fitting on-screen"  

[searchhelp]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/searchhelp.png  

[compasscoord]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/compascoord.png

[screencoord]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/screencoord.png

In the last step, we began our new project, set some settings, and constructed a scaffolding on which we can now begin building the rest of our game upon.  
In this step we will be adding a collision system to our game, as well as flesh out our current player movement system.  

Collision
=====
*"I can only show you the door, Neo. You have to walk through it. You...have to open the door first..." -White Morpheus, "Computer Boy"*  

### Roll Your Own
For many types of games, Godot can handle complex collisions between objects using physics the bodies `KinematicBody2D`, `RigidBody2D` and `StaticBody2D`.  
For our needs, this collision system is overkill. Since all our game's objects are going to operate purely on the grid logic of our map, 
we can use this to our advantage and create our own collision system.  All of our "Moving Bodies" will be `Node2D`-based, and all collision will be calculated by our Map. Let's jump right in!  

### Stay On The Floor
One major issue with our game at the moment is the fact that our player can walk through walls. We're looking to our map to express our position in the world and change that position, but we're not considering any conditions of the map cells themselves to see whether or not our object should be able to occupy that cell.
From the Player, we will be able to get the cell we wish to step into. From the Map, we should be able to check that cell, and get the `tile index` of any tile painted on that cell. With that information we can conclude whether that cell is a floor, or something else (which we cannot enter). 
Go ahead and add a new script to your Map node. Since this "poke the cell" behavior is something that is handled by the Map node, this function should reside in a script on that node. 
Auto-complete here should give you the correct path and filename for your new script (otherwise save to `res://core/Map/Map.gd`). Much like we did with the player script, we can delete all the text in this script except the first `extends ..` line. Our map node will soon be performing all kinds of cool functions, but for now we need just this one simple one:  

```python
extends TileMap

# Return TRUE if cell is a floor on the map
func is_floor( cell ):
	return get_cellv( cell ) == 1
  
```  


![][isfloor]  

*The Player throws coordinates at the map and uses the information returned from that to dictate its behavior.*  


If you look through the documentation for the methods of `TileMap`, you will see both `get_cell()` and `get_cellv()` methods (as well as comperable methods for setting cells). Where the former takes two `int` arguments (X and Y), the "v" varients take one `Vector2` argument ( Vector2(X,Y) ). Since the argument we're expecting in our `is_floor( cell )` is a Vector2, it makes sense for us to use the method of our map that also takes a Vector2.  Don't be afraid of using the built-in Search Help or the [online docs](http://docs.godotengine.org) to browse through the functions available to you for the different nodes you're using in your project. Both sources should contain the same information (and lack of), so take whichever is more convenient for you, and keep it close.  

![][searchhelp]  
*The Search Help button will become your very best friend. It is as much of a life-saver as its icon suggests.*  



### Stepping
We want to take all the movement control code (the parts which take your input and turns it into output) of the player into a new function called `step( direction )`. This will attempt to move the player one cell along a local `direction` vector. If the player tries stepping into a wall, we can call a print statement to confirm the action has gone through, since "lack of movement" when we hit a wall isn't necessarily a clear indicator that the code is doing what it should.
Add this function to the player script, just below `var map`, above the other functions:  

```python
# Step one cell in a direction
func step( dir ):
	# Clamp vector to 1 cell in any direction
	dir.x = clamp( dir.x, -1, 1 )
	dir.y = clamp( dir.y, -1, 1 )
	
	# Calculate new cell
	var new_cell = get_map_pos() + dir
	
	# Check for valid cell to step to
	if map.is_floor( new_cell ):
		set_map_pos( new_cell )
	# Announce when we bump something
	else:
		print( "Ow! You hit a wall!" )
```  


### Refactoring
Now that all this getting and setting of map positions is being done in this function, we can remove all of this from our `_input` function and replace it with something more concise:  
```python
# Input
func _input( event ):
	
	# Step Actions
	if event.is_action_pressed("ui_up"):
		step( Vector2( 0, -1 ) )
	if event.is_action_pressed("ui_down"):
		step( Vector2( 0, 1 ) )
	if event.is_action_pressed("ui_left"):
		step( Vector2( -1, 0 ) )
	if event.is_action_pressed("ui_left"):
		step( Vector2( 1, 0 ) )
```  
This process of deleting old code and replacing it with new code as we expand our objects' functionality is refered to as "refactoring our code". It's likely that most of the initial code you write will end up being refactored at least once (if not twice or thrice) over the development of your game.  
 
You'll notice that we've been using a lot of these magic `Vector2` things in our code. If "Vector Math" sounds scary to you, you might want to go give [this article](http://docs.godotengine.org/en/stable/learning/features/math/vector_math.html) a read (it's short). We probably wont be doing anything too fancy with vectors in this project; mostly we will be using them as xy coordinates. But it doesn't hurt having at least a comfortable understanding of vectors, as they are a commonly-used thing in almost any kind of game.  

![][screencoord]  
*Remember that in our 2D environment, our Y axis moves **downward** along the positive axis; where most environments work with their comparable axis going positive **upward**. Since the origin of our canvas(our game window's screenspace) is in the top-left corner, this begins to make sense.*   


Movement
=====
So far, our player movement control is only okay. One feature we want to add to it is the ability to move diagonally. Because of the nature of our game, using combinations of arrow keys to get diagonal movement just will not be possible. Also, once we begin adding User Interface elements to our game, those `ui_` actions could be used to shift the focus of those elements, and we don't want those being confused with our movement actions. We want to dedicate a block of eight (nine, actually) keys on our keyboard to player movement.  

### Speaking Through Actions
If you go to Project Settings > InputMap, you can manage your game's Actions.  Because they can be expressed in a short and understandable way, we'll use "compass coordinates" to express direction. We will say "N"orth equals "Up", "W"est equals "Left", and so on.  For our action names, we'll use a convention of `step_D` where D is the compass direction we want that action to represent.  
![InputMap image][stepactions]


### Eight Degrees Of Freedom
In my opinion, the best keys to use for our 8-directional movement are the "outer ring" of the keypad. Unfortunately, enough keyboards out there don't have dedicated keypads for us to be able to rely on them as a primary control scheme. So we want to provide an alternative for those poor souls. Fortunately for us, we can assign as many input events as we like to a single action! We can bind both sets of keys to our `step_` actions. 
```
7 8 9
4 5 6
1 2 3
```  
or  
```
Q W E
A S D
Z X C
```  
![keylayouts][compasscoord]  
*Compass coordinates gives us a nice way to convert geeky vectors to a more commonly-understood idea*  

(KP5/S are not being used for movement, but we will be using these keys later)  

 


### Acting Is Reacting
Putting these two bits of function together to move our player with our new custom actions is pretty straightforward.  

First, we want to give ourselves a non-confusing way to convert our abstract compass coordinates to the Vector2s the script can work with. We could hard-code it like we already have it, but we should know by now that hard-coding things is (mostly) a Bad Idea.  
We can declare a `Dictionary` in our player script, which holds Vector2 coordinates under compass coordinate keys. 

```python
const DIRECTIONS = {
	"N":	Vector2(0,-1),
	"NE":	Vector2(1,-1),
	"E":	Vector2(1,0),
	"SE":	Vector2(1,1),
	"S":	Vector2(0,1),
	"SW":	Vector2(-1,1),
	"W":	Vector2(-1,0),
	"NW":	Vector2(-1,-1),
	}
```  
We've declared this as a `const`(constant) rather than a `var`(variable). Operationally, constants and variables work exactly the same. The only difference is that constants are *immutable*; a fancy way of saying its value cannot change. Commonly in programming, we indicate constants by typing them in all-caps, or `SCREAMCASE` if you want a cooler-sounding term.  At the risk of going on a tangent, here's a quick rundown of the "case types" typically seen in code:
`snake_case` is what we are using for functions, variables, and most other things 
`camelCase` or   
`PaskalCase` are usually used for Node names and filenames
`SCREAMCASE` is typically reserved just for constants  

We'll be using snake_case for most of what we're doing for this tutorial. You can use whatever style(s) you want in your own code, just keep it consistent, or later you will be confusing yourself over whether that function you want is `do_stuff()` or `doStuff()` or `DoStuff()`! Now let's get back to our game..  

Now, all we need to do is replace the wall of ifs in our player's `_input` function with the eight directions we need. Rather than passing a Vector2 directly to `step()`, we're passing a reference to our DIRECTIONS dictionary:  

```python
# Input
func _input( event ):
	
	if Input.is_action_pressed( "step_n" ):
		step( DIRECTIONS.N )
	if Input.is_action_pressed( "step_ne" ):
		step( DIRECTIONS.NE )
	if Input.is_action_pressed( "step_e" ):
		step( DIRECTIONS.E )
	if Input.is_action_pressed( "step_se" ):
		step( DIRECTIONS.SE )
	if Input.is_action_pressed( "step_s" ):
		step( DIRECTIONS.S )
	if Input.is_action_pressed( "step_sw" ):
		step( DIRECTIONS.SW )
	if Input.is_action_pressed( "step_w" ):
		step( DIRECTIONS.W )
	if Input.is_action_pressed( "step_nw" ):
		step( DIRECTIONS.NW )
```  
Now, we could probably slim this down with some programming trickery, and do the same thing with only a handful of lines. But there is a point where you're just trying to swat flies with a bazooka. This code is a little bloated, but it's easy to understand (even if you don't know the programming language) and easy to work with. At this early stage of our project, this is more important than being optimized. While it's good to be always thinking ahead in your project and anticipating future issues, overengineering and premature optimization can lead to your design becoming too rigid and hard to add to without breaking everything else. We want our baby game to be soft and squishy, and not shatter on us when we press on it too hard.  

Fire up your game and you should now be able to move in all eight directions, using whatever key(s) you've bound to your step actions!  

[here](https://github.com/YeOldeDM/realms-of-todog/tree/42988396607e866ef7051021e55aea65940496de) you can download a snapshot of the project at this current step, if you'd like something to compare to your own project. 

That is it for this step! The next thing we're going to work on is it's own thing, so we will give it its own step. It's time for [Step 3!](../tutorials/step-3-things.html)  








