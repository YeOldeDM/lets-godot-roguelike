<!--
.. title: Step 1: Learning To Crawl
.. slug: step-1-setup
.. date: 2017-06-1 01:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->
[newproject]: https://raw.githubusercontent.com/YeOldeDM/lets-godot-roguelike/step-1/img/newproject.png "Our brand-new project, sitting atop our project list, eager to be filled with bugs"

[imageloader]: https://raw.githubusercontent.com/YeOldeDM/lets-godot-roguelike/step-1/img/imageloadersettings.png "Image Loader settings will ensure our pixel art doesn't get turned into blurry goop"

[mapscene]: https://raw.githubusercontent.com/YeOldeDM/lets-godot-roguelike/step-1/img/mapscene.png "Our very first scene, waiting to make the awesome happen"

[convert2tileset]: https://raw.githubusercontent.com/YeOldeDM/lets-godot-roguelike/step-1/img/convert2tileset.png "Converting our scene to a tileset resource"

The Project
=====
### New Project
We're going to jump straight into building our project in Godot.  
Start up Godot and create a new project from the Project Manager.  Find a *good home* for your project and create a folder there. *Make sure you are creating your project in the location you want to keep it!* You can move your project folder around your hard drive and re-import it from the new source if you need to, but it's a pain in our ass we want to avoid if we can.  

![new project screenshot][newproject]  

### The project folder
Because it's important to understand early on, I'm going to explain the project folder just a bit. If you're already familiar with working with project within Godot, you can skip to the next section.  

When working on our game, we will eventually want to be able to navigate the file structure of our project, using a path string which will look something like:
`res://icon.png`  
If our project was located in `C:\MyProject`, the above resource path would point to `C:\MyProject\icon.png`, which is the Gobot image acting as your project's default icon. Note that this is set up so it's functionally impossible for you to access anything "above" your project folder. This means that any and all resources you will use in your game *must* be within the almighty Project Folder.  
There is, of course, an exception to all of this. And it is called `user://`. But we will get to that much farther down the road, so there's no need to worry about it.

### A Decent file structure
Just as it's important for us to make a firm early decision about our project folder location, we want to exercise the same sort of diligence when constructing the file structure within our project folder. Games are made up of a thousand tiny pieces, and trying to keep these peices organized will help us work faster and better, which is good.
*(picture diagram of mock file structure)*  

### Bringing in graphics  
The graphics resource being used for this project is truely huge in quantity, and there is no way we will be using all of them by the time this thing is finished. It might be tempting to throw this resource folder into your project folder and work with it from there. **Don't do this!** Keep your source folder close, but only bring in the graphics you need, when you need them. We will be using our own `graphics/` folder with its own structure (which is close to what the source folder follows, but not quite).  

Project Settings
=====
Before we get to work, we want to quickly go through our project settings. There are only a couple things we need to do here, but it's important we do it now rather than later. Project Settings can be found under the Scene menu.  
The one most important setting we need to set is under the Image Loader category. Find the `filter` and `gen_mipmaps` options and make sure they are turned *OFF* (they will be ON by default).  While filters are great for high-resolution graphics, they make low-rez pixel art look like muddy, blurry garbage. We'll be using low-rez pixel art, so we want it to look good.
![image loader][imageloader]  


For a bit of early polish, we can also go to the Application category of our Project Settings. There you can give your game a proper name (the default will be your project folder name) and set a custom icon. This will change the name and icon seen for your project in the Project Manager.

Can We Start Yet??
=====
*Yes we can!  *
We begin by creating a Minimal Viable Product. We want to strip our game down to only what is essensial to make the game work
and still be considered "a game". For our roguelike, we only *really* need two important things: A world for the player to navigate, and a player-controlled agent to navigate that world with.  

## The Map
Our first scene will be our game's Map. The Map scene will be the object representing the static world our game will exist in. In short, 
the dungeon's floors and walls.  
Add your first node to this scene. The node we want to add is `TileMap`.  Once you add it, it will show up in your SceneTree panel. When you select it there, you can see all its exposed properties show up in the Inspector panel. This is the primary way you will be working with nodes throughout this project.  
There is really not much we need to do with our new TileMap's properties yet. The one thing you do want to do is rename `TileMap` to simply `Map`.  Giving your nodes meaningful names is a good habit to get into.  Our game will only really be "one map", so its name doesn't need to be any more descriptive than this.  

![][mapscene]

If you've saved your scene before renaming your base node *(and good for you for saving early and often if you did!)* you will want to back up a little.  When you first save a new scene, Godot will automatically name the file after your base node. If we name our node before we save, then we ensure our file name matches the base node name. This is a good bit of organization we can get for free, as long as we follow the procedure!  
We also want to find a good home for our new Map.tscn file (and it should be named "Map.tscn" by now).  For core game scenes, we'll create a new folder in `res://` called "core". Within that, we want to create a dedicated folder for our scene, which is named after our scene (which is named after its base node). The path of this new scene should be `res://core/Map/Map.tscn`. It might seem redundant, but go with it, later on we'll see why we've done it this way. If you did save your scene somewhere else, go back and delete it now. Clean projects are happy projects.  

**Sidenote**  
This first node in a scene is often called the "base node" or "top node" (some also might call it the "root node", but that might be confusing with another part of your scene which is called "root", so I avoid using it). So whenever base node or top node is referred to in these articles, that is what we're talking about.
Each scene can only have one base node, and once that base node is added, it can't be moved or deleted without deleting the rest of the tree. You can change the type of the base node, but it's another one of those cases where it's best to Get It Right The First Time, or Start Over Soon.

### Tilesets
Our TileMap will be of little use to us unless we have some way to paint tiles onto its cells. To do that, we need to create a tileset
resource.  To do that we need to create a new scene to construct our tiles in.  
The base node of this scene needs to be a `Node`. Just the plain white-circle `Node`. Rename this to "Tileset". Our game will only be using one tileset, so we don't need to be any more descriptive with its name. When we save this scene (do it now!), we can place it in our Map folder as `res://core/Map/Tileset_edit.tscn`. Notice the filename has the extra `_edit` at the end of the name.
We're breaking our procedure here already, aren't we? We could be giving our tileset scene its own dedicated folder and all that, but in this case we're making an exception. Come to think of it, we're making quite a few exceptions to the rules with our tileset scene here, so get used to it.  It's a little awkward to be going in this direction so early in the project, but it's just something that needs to be done before we can move forward.
But since our Tileset will be a resource that will be used exclusively by our Map scene, we can package them together without disturbing our organization. Remember that rules are not laws; if you are smart about the how and when, you can usually get away with breaking the rules without upsetting anyone.  

Now, let's fill our Tileset scene with tiles! As a child of Tileset, add two new `Sprite` nodes. We also need to bring in two of the dungeon tile graphics to use; one for floor tiles and one for wall tiles.  I'm using `\dc-dngn\floor\sandstone_floor0.png` and `\dc-dngn\wall\sandstone_wall0.png`, and copying them in `res://graphics/dun/floor/sandstone_floor0.png` and `res://graphics/dun/wall/sandstone_wall0.png`.  Name these sprites "Wall" and "Floor" and give them appropriate tiles (loading them from `res://graphics/`). Tilesets work on tile indicies, so the order of your sprites is important. Our first tile will be index 0, the second 1, and so on.  It will matter later which number represents walls and which are floors, so we want to be consistent. So: 
`WALLS = 0`  `FLOORS = 1`  
If for whatever reason, it makes more sense to you to switch them around, you can. Just pick a pattern and stick to it.  
We only need these two tiles to begin working with our tilemap. We will be adding much (much) more to this later, once we are ready for that. Before we can use this with our Map scene, we need to convert it to a tileset resource. Go to Scene > Convert to.. > Tileset

![][convert2tileset]

Save this alonside our other scenes in `res://core/Map/Tileset.tres`. Now, we can see where we might get mixed up if we tried having `Tileset.tres` and `Tileset.tscn` living next to each other (thus the `_edit` extension for the file we use to edit the resource).

## The Player
Our Player object will be constructed as its own scene. We'll instance it as a child of our Map node.  

### The Icon
The `Node2D` has no visual elements, so we need to add some. Add a Sprite child named "Icon".  

