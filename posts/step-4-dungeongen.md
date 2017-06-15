<!--
.. title: Step 4: Enter The Dungeon Generator
.. slug: step-4-dungeongen
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS

In this step, we will be creating a random dungeon generator for our game. Things like this can be incredibly complex and involve all kinds of advanced concepts. But we will be making something that is pretty basic and easy to work with, and still produces interesting results.  

## DunGen
Create a new AutoLoad "DunGen" at `res://global/DunGen.gd`. Unless stated otherwise, all code being written in this step will be in this file.   

### Concepts
#### RNG  
#### 2D Array  
#### Rect2  

### Digital Dice, and How To Roll Them
This will actually be the first time our game requires random numbers. Since Godot doesn't provide a built-in solution for the kind of RNG we need, we want a helper function:  
`func rnd(n,m)`  

### Mother Method Most Mysterious
Our script should work in such a way that we only need to call one function of DunGen in order for it to do its thing. DunGen in turn will work entirely within itself, and return a package of data representing our generated dungeon.  
`func Generate()`  
We want some parameters we can feed to Generate. By providing default values to these arguments in our function declaration, we can call `Generate()` without any arguments, or only *some of the arguments*.  
Vector2 map_size  
Vector2 room_size  
int room_count
var floor_id  
var wall_id  

*[randomize and declare start_pos]*  

*[Construct 2D array of wall_id]*

### Rooms
`func create_room( room_size, floor_id )`  
`func carve_room( room, floor_id )`  
Have Map iterate through room_count, create_rooms and append to an array.  
Check intersections between new rooms and existing rooms.  

#### Room Center
We need another helper function so we can find the center cell of our room Rect2s.  
`func rect_center( rect ):`  

### Halls
As we carve rooms into our map, we also want to carve hallways that connect those rooms together.  
After carving the first room, we get the center point between the new room, and the previously-created room. We then carve an L-shaped hallway between them.  
`func carve_h_hall()`  
`func carve_v_hall()`  
`func carve_hall( A,B )`  
After carving the first room, we'll designate its center as our player start_pos.  

### Putting It All Together
Now our Generate function is generating all our data. We want to return this data as a dictionary:  
`return {"map": map, "start_pos": start_pos}`  
Our Map script will use this package of data to create the game's world and spawn the player at some logical starting point.  

By now, our dungeon generator script is nearly complete already! We will be tweaking it a little in future steps, but for now it's doing all the things we need it to do, so let's move on and integrate this new big tool into the rest of our game..  

## The Art of Building Without Building
Now that our DunGen script can return something we can work with, let's use that data to paint the actual cells into our Map node. Before, we were simply painting our dungeons by hand. Now it's time to have it start doing this in code. Our data map is an array of integers that correlate to our map's cell indicies, so this is real straight-forward. Go to the Map.gd script and change the code under `_ready` like so:  


We now delete the code spawning all our non-player things. Modify the player spawn call to use the start_pos value returned by `DunGen.Generate()`.  


## Conclusion
Get excited! You can now run around and explore your procedurally-generated dungeon! Play around with your Generate parameters to see what kind of results you can get.  
That's going to be it for this step. In the next step, we'll expand on this and have our code fill our dungeon's rooms with Things. We'll also modify our Map so it paints random variations of a set of tiles, so our dungeon doesn't look so blah!  
