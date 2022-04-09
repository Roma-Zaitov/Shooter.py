from pygame import*
from random import randint

#фоновая музыка
mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')

#шрифты и надписи
font.init()
font1 = font.SysFont('Arial', 80)
win = font1.render('YOU WIN!', True, (255, 255, 255))
lose = font1.render('YOU LOSE!', True, (180, 0, 0))
font2 = font.SysFont('Arial', 36)

clock = time.Clock()

# нам нужны такие картинки:
img_back = "galaxy.jpg" # фон игры
img_hero = "rocket.png" # герой
img_bullet = "bullet.png" # пуля
img_enemy = "ufo.png" # враг

score = 0 # сбито кораблей
lost = 0 # пропущено кораблей
max_lost = 3 # проиграли, если пропустили столько

# класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
  # конструктор класса
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        # Вызываем конструктор класса (Sprite):
        sprite.Sprite.__init__(self)

        # каждый спрайт должен хранить свойство image - изображение
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed

        # каждый спрайт должен хранить свойство rect - прямоугольник, в который он вписан
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
 
  # метод, отрисовывающий героя на окне
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

# класс главного игрока
class Player(GameSprite):
    # метод для управления спрайтом стрелками клавиатуры
    def update(self): 
        keys = key.get_pressed()
        if keys[K_a] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_d] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
        if keys[K_w] and self.rect.y > 140:
            self.rect.y -= self.speed
        if keys[K_s] and self.rect.y < win_width - 300:
            self.rect.y += self.speed
  # метод "выстрел" (используем место игрока, чтобы создать там пулю)
    def fire(self):
        bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top, 15, 20, -15)
        bullets.add(bullet)
     
# класс спрайта-врага   
class Enemy(GameSprite):
    # движение врага
    def update(self):
        self.rect.y += self.speed
        global lost
        # исчезает, если дойдет до края экрана
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost = lost + 1

class Asteroid(GameSprite):
    # движение врага
    def update(self):
        self.rect.y += self.speed
        
        # исчезает, если дойдет до края экрана
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            
# класс спрайта-пули   
class Bullet(GameSprite):
    # движение врага
    def update(self):
        self.rect.y += self.speed 
        # исчезает, если дойдет до края экрана
        if self.rect.y < 0:
            self.kill()

# Создаем окошко
win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))

# создаем спрайты
ship = Player(img_hero, 5, win_height - 120, 80, 100, 10)

monsters = sprite.Group()
for i in range(1, 6):
    monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    monsters.add(monster)

asteroids = sprite.Group()
for i in range(1, 4):
    asteroid = Asteroid("asteroid.png", randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    asteroids.add(asteroid)

live = 3
bullets = sprite.Group()

# переменная "игра закончилась": как только там True, в основном цикле перестают работать спрайты
finish = False 
# Основной цикл игры:
run = True # флаг сбрасывается кнопкой закрытия окна 
num_fire = 0
while run:
    # событие нажатия на кнопку Закрыть
    for e in event.get():
        if e.type == QUIT:
            run = False
        # событие нажатия на пробел - спрайт стреляет
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:
                if num_fire <5 and rel_time == False:
                    num_fire = num_fire + 1
                    fire_sound.play()
                    ship.fire()

                    if num_fire >=5 and rel_time == False :
                        last_time = timer()
                        rel_time = True

    if not finish:
        # обновляем фон
        window.blit(background,(0,0))

        # производим движения спрайтов
        ship.update()
        monsters.update()
        bullets.update()
        asteroids.update()

        # обновляем их в ыновом местоположении при каждой итерации цикла
        ship.reset()
        monsters.draw(window)
        bullets.draw(window)
        asteroids.draw(window)

#пишем текст на экране
        text = font2.render("Score: " + str(score), 1, (255, 255, 255))
        window.blit(text, (10, 20))

        text_lose = font2.render("Missed: " + str(lost), 1, (255, 255, 255))
        window.blit(text_lose, (10, 50))

# проверка столкновения пули и монстров (и монстр, и пуля при касании исчезают)
        collides = sprite.groupcollide(monsters, bullets, True, True)
        for c in collides:
            # этот цикл повторится столько раз, сколько монстра подбито
            score = score + 1
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)
            
        if sprite.spritecollide(ship, asteroids, True):
            asteroid = Asteroid("asteroid.png", randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            asteroids.add(asteroid)
            live -= 1
        if live == 0: 
            finish = True ('Game Over')
        display.update()
    # цикл срабатывает каждую 0.05 секунд
    clock.tick(20)
