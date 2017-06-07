<!--
.. title: Step 3: Enter The Thing
.. slug: step-3-things
.. date: 2017-06-19 04:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

# WORK IN PROGRESS

In the last step, we added the all-important collision system to our game. Our player is feeling kind of lonely though, so in this step we will introduce the system we will be using to populate our dungeon with everything that isn't the dungeon itself.  


### The Thing
Generalization. Everything inside the dungeon is a Thing.  
Save Player.tscn as `res://things/Thing.tscn` and rename top node to "Thing"  
`res://things/Thing.gd`  
Migrate all public functions of Player.gd to Thing.gd.  
Change Player.gd so it `extends "res://things/Thing.gd"`.  
Now, Player will inherit all members and methods of Thing. It also has its own members and methods that are unique to the Player.  
All other types of things will also inherit Thing, or inherit a script which eventually inherits Thing.  
Godot's node system works on this same principle (`Sprite` inherits `Node2D` inherits `Node`)  

Class inheritence. 
What's a class? 
What's inheritence? 
Why?  

### The Database
`res://things/Database.tscn`  Top node is `Node` "Database"  
Create "category" `Node`s to organize your Things. Can really be set up however you like. 
We're going to re-make our Player object.  
Copy `res://core/Player/Player.gd` to `res://things/Player.tscn`
Start with an instance of Thing.tscn. Change its script to Player.gd (at the location you copied it to).  
Ensure your new Player is working, then delete the old Player (the whole `res://core/Player` folder).  

### The Global
`res://global/RPG.gd`  
All global functionality is done here, and called with ease as `RPG.[x]`!  
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

