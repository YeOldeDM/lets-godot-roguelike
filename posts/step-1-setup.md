<!--
.. title: Step 1: Learning To Crawl
.. slug: step-1-setup
.. date: 2017-06-5 02:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->


[newproject]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/newproject.png "Our brand-new project, sitting atop our project list, eager to be filled with bugs"

[imageloader]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/imageloadersettings.png "Image Loader settings will ensure our pixel art doesn't get turned into blurry goop"

[mapscene]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/mapscene.png "Our very first scene, waiting to make the awesome happen"

[convert2tileset]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/convert2tileset.png "Converting our scene to a tileset resource"

[examplemap]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/examplemap.png "My test dungeon. It's not very interesting."

[filestruct]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/filestruct.png "The beginning of the project file structure"

[topnode]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/topnode.png "There can be only one!"

[tilescene]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/tilesscene.png "The Tiles_edit SceneTree"

[step1scene]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/step1scene.png "The working product so far"

[projectfolder]: https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/projfold.png "Our happy little project folder, filled with happy little file dreams"

*"The secret of getting ahead is getting started. The secret of getting started is breaking your complex overwhelming task into small manageable tasks, and then starting on the first one." -Mark Twain  

The Project
=====
### New Project
![](https://github.com/YeOldeDM/lets-godot-roguelike/raw/master/img/newproject.png)

We're going to jump straight into building our project in Godot.  
Start up Godot and create a new project from the Project Manager.  Find a *good home* for your project and create a folder there. *Make sure you are creating your project in the location you want to keep it!* You can move your project folder around your hard drive and re-import it from the new source if you need to, but it's a pain in our ass we want to avoid if we can.  



### The project folder  
![][projectfolder]  
Because it's important to understand early on, I'm going to explain the project folder just a bit. If you're already familiar with working with project within Godot, you can skip to the next section.  

When working on our game, we will eventually want to be able to navigate the file structure of our project, using a path string which will look something like:
`res://icon.png`  
If our project was located in `C:\MyProject`, the above resource path would point to `C:\MyProject\icon.png`, which is the Gobot image acting as your project's default icon. Note that this is set up so it's functionally impossible for you to access anything "above" your project folder. This means that any and all resources you will use in your game *must* be within the almighty Project Folder.  
There is, of course, an exception to all of this. And it is called `user://`. But we will get to that much farther down the road, so there's no need to worry about it.

### A Decent file structure
Just as it's important for us to make a firm early decision about our project folder location, we want to exercise the same sort of diligence when constructing the file structure within our project folder. Games are made up of a thousand tiny pieces, and trying to keep these peices organized will help us work faster and better, which is good.  

![][filestruct]  
*An image of what our project folder's file structure should be like. The blacked-out folders are extra things that shouldn't have been in that screenshot...*

### Bringing in graphics  
The graphics resource being used for this project is truely huge in quantity, and there is no way we will be using all of them by the time this thing is finished. It might be tempting to throw this resource folder into your project folder and work with it from there. **Don't do this!** Keep your source folder close, but only bring in the graphics you need, when you need them. We will be using our own `graphics/` folder with its own structure (which is close to what the source folder follows, but not quite).  

Project Settings
=====
Before we get to work, we want to quickly go through our project settings. There are only a couple things we need to do here, but it's important we do it now rather than later. Project Settings can be found under the Scene menu.  
The one most important setting we need to set is under the Image Loader category. Find the `filter` and `gen_mipmaps` options and make sure they are turned *OFF* (they will be ON by default).  While filters are great for high-resolution graphics, they make low-rez pixel art look like muddy, blurry garbage. We'll be using low-rez pixel art, so we want it to look good.
![image loader][imageloader]  


For a bit of early polish, we can also go to the Application category of our Project Settings. There you can give your game a proper name (the default will be your project folder name, if you didn't change that when you created your project) and set a custom icon. This will change the name and icon seen for your project in the Project Manager.

Can We Start Yet??
=====
*Yes we can!  *
We begin by creating a Minimal Viable Product. We want to strip our game down to only what is essensial to make the game work
and still be considered "a game". For our roguelike, we only *really* need two important things: A world for the player to navigate, and a player-controlled agent to navigate that world with.  

## The Map
Our first scene will be our game's Map. The Map scene will be the object representing the static world our game will exist in. In short, 
the dungeon's floors and walls.  
Add your first node to this scene. The node we want to add is `TileMap`.  Once you add it, it will show up in your SceneTree panel. When you select it there, you can see all its exposed properties show up in the Inspector panel. This is the primary way you will be working with nodes throughout this project.  
There is really not much we need to do with our new TileMap's properties yet. The one thing you do want to do is rename `TileMap` to simply `Map`.  Giving your nodes meaningful names is a good habit to get into.  Our game will only really be "one map", so its name doesn't need to be any more descriptive than this. You also want to set the Map's Cell Size to [32,32]; the tile graphics we'll be using are all 32x32 pixels in size, and we want our map cells to match this. You can find this property in the inspector tab while the Map node is selected.  

![][mapscene]

If you've saved your scene before renaming your base node *(and good for you for saving early and often if you did!)* you will want to back up a little.  When you first save a new scene, Godot will automatically name the file after your base node. If we name our node before we save, then we ensure our file name matches the base node name. This is a good bit of organization we can get for free, as long as we follow The Procedure!  
We also want to find a good home for our new Map.tscn file (and it should be named "Map.tscn" by now).  For core game scenes, we'll create a new folder in `res://` called "core". Within that, we want to create a dedicated folder for our scene, which is named after our scene (which is named after its base node). The path of this new scene should be `res://core/Map/Map.tscn`. It might seem redundant, but go with it, later on we'll see why we've done it this way. If you did save your scene somewhere else, go back and delete it now. Clean projects are happy projects.  
One final thing we can do before we move on is to set this scene as our `main_scene`. Go to Project Settings > Application. There you will find a `main_scene` property and a field to select a file from your project folder. 

**Sidenote**  
This first node in a scene is often called the "base node" or "top node" (some also might call it the "root node", which would make sense except it might be confusing with the hidden base of your scene which is actually called "root"). So whenever base node or top node is refered to in these articles, that is what we're talking about. That node at the top of your SceneTree.  
![][topnode]  
Each scene can only have one base node, and once that base node is added, it can't be moved or deleted without deleting the rest of the tree. You can change the type of the base node by right-clicking it and choosing the option there, but it's another one of those cases where it's best to Get It Right The First Time, or Start Over Soon. Changing Type is also known to break nodes on occasion as well, so use the option carefully.  Godot may be pretty stable, but it's not without its faults...  

### Tilesets
Our TileMap will be of little use to us unless we have some way to paint tiles onto its cells. To do that, we need to create a **tileset
resource**.  To do that we need to create a new scene to construct our tiles in.  
The base node of this scene needs to be a `Node`. Just the plain white-circle `Node`. Rename this to "Tileset". Our game will only be using one tileset, so we don't need to be any more descriptive with its name. When we save this scene *(do it now!)*, we can place it in our Map folder as `res://core/Map/Tileset_edit.tscn`. Notice the filename has the extra `_edit` at the end of the name.
We're breaking The Procedure here already, aren't we? And just after we established it! *sigh*  
We could be giving our tileset scene its own dedicated folder and all that, but in this case we're making an exception. *Come to think of it, we're making quite a few exceptions to the rules with our tileset scene here, so get used to it.*  It's a little awkward to be going in this direction so early in the project, but it's just something that will be good to get out of the way before we move forward.
But since our Tileset will be a resource that will be used exclusively by our Map scene, we can package them together without disturbing our organization. Remember that rules are not laws; if you are smart about the how and when, you can usually get away with breaking the rules without upsetting anyone.  

Now, let's fill our Tileset scene with tiles! As a child of Tileset, add two new `Sprite` nodes. We also need to bring in two of the dungeon tile graphics to use; one for floor tiles and one for wall tiles.  I'm using `\dc-dngn\floor\sandstone_floor0.png` and `\dc-dngn\wall\sandstone_wall0.png`, and copying them in `res://graphics/dun/floor/sandstone_floor0.png` and `res://graphics/dun/wall/sandstone_wall0.png`.  Name these sprites "Wall" and "Floor" and give them appropriate tiles (loading them from `res://graphics/`). Tilesets work on tile indicies, so the order of your sprites is important. Our first tile will be index 0, the second 1, and so on.  It will matter later which number represents walls and which are floors, so we want to be consistent. So: 
`WALLS = 0`  `FLOORS = 1`  
![][tilescene]  
Your scene should be looking like this. The position of the sprites in this scene does not matter, though it doesn't hurt to lay them out in some organization.  

We only need these two tiles to begin working with our tilemap. We will be adding much (much) more to this later, once we are ready for that. Before we can use this with our Map scene, we need to convert it to a tileset resource. Go to Scene > Convert to.. > Tileset

![][convert2tileset]

Save this alonside our other scenes in `res://core/Map/Tileset.tres`. Now, we can see where we might get mixed up if we tried having `Tileset.tres` and `Tileset.tscn` living next to each other (thus the `_edit` extension for the file we use to edit the resource).

Once this is done, we can return to the Map scene. Select the Map node, go to its Tile Set property in the inspector and load the new `Tileset.tres` you just created. A new dock should open on the side of your screen with the two tiles in your tileset. Go ahead now and paint some tiles on the screen to make sure things are looking right. Just make sure you work within the blue rectangle (which can be hard to see over the orange tileset grid), as that is your game's viewport. Hit the Play button and note how the view from in-game compares to the view from the editor.  
Of course --with this being a roguelike-- we'll eventually have our game generate for us an *infinite supply of dungeons through the use of black magic*, but for our Minimal Viable Product, we can hand-craft our own mini-dungeons to play around in for now.  

![examplemap][examplemap]

## The Player
We are half way there!  We have a little world space for our little game people to exist in. Now it's time to make one of those little game people. The most important and interesting little person in this game universe: The Player!
Our Player object will be constructed as its own scene, and brought into our Map scene as an instance. If you don't understand instances too well yet, don't worry. Just keep following along and soon it will all start to click.  
In our new scene, let's create a `Node2D` as our base node. This is the one with the blue circle.  A Node2D is node which contains transform data (its position, rotation and scale). The only thing we really need to work with is position, as we want to be able to move our Player around its world. 
Name this Node2D "Player" and save it as `res://core/Player/Player.tscn`.  
We also want to assign a script to this node. Right-click it and select Add Script. If we've followed The Procedure, Godot should already know where and what to call our new script and we can just hit the Create button and move forward. If not, we want to save our new script as `res://core/Player/Player.gd`. We'll come back to this soon and begin writing the first of our code, but before we do that our Player needs one important part added to it.  

### The Icon
The `Node2D` has no visual elements, so we need to add some if we want to be able to tell where our player is on the map. Add a `Sprite` child named "Icon".  Find a good graphic to use (`\player\base\` or `\dc-mon` if you're feeling creative) and copy it to your `res://graphics/player/base/` folder. We will be utilizing all those "paperdoll" elements found in the player folder of the source later down the road, so we'll begin setting up the local structure now. We'll be patting ourselves on the back once we start bringing in more graphics and we don't have to break file dependancies as we are forced to re-organize.  
Before we are done here, disable the sprite's `Centered` property, as the origin of our Meeples will be the top-left corner, not their center (if doesn't make sense now, it should later!)  

### Bringing It Together
We now have the two important elements of our Minimal Viable Product. To bring them together, let's to back to the Map scene. Select the Map node, and click the "Instance scene file as node" button (it's next to the Add Node button, and looks like a link of chains) and select the Player.tscn file you just created. This should bring in an instance (a unique duplicate) of that scene into our Map scene, as a child node of our Map node. This is where the line between "node" and "scene" become a little blurry, and the exact purpose of a "scene" might seem a little less obvious. Once you work with these scenes and nodes long enough, the whole thing will feel natural and intuitive, or at least less weird.  

You should now have your little Player icon floating in space in the top corner of your map. Use the Move tool to move your player somewhere toward the middle of your viewport and feels more important. You can enable snap to grid, and set the grid size to 16x16 (half the resolution of our tiles). This will let you easily snap your player around the map and keep it aligned with the grid, if you're picky about things like that.   

Now, we can hit play and stare at our player standing in the middle of our mini-dungeon. You might be able to get away with calling this "a game" at this point, but it's pretty boring. We need to have a way to let the human player control his virtual player. This is where we begin writing script. This will be just the beginning of a long scripting journey, so grab some coffee and move back over to your player script so that it's open in the script editor. You can access it either through its instance in the Map scene, or by going back to the Player scene and opening it there.  

### See Player. See Player Run. Run Player Run!
Whenever you create a new script, godot will fill that script with some code automatically. Without touching your script yet, it should look like this:  

```python
extends Node2D

# class member variables go here, for example:
# var a = 2
# var b = "textvar"

func _ready():
	# Called every time the node is added to the scene.
	# Initialization here
	pass
```
Delete all of the commented lines (the lines beginning with `#`).  We want the player to react to input from the computer's devices; the keyboard specifically. We tell our node we want to do this by calling `set_process_input(true)`. By calling this in the node's `_ready()` function, we ensure that this happens "right away".  
By turning this process on, the node calls a special function called `_input()` whenever the game registers device input. By defining this function in our script, we can do things with this input, process it in all kinds of ways, then use that processed stuff to make things happen in our world. Like how games usually work.  

```python
extends Node2D

func _ready():
	set_process_input( true )

func _input( event ):
	print( event )
```
The `_input()` function takes one argument, which is the InputEvent it is receiving. If you play your game with this code (making sure your Player is in the scene you're playing) you should see your debugger output spewing out massive amount of data as you do things with your mouse and/or keyboard (or your controller if you have one plugged in). This is a quick and dirty test to ensure that everything is working just as it should!  
This doesn't do us much good, though. What we need to do, is sift through all this input and just catch the events we want, and turn those events into game action. We can do this in a simple way with `actions`. If you go to Project Settings then go to the InputMap tab, you can see several actions already defined. An action is just a string key, bound to one or more (or none) keys/buttons.  You should notice four actions called `ui_up`, `ui_down`, `ui_left` and `ui_right`. We'll borrow these temporarily to act as our player's movement actions. These are bound to their respective arrow keys on the keyboard, so they'll serve our purpose well.  

We can provide a condition in our `_input` function to check if the event is an action press by calling `Input.is_action_pressed("action_name")` where "action_name" is the string key of the action we want to check for.  Let's declare four local variables within `_input` and assign them booleen values based on the pressed status of their respective actions in that input event:  

```python
extends Node2D

func _ready():
	set_process_input( true )

func _input( event ):
	
	# Input
	var UP = event.is_action_pressed("ui_up")
	var DOWN = event.is_action_pressed("ui_down")
	var LEFT = event.is_action_pressed("ui_left")
	var RIGHT = event.is_action_pressed("ui_right")
```
Now to transfer those input signals to actual movement, we want to use Node2D's `set_pos()` and `get_pos()` methods. Basically, we get our current position, add some value to the X or Y values of that position based on input, then set the new position. We could hard code this to get something that looks like:  

```python
if UP:
	var pos = get_pos()
	pos.y -= 32
	set_pos(pos)
elif DOWN:
	var pos = get_pos()
	pos.y += 32
	set_pos(pos)
```
but that is hard-to-maintain, redundant, rigid, and generally fugly code.  First off, we want to make our lives easier and work within the resolution of our map cells, rather than pixels. Luckily, godot makes this task pretty easy.  
Our `TileMap` node has two very cool methods called `world_to_map()` and `map_to_world()` which converts between pixel and cell coordinates. By giving `world_to_map()` our player's pixel position (what it returns with `get_pos()`), we can get back our player's map position. By giving `map_to_world()` a cell that we wish to move the player to, we can get back the pixel coordinates we can then give to the player's `set_pos()`. 
We encapsulate all this in a couple helper functions we can write in the player script, below the first `extends ..` line and above `_ready()`:  

```python
extends Node2D

func get_map_pos():
	pass

func set_map_pos( cell ):
	pass

func _ready():
	set_process_input( true )
```
We've put `pass`, which is an instruction to "do nothing", under these new functions as placeholders, since it's illegal for us to declare empty functions (the same thing was added to `_ready()` when our script was created). Once we fill in the functions with real code, we can delete the `pass`.  
Yo start, we need our player to have a link to its map parent. Since the player (and everything else living in the map) will always be a direct child of the Map node, we can know that the map can be accessed by the player with `get_parent()`, so we'll assign this to a global variable at the top of our script (still below `extends`, that line should always be at the top):  

```python
onready var map = get_parent()
```  
So whenever we want to do anything with our Map scene from this script, we call upon this `map` variable. If we called `print( map.get_name() )`, we would get "Map", the name of that node, as our output. 
The `onready` at the beginning tells the script to wait until after this node has called `_ready()` and established who its parent is. Otherwise, it would try (and fail) to navigate a path to a node which is not ready yet, since these global vars are declared before `_ready` is called, and  

*a parent node is not ready until all its children are ready.*  This is an important rule to remember once we start getting a lot of nodes in our main scene trying to do funky things in their `_ready` functions.  

Rule of Thumb: if a global var requires the node to look outside itself, use `onready`.  
Now that we can easily access the map from player, let's fill in those map_pos functions:  

```python
func get_map_pos():
	return map.world_to_map( get_pos() )

func set_map_pos( cell ):
	set_pos( map.map_to_world( cell ) )
```
Now, writing our movement code should be much easier!  We begin each loop by getting the player's map position. Then based on input, we add or subtract 1 from either the X or Y axis. If the resulting position is different than the current position (that is, it modified the `new_cell` variable due to input), move the player to the modified position. Otherwise, don't bother. Here's what that looks like:  

```python
	# Define a vector to modify
	var new_cell = get_map_cell()
	
	# Modify new_cell based on actions
	if UP:
		new_cell.y -= 1
	if DOWN:
		new_cell.y += 1
	if LEFT:
		new_cell.x -= 1
	if RIGHT:
		new_cell.x += 1
	
	# If new_cell was modified, set our position
	if new_cell != get_map_pos():
		set_map_pos( new_cell )
```
Here is a Game Design Pattern. `Get` some data. `Do` things to that data. `Set` the changed data. Almost everything your game does can almost always be broken down into this pattern, and thinking through problems in this way can usually lead to good solutions.

That's it!  If you've constructed your script properly, you should be able to Hit Play and procede to march your happy little player around your map with the arrow keys. Notice he always keeps his alignment snapped to the grid, even if his initial placement is odd, or you change the cell size of the tilemap(!). We could completely rehaul our tilemap's design and go with any resolution we desire, and never have to touch our code to make it compatible. 

For the sake of completeness, here is the complete player script so far:  

```python
extends Node2D

# Map node
onready var map = get_parent()

# Get our position in Map Coordinates
func get_map_pos():
	return map.world_to_map( get_pos() )

# Set our position to Map Coordinates
func set_map_pos( cell ):
	set_pos( map.map_to_world( cell ) )


# Init
func _ready():
	set_process_input( true )

# Input
func _input( event ):
	
	# Action flags
	var UP = event.is_action_pressed("ui_up")
	var DOWN = event.is_action_pressed("ui_down")
	var LEFT = event.is_action_pressed("ui_left")
	var RIGHT = event.is_action_pressed("ui_right")
	
	# get our map position to modify
	var new_cell = get_map_pos()
	
	# Modify new_cell based on actions
	if UP:
		new_cell.y -= 1
	if DOWN:
		new_cell.y += 1
	if LEFT:
		new_cell.x -= 1
	if RIGHT:
		new_cell.x += 1
	
	# If new_cell was modified, set our position
	if new_cell != get_map_pos():
		set_map_pos( new_cell )




```
I've added plenty of comments to the code to verbally illustrate what a particular chunk of code is really doing.  

![][step1scene]  
*Here is a screenshot of the main scene of our game.*  

[here](https://github.com/YeOldeDM/realms-of-todog/tree/39c93e2e5a3b4ad1002e8b989e0235a6fb650226) you can download a snapshot of the project at this current step, if you'd like something to compare to your own project.  

That's it for this step! In the next step we will make it so our player can't walk through walls, and expand on the movement mechanics we've established here. Since we spent the extra time in this step setting the stage, this next leap should be straight-forward and easy to get through. Once you're ready, it's time for [Step 2](../step-2-collision.html)  


