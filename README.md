
import pygame
import random
import sys

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
FPS = 60
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
PLAYER_SPEED = 5
BULLET_SPEED = -10
ENEMY_SPEED = 3
SPAWN_RATE = 30  # Lower is faster spawning

# Initialize screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("2D Shooting Game")
clock = pygame.time.Clock()

# Load assets
player_img = pygame.image.load("player.png")  # Replace with your player image path
enemy_img = pygame.image.load("enemy.png")  # Replace with your enemy image path
bullet_img = pygame.image.load("bullet.png")  # Replace with your bullet image path
background = pygame.image.load("background.jpg")  # Replace with your background path

# Scale images
player_img = pygame.transform.scale(player_img, (50, 50))
enemy_img = pygame.transform.scale(enemy_img, (50, 50))
bullet_img = pygame.transform.scale(bullet_img, (10, 20))
background = pygame.transform.scale(background, (WIDTH, HEIGHT))

# Game variables
score = 0
game_over = False

# Classes
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = player_img
        self.rect = self.image.get_rect()
        self.rect.center = (WIDTH // 2, HEIGHT - 60)
        self.speed_x = 0

    def update(self):
        self.speed_x = 0
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.speed_x = -PLAYER_SPEED
        if keys[pygame.K_RIGHT]:
            self.speed_x = PLAYER_SPEED
        self.rect.x += self.speed_x
        # Keep the player within screen bounds
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH

    def shoot(self):
        bullet = Bullet(self.rect.centerx, self.rect.top)
        bullets.add(bullet)
        all_sprites.add(bullet)

class Enemy(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = enemy_img
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, WIDTH - self.rect.width)
        self.rect.y = random.randint(-100, -40)
        self.speed_y = ENEMY_SPEED

    def update(self):
        self.rect.y += self.speed_y
        if self.rect.top > HEIGHT:
            self.rect.y = random.randint(-100, -40)
            self.rect.x = random.randint(0, WIDTH - self.rect.width)

class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = bullet_img
        self.rect = self.image.get_rect()
        self.rect.center = (x, y)
        self.speed_y = BULLET_SPEED

    def update(self):
        self.rect.y += self.speed_y
        # Remove the bullet if it goes off-screen
        if self.rect.bottom < 0:
            self.kill()

# Create sprite groups
all_sprites = pygame.sprite.Group()
players = pygame.sprite.Group()
enemies = pygame.sprite.Group()
bullets = pygame.sprite.Group()

# Create player
player = Player()
all_sprites.add(player)
players.add(player)

# Spawn initial enemies
for _ in range(8):
    enemy = Enemy()
    all_sprites.add(enemy)
    enemies.add(enemy)

# Main game loop
running = True
while running:
    clock.tick(FPS)
    
    # Events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                player.shoot()

    # Update
    if not game_over:
        all_sprites.update()

        # Check for collisions
        hits = pygame.sprite.groupcollide(enemies, bullets, True, True)
        for hit in hits:
            score += 10
            # Respawn a new enemy
            enemy = Enemy()
            all_sprites.add(enemy)
            enemies.add(enemy)

        # Check if any enemies hit the player
        hits = pygame.sprite.spritecollide(player, enemies, False)
        if hits:
            game_over = True

    # Draw
    screen.blit(background, (0, 0))
    all_sprites.draw(screen)

    # Draw score
    font = pygame.font.SysFont("Arial", 24)
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))

    # Game Over screen
    if game_over:
        font = pygame.font.SysFont("Arial", 48)
        game_over_text = font.render("GAME OVER", True, RED)
        screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 2))
        restart_text = font.render("Press R to Restart", True, RED)
        screen.blit(restart_text, (WIDTH // 2 - restart_text.get_width() // 2, HEIGHT // 2 + 50))

    # Update display
    pygame.display.flip()

    # Restart the game
    keys = pygame.key.get_pressed()
    if keys[pygame.K_r]:
        game_over = False
        score = 0
        for enemy in enemies:
            enemy.kill()
        for _ in range(8):
            enemy = Enemy()
            all_sprites.add(enemy)
            enemies.add(enemy)

pygame.quit()
sys.exit()
