## Jump UP！
Jump Up is a minimalist side-scroll runner built with MakeCode Arcade.Live demo：https://makecode.com/_2JxJJjU3P9cq
## Table of Contents
### General Information
### Technologies Used
### Features
### Screenshots
### Setup
### Usage
### Project Status
### Room for Improvement
### Acknowledgements
### Contact





## General Information
This is a test heading.
An introductory parkour game built with MakeCode Arcade (Python). Practiced: character physics-based jumping, breaking down game progression, restarting after death, and phased difficulty (day → dusk → night).
Core mechanics: Press A to jump; during Day, adapt to ground obstacles; at Dusk, accelerate the pace; at Night, flying enemies appear and jump windows become tighter. Character dies upon collision with objects, displays score, and presses “A” to restart.

## Technologies Used
MakeCode Arcade (Python mode)  

## Features
Jump Physics and Ground Constraints，Press “A“to apply upward initial velocity. Gravity applies every frame. Upon landing, velocity resets to zero and remains at ground level.  
Phased Difficulty: Day → Dusk → Night 
Day: Introductory pace, ground obstacles  
Dusk: Faster respawns, more urgent movement  
Night: Adds flying enemies, restricts jump rhythm 
Scores are determined by survival time and gold coins collected. A “High score” is recorded upon character death.



## Screenshots
<img width="588" height="438" alt="image" src="https://github.com/user-attachments/assets/f179913a-270b-4989-99dc-030a5964b39d" />


<img width="626" height="838" alt="image" src="https://github.com/user-attachments/assets/05d7179b-e509-4f19-9044-0022e61ca33e" />

<img width="586" height="816" alt="image" src="https://github.com/user-attachments/assets/f396ee77-4e05-4004-8593-f19294448659" />

<img width="584" height="808" alt="image" src="https://github.com/user-attachments/assets/f6e61eb7-c392-42ee-a31c-4ca9ef48d16a" />

## Setup
# Requirements / Dependencies
1.Platform: Microsoft MakeCode Arcade

2.Language: Arcade Python

3.There is no requirements.txt, Pipfile.lock. Everything runs inside the MakeCode Arcade editor.


# Get Started (Browser)
1.Open MakeCode Arcade: https://arcade.makecode.com

2.Switch to Python (top toolbar).

3.Paste game code into main.py

4.Press Run to play in the simulator.

5.Controls: A = jump. Avoid blocks, collect coins; NIGHT mode adds flying drones.

## Usage
### 1) Constants & Global State
# Physics / difficulty / global flags
```python
#GROUND_Y = 110
GRAVITY  = 700
BASE_SPEED = -60

game_running = 0
high_score   = 0
world_stage  = -1  # 0=DAY,1=DUSK,2=NIGHT
current_anim_frame = 0
```
### 2) Player Sprite, Ground Line & Setup
# Player art
```
runner_frame_a = img("""...""")
runner_frame_b = img("""...""")
runner_jump_frame = img("""...""")

runner = sprites.create(runner_frame_a, SpriteKind.player)
runner.z = 10
runner.x = 28
runner.bottom = GROUND_Y
runner.ay = GRAVITY
runner.set_stay_in_screen(True)
```
# Ground guide line (ghost so it doesn't collide)
```
line = sprites.create(image.create(160, 2), SpriteKind.projectile)
line.image.fill(5)
line.left = 0
line.y = GROUND_Y + 1
line.set_flag(SpriteFlag.GHOST, True)
```
### 3) Clamp-To-Ground (no sinking below floor)
```
def clamp_to_ground():
    if runner.bottom > GROUND_Y:
        runner.bottom = GROUND_Y
        runner.vy = 0

game.on_update(clamp_to_ground)
```
### 4) Jump Input (A button)
```
def on_a_pressed():
    if runner.bottom >= GROUND_Y:     # only jump if grounded
        runner.set_image(runner_jump_frame)
        runner.vy = -240
        music.ba_ding.play()          # built-in SFX cue

controller.A.on_event(ControllerButtonEvent.PRESSED, on_a_pressed)
```
### 5) Score & Difficulty Scaling
```def add_score():
    if game_running == 1:
        info.change_score_by(1)

def current_speed():
    extra = info.score()
    if extra > 120:
        extra = 200
    return BASE_SPEED - extra

def retune_speed():
    if game_running == 1:
        for e in sprites.all_of_kind(SpriteKind.enemy):
            e.vx = current_speed()
```




### 6) Ground Enemy Spawner (blocks)
```
def spawn_enemy():
    if game_running == 1:
        e = sprites.create(img("""...block sprite..."""), SpriteKind.enemy)
        e.bottom = GROUND_Y
        e.x = 200
        e.vx = current_speed()
        e.lifespan = 5000
```
### 7) Flying Drone Spawner (NIGHT only)
```def spawn_flying_drone():
    if game_running == 1 and world_stage == 2:
        if randint(0, 100) < 40:
            drone = sprites.create(img("""...drone sprite..."""), SpriteKind.enemy)
            drone.x = 200
            drone.vx = current_speed()
            drone.bottom = GROUND_Y - 30
            drone.lifespan = 4000
```



### 8) Coins: Kind, Spawner & Pickup
```
CoinKind = SpriteKind.create()

def spawn_coin():
    if game_running == 1 and randint(0, 100) < 30:
        coin = sprites.create(img("""...coin sprite..."""), CoinKind)
        coin.x = 160
        coin.y = randint(60, GROUND_Y - 20)
        coin.vx = current_speed()
        coin.lifespan = 4000

def on_get_coin(player, coin):
    music.power_up.play()
    info.change_score_by(3)
    coin.destroy(effects.hearts, 200)

sprites.on_overlap(SpriteKind.player, CoinKind, on_get_coin)
```
### 9) Background Decor (parallax)
```
def spawn_decor():
    if game_running == 1 and randint(0, 100) < 50:
        decor = sprites.create(img("""...plant/tree..."""), SpriteKind.projectile)
        decor.bottom = GROUND_Y
        decor.x = 160
        decor.vx = -30
        decor.set_flag(SpriteFlag.GHOST, True)
        decor.lifespan = 8000
```

### 10) Runner Animation (run vs jump)
```
def animate_runner():
    global current_anim_frame
    if game_running != 1:
        return
    if runner.bottom < GROUND_Y:
        runner.set_image(runner_jump_frame)
        return
    if current_anim_frame == 0:
        runner.set_image(runner_frame_a)
        current_anim_frame = 1
    else:
        runner.set_image(runner_frame_b)
        current_anim_frame = 0
```
### 11) World Theme Switcher (DAY → DUSK → NIGHT)
```
def update_world_theme_with_notice():
    global world_stage
    score_now = info.score()

    if score_now >= 120:
        if world_stage != 2:
            world_stage = 2
            game.splash("NIGHT MODE")
        scene.set_background_image(img("""...night background..."""))
        effects.blizzard.start_screen_effect()

    elif score_now >= 60:
        if world_stage != 1:
            world_stage = 1
            game.splash("DUSK MODE")
        scene.set_background_image(img("""...dusk background..."""))
        effects.confetti.start_screen_effect()

    else:
        if world_stage != 0:
            world_stage = 0
            game.splash("DAY MODE")
        scene.set_background_image(img("""...day background..."""))
        effects.clouds.start_screen_effect()
```
### 12) Enemy Collision → Game Over & Prompt
```
def handle_collision(player, enemy):
    global high_score, game_running
    if game_running == 0:
        return
    game_running = 0

    current_score = info.score()
    if current_score > high_score:
        high_score = current_score

    runner.start_effect(effects.fire, 200)
    enemy.start_effect(effects.disintegrate, 200)
    music.wawawawaa.play()
    pause(100)

    game.splash(
        "Your Score: " + str(current_score)
        + "\nBest Score: " + str(high_score)
        + "\n\nPress A to play again"
    )
    while not controller.A.is_pressed():
        pause(50)
    start_new_run()

sprites.on_overlap(SpriteKind.player, SpriteKind.enemy, handle_collision)
```
### 13) Start / Reset Run
```
def start_new_run():
    global game_running, world_stage

    info.set_score(0)
    runner.bottom = GROUND_Y
    runner.x = 28
    runner.vx = 0
    runner.vy = 0
    runner.ay = GRAVITY
    runner.set_flag(SpriteFlag.INVISIBLE, False)
    runner.set_stay_in_screen(True)

    for e in sprites.all_of_kind(SpriteKind.enemy):
        e.destroy()
    for c in sprites.all_of_kind(CoinKind):
        c.destroy()

    world_stage = -1
    scene.set_background_image(assets.image("bg_day"))  # or your DAY img(...)
    game_running = 1
```
### 14) Help / Splash (controls hint)
```
game.splash(
    "ROBOT RUNNER",
    "A = Jump\nAvoid blocks\nDodge drones at NIGHT\nCollect coins!"
)
```
### 15) Kick Off First Run
```
start_new_run()
```
### 16) Game Loop Scheduling (timers)
```
game.on_update_interval(500, add_score)
game.on_update_interval(1200, spawn_enemy)
game.on_update_interval(1000, spawn_flying_drone)
game.on_update_interval(800, retune_speed)
game.on_update_interval(1500, spawn_coin)
game.on_update_interval(500, spawn_decor)
game.on_update_interval(100, animate_runner)
game.on_update_interval(500, update_world_theme_with_notice)
```
### 17) Built-in Sound References
```
# Jump cue
music.ba_ding
# Coin pickup cue
music.power_up
# Game over cue
music.wawawawaa
```

## Project Status
All essential gameplay features have been implemented: - Core loop of jumping/running - Obstacles - Coins - Drones in dusk/night levels - Scoring and high score system - Animations - Sound effects - Backgrounds.


No critical bugs. It runs reliably in the simulator and on Arcade hardware, so there’s no maintenance pressure.

## Room for Improvement

More enemies and spawn patterns (e.g., dual-channel enemy spawns, random elevation).  
Mission/achievement system and difficulty curve refinements.  
Sound effects and background music; more comprehensive visual effects and special effects.  
Pause menu.  


## Acknowledgements
Microsoft MakeCode Arcade Team — for the official docs, starter projects and built-in assets that made this game possible.

Arcade Tutorials that inspired the core mechanics (runner jump, sprite overlap, timed spawners, screen effects, and music).

Class feedback — thanks to classmates and the instructor for play-testing and suggestions on difficulty pacing.

Inspired by Google’s offline “Chrome Dino” game.


## Contact
Created by @Junjie Li - feel free to contact me!






































































































