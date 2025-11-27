from pygame import *
import random
mixer.init()

WIDTH, HEIGHT = 700, 500
screen = display.set_mode((WIDTH, HEIGHT))
display.set_caption("Космический шутер")

clock = time.Clock()
FPS = 80

background = transform.scale(image.load("galaxy.jpg"), (WIDTH, HEIGHT))

mixer.music.load("space.ogg")
mixer.music.play(-2)
fire_sound = mixer.Sound("fire.ogg")

font.init()
stats_font = font.SysFont('Arial', 36)



class GameSprite(sprite.Sprite):
    def __init__(self, image_file, x, y, width, height, speed):
        super().__init__()
        self.image = transform.scale(image.load(image_file), (width, height))
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.speed = speed
    
    def draw(self, surface):
        surface.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update(self):
        keys = key.get_pressed()
        if keys[K_a] and self.rect.x > 0:
            self.rect.x -= self.speed
        if keys[K_d] and self.rect.x < WIDTH - self.rect.width:
            self.rect.x += self.speed

class Enemy(GameSprite):
    def update(self):   
        self.rect.y += self.speed
        if self.rect.y > HEIGHT:
            self.rect.y = 0
            self.rect.x = random.randint(0, WIDTH - seфlf.rect.width)
            global missed_enemies
            missed_enemies += 1

class Asteroid(GameSprite):
    def update(self):   
        self.rect.y += self.speed
        if self.rect.y > HEIGHT:
            self.rect.y = 0
            self.rect.x = random.randint(0, WIDTH - seфlf.rect.width)  

class Bullet(GameSprite):
    def update(self):
        self.rect.y -= self.speed
        if self.rect.y < 0:
            self.kill()

player = Player("rocket.png", WIDTH // 2 - 25, HEIGHT - 80, 50, 80, 5)

enemies = sprite.Group()
for i in range(5):
    enemy = Enemy("ufo.png", random.randint(0, WIDTH - 50), -100, 50, 50, 1)
    enemies.add(enemy)

asteroids = sprite.Group()
for i in range(3):
    asteroid = Asteroid("asteroid.png", random.randint(0, WIDTH - 50), -100, 50, 50, 1)
    asteroids.add(asteroid)


bullets = sprite.Group()

missed_enemies = 0
destroyed_enemies = 0

running = True
result_message = ""

while running:
    for e in event.get():
        if e.type == QUIT:
            running = False
        if e.type == KEYDOWN:
            if e.key == K_SPACE:
                bullet = Bullet("bullet.png", player.rect.centerx - 5, player.rect.top, 100, 100, 10)
                bullets.add(bullet)
                fire_sound.play()  
    
    player.update()
    enemies.update()
    bullets.update()
    asteroids.update()
    






#         я устал
#          ниже
#          гпт
#           ↓
#

    # Проверка столкновений пуль с врагами
    hits = sprite.groupcollide(bullets, enemies, True, True)
    for hit in hits:
        destroyed_enemies += 1
        # Создаем нового врага вместо уничтоженного
        enemy = Enemy("ufo.png", random.randint(0, WIDTH - 50), -100, 50, 50, 2)
        enemies.add(enemy)
    
    num_fire = 0


    # Проверка условий игры
    if sprite.spritecollide(player, enemies, False):
        result_message = "Игра окончена! Вы проиграли - столкновение с врагом!"
        running = False
    elif sprite.spritecollide(player, asteroids, False):
        result_message = "Игра окончена! Вы проиграли - столкновение с астероидом!"
        running = False
    elif missed_enemies >= 3:
        result_message = "Игра окончена! Вы проиграли - пропущено слишком много врагов!"
        running = False
    elif destroyed_enemies >= 10:
        result_message = "Поздравляем! Вы выиграли - уничтожено 10 врагов!"
        running = False
    
    screen.blit(background, (0, 0))
    
    missed_text = stats_font.render(f"Пропущено: {missed_enemies}", True, (255, 255, 255))
    destroyed_text = stats_font.render(f"Сбито: {destroyed_enemies}", True, (255, 255, 255))
    
    screen.blit(missed_text, (10, 10))
    screen.blit(destroyed_text, (10, 50))
    
    player.draw(screen)
    enemies.draw(screen)
    bullets.draw(screen)
    asteroids.draw(screen)
    
    

    # Если игра окончена, показываем сообщение на экране
    if result_message:
        result_text = stats_font.render(result_message, True, (255, 255, 255))
        screen.blit(result_text, (WIDTH//2 - result_text.get_width()//2, HEIGHT//2))
    
    display.flip()
    clock.tick(FPS)

# После выхода из цикла показываем сообщение в консоли
if result_message:
    print(result_message)
    # Небольшая задержка чтобы увидеть сообщение на экране
    time.wait(3000)
