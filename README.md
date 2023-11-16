# shump-10
Shump! is a simple shoot-em-up game, written in Pygame library

### Class Player
```
  class Player(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.Surface((50, 40))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.centerx = WIDTH / 2
        self.rect.bottom = HEIGHT - 10
        self.speedx = 0
```
##### This method is called when a new instance of the Player class is created. It initializes various attributes such as position, image, speed.

```
    def update(self):
        self.speedx = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speedx = -8
        if keystate[pygame.K_RIGHT]:
            self.speedx = 8
        if keystate[pygame.K_SPACE]:
            self.shoot()
        self.rect.x += self.speedx
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH
        if self.rect.left < 0:
            self.rect.left = 0
```
##### This method is called during each iteration of the game loop and is responsible for updating the player's state.
##### "keystate" retrieves the state of all keys on the keyboard. And this time, we'll use LEFT, RIGHT key and spacebar. Those keys will control the player's movement and shooting.

```
    def shoot(self):
        now = pygame.time.get_ticks()
        if now - self.last_shot > self.shoot_delay:
            self.last_shot = now
            bullet = Bullet(self.rect.centerx, self.rect.top)
            all_sprites.add(bullet)
            bullets.add(bullet)
            shoot_sound.play()
```
##### This function handles the player's shooting action.
##### 'self.last_shot','self.shoot_delay' are implementing cooldown

### class Mob
```
class Mob(pygame.sprite.Sprite):
    def __init__(self):
        self.last_update = pygame.time.get_ticks()
        ...

    def rotate(self):
            self.rect.center = old_center
            ...

    def update(self):
            self.speedy = random.randrange(1, 8)
            ...
```
##### Initialize method is called when a new instance of the Player class is created. It initializes various attributes such as position, image, radius, rotation and speed.
##### Rotate method is called for rotating the mob's image. It is called during each iteration of the game loop. Rotates the mob's image based on its rotation speed (self.rot_speed).
##### 
##### Update method is called during each iteration of the game loop and for updating the mob's state. It calls the Rotate method to update the mob's rotated image. It also updates the position of the mob based on its speed (self.speedx and self.speedy).
##### 

### class Bullet
```
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        self.speedy = -10
        ...

    def update(self):
            self.kill()
            ...
```
##### Firstly, initialize various attiributes of bullets such as area, vertical speed. 
##### Then update method will update the position of the bullet based on its vertical speed. When the bullet get out of its area, it removes the bullet and moves off the top of the screen.
##### 
### class Explosion
```
  class Explosion(pygame.sprite.Sprite):
    def __init__(self, center, size):
        self.frame_rate = 50

    def update(self):
        now = pygame.time.get_ticks()
        if now - self.last_update > self.frame_rate:
            self.last_update = now
            self.frame += 1
            if self.frame == len(explosion_anim[self.size]):
                self.kill()
            else:
                center = self.rect.center
                self.image = explosion_anim[self.size][self.frame]
                self.rect = self.image.get_rect()
                self.rect.center = center
```
##### The initialize method initializes the attributes of explosions such as size, area, frame counter. 
##### Then update method gets the current time, then checks if it's time to update the frame considering the frame rate. 
##### When it's time to update the frame of the explosion animation, advances to the next frame by update self.frame. Then checks if all frames have been shown. If all frames have been shown, call 'kill()' method to remove instances of classes which are no longer needed. If not, updates the image and rectangle with the current frame.

### Loading Graphics
#### Images
```
img_star = "assets/Stars.png"
background = pygame.image.load(img_star).convert()
background_rect = background.get_rect()
player_img = pygame.image.load("assets/neenah.png").convert()
bullet_img = pygame.image.load("assets/Heart.png").convert()
meteor_images = []
meteor_list = ['cannonbob.png', 'cannonbobmouth.png']
for img in meteor_list:
    meteor_images.append(pygame.image.load(path.join(all_dir, img)).convert())
explosion_anim = {}
explosion_anim['lg'] = []
explosion_anim['sm'] = []
for i in range(9):
    img = pygame.image.load("assets/rainbowstar.png").convert()
    img_lg = pygame.transform.scale(img, (75, 75))
    explosion_anim['lg'].append(img_lg)
    img_sm = pygame.transform.scale(img, (32, 32))
    explosion_anim['sm'].append(img_sm)
```
##### First of all, loads the background image using 'pygame.image.load' and converts it to the Pygame surface format with function 'convert()'. Then loads the player's image, the bullet image and several meteor images. Iterating for loop(for img in meteor_list), it loads two different meteor image from the specified directory(all_dir) then converts it.
##### 
##### For explosion animation, code creates a directory to store several explosion animation images. 
##### Iterating for loop, loads the base explosion image then scales the image to a larger (or smaller) size for the large (small) explosion.

#### Sounds
```
shoot_sound = pygame.mixer.Sound(path.join(all_dir, 'playerhit.mp3'))
expl_sounds = []
for snd in ['hit01.wav', 'impactshort.wav']:
    expl_sounds.append(pygame.mixer.Sound(path.join(all_dir, snd)))
pygame.mixer.music.load(path.join(all_dir, 'cutie_pie.wav'))
pygame.mixer.music.set_volume(0.4)
```
##### Setting 'playerhit.mp3' as shoot_sound, explosion sounds would be 'hit01.wav' or 'impactshort.wav'. Function 'expl_sounds.append' loads each explosion sound and appends it to the list
##### Background music would be 'cutie_pie.wav'. Finally sets the volume of the background music.
##### 
### Running loop
```
...
    all_sprites.update()

    hits = pygame.sprite.groupcollide(mobs, bullets, True, True)
    for hit in hits:
        score += 50 - hit.radius
        random.choice(expl_sounds).play()
        expl = Explosion(hit.rect.center, 'lg')
        all_sprites.add(expl)
        newmob()

    # check to see if a mob hit the player
    hits = pygame.sprite.spritecollide(player, mobs, True, pygame.sprite.collide_circle)
    for hit in hits:
        player.shield -= hit.radius * 2
        expl = Explosion(hit.rect.center, 'sm')
        all_sprites.add(expl)
        newmob()
        if player.shield <= 0:
            running = False
...
```
##### All the updates of game state start with all_sprites.update().
##### Firstly, check for collisions between bullets and mobs, updating the score and creating explosion effects accordingly.
##### Nextly, check for collisions between mobs and the player, reducing the player's shield and creating explosion effects.
##### Both for loops are updating the score, playing sounds, creating explosions, and managing the player's shield strength. Additionally, the newmob() is called to spawn new mobs after collisions.
##### If the player's shield drops to zero, the game loop is exited.

## Run the game
> $ python shump-10.py

## Game Control
#### Left/Right Arrow Keys: Move the player
#### Spacebar: Shoot bullets
