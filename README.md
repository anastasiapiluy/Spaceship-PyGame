# Spaceship-PyGame

[Необходимые файлы]()

### Первая версия
```python
import random

import pygame

pygame.init()
screenHeight, screenWidth = 700, 600
screen =pygame.display.set_mode((screenWidth,screenHeight))
pygame.display.set_caption("Spaceship Game")
clock = pygame.time.Clock()

done = False
bonusShield = False
bonusPressed = False
bonusLife = False
lives = 3
score = 0

size = 60
speed = 10
border = 575
x = screenWidth//2-size//2
y = border-size//2

bullets = []
bulletSize = 10
bulletSpeed = 20
timer = 0
gunW, gunH = 10, 20
guns = []

asteroidSize = 50
asteroidSpeed = 3
astTime = 0
asteroidsAmount = 5
starsAmount = 100
starSize = 1
stars = []
asteroids = []

def generateStars():
    stars = [(random.randint(0,screenWidth), random.randint(0,screenHeight)) for i in range(starsAmount)]
    return stars


def asteroidsCollision(bullets):
    global asteroids
    ls = []
    for asteroid in asteroids:
        ast = pygame.Rect(asteroid[0],asteroid[1],asteroidSize,asteroidSize)
        beat = False
        for bullet in bullets:
            bul = pygame.Rect(bullet[0],bullet[1],bulletSize,bulletSize)
            if bul.colliderect(ast):
                bullets.remove(bullet)
                beat = True
                break
        if not beat:
            ls.append(asteroid)
    asteroids = ls

def createGuns():
    global x,y,size,gunW, gunH, bulletSpeed
    indent = 5
    leftGun = pygame.math.Vector2(x+indent,y-indent*2)
    rightGun = pygame.math.Vector2(x+size-indent-gunW,y-indent*2)
    return [leftGun, rightGun]

def fire(guns):
    for gun in guns:
        bullets.append([gun.x+gunW//2, gun.y])

while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and timer == 0:
                fire(guns)
                timer = 45
    if astTime == 0:
        asteroids.append([random.randint(0,screenWidth-asteroidSize), random.randint(0,50)])
        astTime = 100
    for asteroid in asteroids:
        asteroid[1] += asteroidSpeed
    asteroids = [i for i in asteroids if i[1] < screenHeight + asteroidSize]

    if astTime > 0:
        astTime -= 1

    for bullet in bullets:
        bullet[1] -= bulletSpeed
    bullets = [i for i in bullets if i[1] > 0]

    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        x -= speed
    elif keys[pygame.K_RIGHT]:
        x += speed

    guns = createGuns()

    x = max(0-size//2, min(x,screenWidth-size//2))
    if timer>0:
        timer -= 1
    stars = generateStars()
    asteroidsCollision(bullets)

    screen.fill((0,0,50))
    for star in stars:
        pygame.draw.circle(screen, (255,255,255), (star[0], star[1]), starSize)

    pygame.draw.rect(screen, (255,0,0),pygame.Rect(x,y,size,size))
    for gun in guns:
        pygame.draw.rect(screen, (0,0,0), pygame.Rect(gun.x,gun.y,gunW,gunH))

    for bullet in bullets:
        pygame.draw.rect(screen, (100,100,255),(bullet[0],bullet[1],bulletSize,bulletSize))

    for asteroid in asteroids:
        pygame.draw.rect(screen, (200,200,200), (asteroid[0], asteroid[1], asteroidSize, asteroidSize))


    pygame.display.update()
    clock.tick(60)

pygame.quit()

```

### Финальная версия
```python
import random

import pygame

pygame.init()
screenHeight, screenWidth = 700, 600
screen =pygame.display.set_mode((screenWidth,screenHeight))
pygame.display.set_caption("Spaceship Game")
clock = pygame.time.Clock()

font = pygame.font.SysFont("Arial", 20)
over = pygame.font.SysFont("Arial", 80)
mid = pygame.font.SysFont("Arial", 50)

done = False
bonuses = []
bonusTypes = ["Shield", "Life", "Pressed"]
bonusSize = 50
shield = False
bonusSpeed = 5
shieldTimer = 0
pressedTimer = 0

lives = 3
safeTimer = 0
score = 0
state = "play"
bonusTime = 0

size = 130
speed = 10
border = 575
x = screenWidth//2-size//2
y = border-size//2

bullets = []
bulletSize = 30
bulletSpeed = 20
timer = 0
gunW = 10
guns = []

smallAsteroidSize = 50
smallAsteroidSpeed = 3
bigAsteroidSize = 80
bigAsteroidSpeed = 1
astTime = 0
asteroidsAmount = 5
starsAmount = 100
starSize = 1
stars = []
asteroids = []

spaceshipImg = pygame.image.load("spaceship.png")
spaceshipImg = pygame.transform.scale(spaceshipImg, (size, size))
shieldImg = pygame.image.load("shield.png")
shieldImg = pygame.transform.scale(shieldImg, (size+10, size+10))
bulletImg = pygame.image.load("bullet.png")
bulletImg = pygame.transform.scale(bulletImg, (bulletSize, bulletSize))
smallAstImg = pygame.image.load("smallAsteroid.png")
smallAstImg = pygame.transform.scale(smallAstImg, (smallAsteroidSize, smallAsteroidSize))
bigAstImg = pygame.image.load("bigAsteroid.png")
bigAstImg = pygame.transform.scale(bigAstImg, (bigAsteroidSize, bigAsteroidSize))
bigAstImg2 = pygame.image.load("bigAsteroid2.png")
bigAstImg2 = pygame.transform.scale(bigAstImg2, (bigAsteroidSize, bigAsteroidSize))
bigAstImg1 = pygame.image.load("bigAsteroid1.png")
bigAstImg1 = pygame.transform.scale(bigAstImg1, (bigAsteroidSize, bigAsteroidSize))
heartImg = pygame.image.load("life.png")
heartImg = pygame.transform.scale(heartImg, (25, 25))
bonusLifeImg = pygame.image.load("heartBonus.png")
bonusLifeImg = pygame.transform.scale(bonusLifeImg, (bonusSize, bonusSize))
bonusShImg = pygame.image.load("shieldBonus.png")
bonusShImg = pygame.transform.scale(bonusShImg, (bonusSize, bonusSize))
bonusBulImg = pygame.image.load("bulletBonus.png")
bonusBulImg = pygame.transform.scale(bonusBulImg, (bonusSize, bonusSize))

def oldScore():
    try:
        with open("maxGameScore.txt", "r") as f:
            return int(f.read())
    except:
        return 0

def updateScore(score):
    old = oldScore()
    if score > old:
        with open("maxGameScore.txt", "w") as f:
            f.write(str(score))

def generateStars():
    stars = [(random.randint(0,screenWidth), random.randint(0,screenHeight)) for i in range(starsAmount)]
    return stars

def asteroidsCollisionB(bullets):
    global asteroids, score
    ls = []
    for asteroid in asteroids:
        ast = pygame.Rect(asteroid[0],asteroid[1],asteroid[2],asteroid[2])
        beat = False
        for bullet in bullets[:]:
            bul = pygame.Rect(bullet[0],bullet[1],bulletSize,bulletSize)
            if bul.colliderect(ast):
                bullets.remove(bullet)
                asteroid[4] -= 1
                if asteroid[4] <= 0:
                    beat = True
                    score += 100 if asteroid[2] == smallAsteroidSize else 200
                    break
        if not beat:
            ls.append(asteroid)
    asteroids = ls

def asteroidsCollisionS():
    global lives, safeTimer, state, maxScore
    if safeTimer > 0:
        return
    ship = pygame.Rect(x,y,size,size)
    for asteroid in asteroids:
        ast = pygame.Rect(asteroid[0], asteroid[1], asteroid[2], asteroid[2])
        if ship.colliderect(ast) and not shield:
            lives -= 1
            if lives == 0:
                state = "lose"
                maxScore = max(score, maxScore)
                updateScore(maxScore)
            else:
                safeTimer = 90
            break

def createGuns():
    indent = 10
    leftGun = pygame.math.Vector2(x+indent,y-indent*2)
    rightGun = pygame.math.Vector2(x+size-indent-gunW,y-indent*2)
    return [leftGun, rightGun]

def createBonus():
    global bonuses
    bon = random.choice(bonusTypes)
    bonuses.append([random.randint(bonusSize,screenWidth-bonusSize),-bonusSize,bon])

def activateBonus(type):
    global shield, shieldTimer, lives, pressedTimer, timer
    if type == "Life":
        lives += 1
    elif type == "Shield":
        shield = True
        shieldTimer = 300
    else:
        pressedTimer = 300
        timer = 0

def updateBonuses():
    global bonuses
    spaceship = pygame.Rect(x,y,size,size)
    ls = []
    for bonus in bonuses:
        bonus[1] += bonusSpeed
        if bonus[1] > screenHeight:
            continue
        b = pygame.Rect(bonus[0],bonus[1],bonusSize, bonusSize)
        if spaceship.colliderect(b):
            activateBonus(bonus[2])
            continue
        ls.append(bonus)
    bonuses = ls

def fire(guns):
    for gun in guns:
        bullets.append([gun.x+gunW//2, gun.y])

def restart():
    global lives, safeTimer,maxScore, timer, bullets, score, state, guns, astTime, stars, asteroids, shield, bonuses, bonusTime, shieldTimer, pressedTimer
    shield = False
    shieldTimer = 0
    bonuses = []
    bonusTime = 0
    pressedTimer = 0
    lives = 3
    safeTimer = 0
    if score > maxScore:
        maxScore = score
    updateScore(maxScore)
    score = 0
    state = "play"
    bullets = []
    timer = 0
    guns = []
    astTime = 0
    stars = generateStars()
    asteroids = []

maxScore = oldScore()

while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
            if score > maxScore:
                maxScore = score
            updateScore(maxScore)
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and timer == 0 and pressedTimer == 0 and state == "play":
                fire(guns)
                timer = 45
            elif event.key == pygame.K_r:
                restart()
    if state == "play":
        if astTime == 0:
            if random.random() < 0.6:
                asteroidSize = smallAsteroidSize
                asteroidSpeed = smallAsteroidSpeed
                astLives = 1
            else:
                asteroidSize = bigAsteroidSize
                asteroidSpeed = bigAsteroidSpeed
                astLives = 3
            asteroids.append([random.randint(0,screenWidth-asteroidSize), random.randint(-10,10), asteroidSize, asteroidSpeed, astLives])
            astTime = 70
        for asteroid in asteroids:
            asteroid[1] += asteroid[3]
        asteroids = [i for i in asteroids if i[1] < screenHeight + i[2]]

        bonusTime -= 1
        if bonusTime <= 0:
            createBonus()
            bonusTime = random.randint(300, 800)
        updateBonuses()

        if astTime > 0:
            astTime -= 1

        shieldTimer -= 1
        if shieldTimer <= 0:
            shield = False
            shieldTimer = 0

        for bullet in bullets:
            bullet[1] -= bulletSpeed
        bullets = [i for i in bullets if i[1] > 0]

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            x -= speed
        elif keys[pygame.K_RIGHT]:
            x += speed

        guns = createGuns()

        if pressedTimer > 0 and timer == 0:
            fire(guns)
            timer = 10

        if pressedTimer > 0:
            pressedTimer -= 1

        x = max(0-size//2, min(x,screenWidth-size//2))
        if timer>0:
            timer -= 1

        if safeTimer > 0:
            safeTimer -= 1

        stars = generateStars()
        asteroidsCollisionB(bullets)
        asteroidsCollisionS()

    screen.fill((0,0,50))
    for star in stars:
        pygame.draw.circle(screen, (200,200,200), (star[0], star[1]), starSize)

    if safeTimer%10<5:
        spaceship = screen.blit(spaceshipImg, (x, y))
    if shield:
        screen.blit(shieldImg, (x-5, y-5))

    for bullet in bullets:
        screen.blit(bulletImg,(bullet[0],bullet[1]))

    for asteroid in asteroids:
        if asteroid[2] == smallAsteroidSize:
            screen.blit(smallAstImg, (asteroid[0], asteroid[1]))
        else:
            if asteroid[4] == 3:
                screen.blit(bigAstImg, (asteroid[0], asteroid[1]))
            elif asteroid[4] == 2:
                screen.blit(bigAstImg2, (asteroid[0], asteroid[1]))
            else:
                screen.blit(bigAstImg1, (asteroid[0], asteroid[1]))

    for i in range(lives):
        screen.blit(heartImg, (400+i*40,40))

    for bonus in bonuses:
        if bonus[2] == "Life":
            screen.blit(bonusLifeImg, (bonus[0], bonus[1]))
        elif bonus[2] == "Shield":
            screen.blit(bonusShImg, (bonus[0], bonus[1]))
        else:
            screen.blit(bonusBulImg,(bonus[0], bonus[1]))

    text = font.render(f"Score: {score}", True, (255,255,255))
    screen.blit(text, (screenWidth//2-text.get_width()//2,40))

    if state == "lose":
        bigText = over.render("GAME OVER", True, (255,0,100))
        screen.blit(bigText, (screenWidth//2-bigText.get_width()//2,200))
        smallText = mid.render(f"Your score: {score}", True, (255,255,255))
        screen.blit(smallText, (screenWidth//2-smallText.get_width()//2,320))
        best = mid.render(f"Best score: {maxScore}", True, (255,255,255))
        screen.blit(best, (screenWidth//2-best.get_width()//2,400))

    pygame.display.update()
    clock.tick(60)

pygame.quit()

```
