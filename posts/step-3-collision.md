<!--
.. title: Step 2: Learning To Walk (Into Walls)
.. slug: step-2-collision
.. date: 2017-06-12 03:00:00 UTC
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

In the last step, we began our new project, set some settings, and constructed a skeleton on which we can now begin building the rest of our game upon.  
In this step we will be adding a collision system to our game, as well as flesh out our current player movement system.  

Collision
=====
### Roll Your Own
For many types of games, Godot can handle complex collisions between objects using physics the bodies `KinematicBody2D`, `RigidBody2D` and `StaticBody2D`.  
For our needs, this collision system is overkill. Since all our game's objects are going to operate purely on the grid logic of our map, 
we can use this to our advantage and create our own collision system.  

### Stay On The Floor
From our player, when they move, we can get the map cell they are going to move into. From our map, we can see if that cell is a floor (assuming what's
not a floor will be a wall). If we put those two pieces of information together, we can create a condition in our movement code which only
move the player into valid cells.  

```python
func is_floor( cell ):
  
```

### Stepping
We can put all our movement code into a new function of player called `step( direction )`. This will attempt to move the player one cell
along the vector of `direction`. If the player tries stepping into a wall, we can call a print statement to confirm the condition.  

Directional Constants:  
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

```python
func step( direction ):

```

Movement
=====

### Speaking Through Actions

### Eight Degrees Of Freedom
