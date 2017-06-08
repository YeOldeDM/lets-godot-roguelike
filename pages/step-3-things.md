<!--
.. title: Step 3: Enter The Thing
.. slug: step-3-things
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

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

There has been one line in all our scripts we've pretty much been ignoring up to this point, but it's a very important line (the most important, one could argue).  

Change Player.gd so it `extends "res://things/Thing.gd"`.  
Now, Player will inherit all members and methods of Thing. It also has its own members and methods that are unique to the Player.  
All other types of things will also inherit Thing, or inherit a script which eventually inherits Thing.  
Godot's node system works on this same principle (`Sprite` inherits `Node2D` inherits `Node`)  



### The Database
`res://things/Database.tscn`  Top node is `Node` "Database"  

Create "category" `Node`s to organize your Things. Can really be set up however you like. This system is stupid flexible.  

We're going to re-make our Player object.  

Copy `res://core/Player/Player.gd` to `res://things/Player.tscn`  

Start with an instance of Thing.tscn. Change its script to Player.gd (at the location you copied it to).  

You will have to right-click > show children to set the texture of each Thing's Icon. Since these are instances, the scene should "hold" these modifications within the instance.  

Ensure your new Player is working, then delete the old Player (the whole `res://core/Player` folder).  Nothing sucks worse than when you have duplicate files and get into a situation where you're working on one file but testing its results on another, and clawing your eyes out trying to figure out why your changes have no effect!  
(we all are suseptable to the "Godot must be broken!" assumption when such bugs pop up. Take a deep breath. 9 times out of 10, you can be sure that it's not Godot's fault but your own. And if it's you're fault, that means it can certainly be fixed. Fail Faster, my friend!)  


### The Global
`res://global/RPG.gd`  
All global functionality is done here, and called with ease as `RPG.[x]`, where [x] is any function or variable contained within the RPG script. In the future we'll use this script for other universal functions such as rolling dice and all that fun stuff.  
  
RPG._db holds an instance of our Database scene  
`RPG.make_thing( path )` Spawn a duplicate from _db path. This is how we bring new objects into our game.  


### Spawn Points
Map gets new `func spawn( what, where )` to programatically create and place objects.  

Spawn some props in Map's `_ready`.  
Player is spawned along with all other things.  

### Don't Tread On Me (or Do)
First export var: `Thing.blocks_movement`  
Go to Database. You will see the param checkbox for `blocks_movement` in the inspector.   
Set Altar and Player to block movement.  
Create a potion Thing. Set it to not block movement.  

Have Map.spawn add Things to groups. Blockers go in a "blockers" group.  
New Map function `is_cell_blocked(cell)` looks for blockers in the cell. If none, it checks for walls.  

Now `Thing.step()` can check for "not blocked cells" instead of just "floor" cells.  

Spawn some Altars and Potions in your map. You should be blocked by Altars, but be able to walk over Potions.  

Have Map spawn Player last, so it's drawn over other Things. We'll build a proper Z-sort thing later...  


### Watch Where You're Going!
New Thing `export var name` with multiline string hint.  Give Database Things names.  

Map gets new function `get_collider(cell)`. This returns a blocking Thing, the Map if cell is a wall, or null if empty floor.  
Thing.step looks for colliders in new_cell, and prints info based on the collider.  


We are beginning to put some real meat on our game's bones already. With what we have, we could create as many Things as we like and put them wherever we want in our dungeon. In most other game development environments, getting to such a point would take many more hours of careful and dangerous work to construct such a system. This is the power of Godot (and game engines in general!). We're not even close to done yet, though! In the next step, we will be throwing ourselves even deeper down the rabbit hole, and get algorhytmic as we create our game's precious Random Dungeon Generator. I know! I'm excited too!  [Let's do this thing!!!](../step-4-dungeongen.html)  

