# Part 3 - Animate player movement

- [Part 3 - Animate player movement](#part-3---animate-player-movement)
  - [Setup the player-scene for animation](#setup-the-player-scene-for-animation)
  - [Setup the animation-keyframes](#setup-the-animation-keyframes)
    - [walkDown-animation](#walkdown-animation)
    - [the other 3 animations](#the-other-3-animations)
    - [Applying animation in script](#applying-animation-in-script)
      - [Goals](#goals)
      - [Implementation](#implementation)
        - [Store a reference to the AnimationPlayer-node](#store-a-reference-to-the-animationplayer-node)
        - [Define the updateAnimation function](#define-the-updateanimation-function)
          - [Code](#code)
          - [Explanation](#explanation)
        - [Call updateAnimation in \_physics\_process handler](#call-updateanimation-in-_physics_process-handler)
      - [Full code](#full-code)

> Video: [How to animate player movement in Godot 4](https://www.youtube.com/watch?v=-Q0VwNVK3rk&list=PLMQtM2GgbPEVuTgD4Ln17ombTg6EahSLr&index=3&pp=iAQB)
> 
## Setup the player-scene for animation

- Add AnimationPlayer-node to our Player-scene
- In the animation-panel (bottom of editor), click Animation -> "New Animation"
- Name the animation (walkDown)
- Set snap to 0.2 (bottom-right-corner of animation-panel). This sets the time between two animation frames.
- every frames last 0.2 seconds and every animation-cycle consists of 4 frames, so we need to set the animation-length to 0.8 (animation-panel, right to the time at the top)
- right to animation-length there is a loop-switch, which we set on

## Setup the animation-keyframes

### walkDown-animation
- Select the Sprite2d-node
- In the inspector, some properties have a little key-button to the right. These are values which can be animated.
- Under animation, set Frame-coords to x:0 and y:0
- Click the key-button right to the x to create a keyframe
- In the dialog we tell it to create a new track
- we set Frame-coords to x:0 and y:1 and create a new keyframe
- we repeat this until x:0 and y:3, so we have 4 keyframes

### the other 3 animations
- In the animation-panel, click Animation -> Duplicate
- Name the animation walkUp
- Click on each frame and raise x-axis-value under AnimationTrackKeyEdit in the inspector by 1
- Same method is used for walkLeft and walkRight, look on your sprite-sheet to identify which goes where.

### Applying animation in script

#### Goals

1. Store a reference to the AnimationPlayer-node in a property, when the scene is loaded
2. Define an updateAnimation function
    1. Stop the animation when player is not moving
    2. Get the move-direction by checking x & y properties of the velocity and store it in a direction-variable
    3. Play a animation with the name stored in the direction-variable
 3. Call updateAnimation in the _physics_process-handler

#### Implementation

##### Store a reference to the AnimationPlayer-node

We store the reference in a property, which we define right at the start 
of the script. Right below the speed-variable declaration.

As scenes are only warranted to be configured when entering the active scene-tree, their sub-nodes can only be obtained when a call to Node._ready() is made.  
This would lead to code such as this:

```GDScript
var animations

func _ready()
    animations = get_node("AnimationPlayer")
```

We shorten this by using the `@onready` [[annotation]], which does exactly this.

```GDScript
@onready var animations = $AnimationPlayer
```

`$AnimationPlayer` is a shortcut for `get_node("AnimationPlayer")`

##### Define the updateAnimation function

###### Code

```GDScript
func updateAnimation() -> void:
	if velocity.length() == 0:
		animations.stop()
	else:
		var direction = "Down"
		if velocity.x < 0: direction = "Left"
		elif velocity.x > 0: direction = "Right"
		elif velocity.y < 0: direction = "Up"
		
		animations.play("walk" + direction)
```

###### Explanation

To know if the player is moving or not, we test if `velocity.length()` 
matches 0 and call the `stop()`-method of our animations-object.

If it doesnt match it means, the player is moving. At this point we need to know in which direction she/he is moving. To do so we first declare a direction-variable: `var direction = "Down"`

No we only need to test for the other 3 directions. For this we compare velocities x and y with 0 and set our direction-variable accordantly.

##### Call updateAnimation in _physics_process handler

At the end of our _physics_process handler we add a line where we call our new `updateAnimation()` function.

#### Full code

```GDScript
extends CharacterBody2D

@export var speed: int = 35
@onready var animations = $AnimationPlayer

func handleInput() -> void:
	var moveDirection = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	velocity = moveDirection * speed

func updateAnimation() -> void:
	if velocity.length() == 0:
		animations.stop()
	else:
		var direction = "Down"
		if velocity.x < 0: direction = "Left"
		elif velocity.x > 0: direction = "Right"
		elif velocity.y < 0: direction = "Up"
		
		animations.play("walk" + direction)

func _physics_process(delta: float) -> void:
	handleInput()
	move_and_slide()
	updateAnimation()
```
