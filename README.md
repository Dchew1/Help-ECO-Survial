# Help-ECO-Survial
# I'm working on a platformer game in p.y on arcade.makecode and need help.


# =============================================
# ECO SURVIVAL GAME - MAKECODE ARCADE STRUCTURE (Python Version)
# Author: Dajun
# =============================================

# ---------- GLOBAL SETUP ----------
currentLevel = 0
levelCount = 8
collectibleCount = 0
requiredCollectibles = 4

# Define the sprite kinds using pre-defined values in MakeCode Arcade
class SpriteKind:
    Player = SpriteKind.player
    Enemy = SpriteKind.enemy
    Collectible = SpriteKind.collectible
    Goal = SpriteKind.goal

# Initialize the player sprite
myImage0 = sprites.create(assets.image("myImage0"), SpriteKind.Player)
controller.move_sprite(myImage0, 100, 0)
myImage0.ay = 300
scene.camera_follow_sprite(myImage0)

# ---------- FIBONACCI FUNCTION ----------
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

# ---------- SPAWN COLLECTIBLES FUNCTION ----------
def spawn_collectibles(count):
    global collectibleCount
    collectiblesList = [
        "solarPanel",
        "compostBin",
        "windTurbine",
        "recyclePack",
        "pollutionCleaner",
        "cleanEnergy"
    ]
    for i in range(count):
        collectible_name = collectiblesList[randint(0, len(collectiblesList) - 1)]
        col = sprites.create(assets.image(collectible_name), SpriteKind.Collectible)
        tiles.place_on_random_tile(col, assets.tile("collectibleSpawn"))

# ---------- SPAWN ENEMIES FUNCTION ----------
def spawn_enemies(level):
    if level >= 3:
        for i in range(randint(2, 4)):
            enemy_name = "PsharkA" if Math.percent_chance(50) else "PSnake"
            enemy = sprites.create(assets.image(enemy_name), SpriteKind.Enemy)
            tiles.place_on_random_tile(enemy, assets.tile("enemySpawn"))
            enemy.follow(myImage0, 50)

# ---------- NEXT LEVEL FUNCTION ----------
def next_level():
    global currentLevel
    if currentLevel < levelCount - 1:  # Check if we are still below the max level (level 7)
        currentLevel += 1  # Increment the level counter
        load_level(currentLevel)  # Load the next level
    else:
        scene.set_background_image(assets.image("forestTreeGroup0"))  # Show final background when all levels are completed
        game.show_long_text("Congratulations! You've restored the environment and beat Eco Survival!", DialogLayout.FULL)  # Show completion message

# ---------- LEVEL SETUP FUNCTION ----------
def load_level(level):
    global collectibleCount, requiredCollectibles
    # Set environment background
    if level in [0, 1, 2]:
        scene.set_background_image(assets.image("forest 2"))
        if level == 2:
            pause(2000)
            scene.set_background_image(assets.image("healed forest"))
    elif 3 <= level <= 5:
        scene.set_background_image(assets.image("city of hills"))
    elif level >= 6:
        scene.set_background_image(assets.image("forestTreeGroup1"))

    # Set tilemap for each level
    tilemaps = [
        assets.tilemap("level_0"),
        assets.tilemap("level_1"),
        assets.tilemap("level_2"),
        assets.tilemap("level_3"),
        assets.tilemap("level_4"),
        assets.tilemap("level_5"),
        assets.tilemap("level_6"),
        assets.tilemap("level_7")
    ]

    tiles.set_tilemap(tilemaps[level])  # Set the current level's tilemap
    tiles.place_on_random_tile(myImage0, assets.tile("playerSpawn"))  # Spawn player at random position
    collectibleCount = 0  # Reset the collectible count for this level
    requiredCollectibles = fib(level + 4)  # Set the number of required collectibles for the level
    spawn_collectibles(requiredCollectibles)  # Spawn the required number of collectibles
    spawn_enemies(level)  # Spawn enemies for the current level

# ---------- COLLECTIBLE OVERLAP FUNCTION ----------
def on_collectible_overlap(player, col):
    global collectibleCount
    col.destroy()  # Destroy the collectible
    music.ba_ding.play()  # Play sound effect
    collectibleCount += 1  # Increase collectible count
    if collectibleCount >= requiredCollectibles:  # Check if enough collectibles are gathered
        next_level()  # Move to the next level if condition is met

# Attach the overlap event to the player and collectible sprites
sprites.on_overlap(SpriteKind.Player, SpriteKind.Collectible, on_collectible_overlap)

# ---------- START GAME ----------
# Load the first level (level 0) at the start
load_level(currentLevel)
