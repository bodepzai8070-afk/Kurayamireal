import pygame
import math
import random

# Khởi tạo Pygame
pygame.init()

# Cài đặt màn hình
SCREEN_WIDTH = 1000
SCREEN_HEIGHT = 700
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("HTPC Tower Defense - Bảo vệ lãnh thổ!")

# Màu sắc pixel art style
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
BROWN = (139, 69, 19)
GRAY = (128, 128, 128)
DARK_GREEN = (0, 128, 0)
YELLOW = (255, 255, 0)
PURPLE = (128, 0, 128)
ORANGE = (255, 165, 0)
PINK = (255, 192, 203)

# Clock
clock = pygame.time.Clock()
FPS = 60

# Font
font_small = pygame.font.SysFont('monospace', 16)
font = pygame.font.SysFont('monospace', 20)
font_big = pygame.font.SysFont('monospace', 24, bold=True)

class Player:
    def __init__(self):
        self.max_lives = 20
        self.lives = 20
        self.armor = 0  # Giảm damage nhận vào
        self.damage_mult = 1.0  # Sức mạnh tháp
    
    def take_damage(self, damage):
        actual_damage = max(1, damage - self.armor)
        self.lives -= actual_damage
        return actual_damage

class Enemy:
    def __init__(self, wave):
        base_health = 15
        multiplier = 2 ** (wave // 5)  # x2 mỗi 5 wave
        if wave % 10 == 0:  # Boss wave
            multiplier *= 2.5  # x2.5 mỗi 10 wave
            self.is_boss = True
            self.size = 35
            self.speed = 0.8
            self.boss_name = random.choice(["Ly", "Trâm", "Hân", "Sói"])
        else:
            self.is_boss = False
            self.size = 20
            self.speed = 1.5 + (wave * 0.15)
        
        self.health = base_health * multiplier
        self.max_health = self.health
        self.x = 0
        self.y = SCREEN_HEIGHT // 2 - 50
        self.path_index = 0
        self.path = [(0, SCREEN_HEIGHT//2), (250, SCREEN_HEIGHT//2), (250, 120), 
                    (600, 120), (600, 450), (SCREEN_WIDTH-50, 450)]
        self.money_reward = 15 + (wave * 2)
    
    def update(self):
        if self.path_index < len(self.path) - 1:
            target_x, target_y = self.path[self.path_index + 1]
            dx = target_x - self.x
            dy = target_y - self.y
            dist = math.hypot(dx, dy)
            if dist < self.speed:
                self.path_index += 1
            else:
                self.x += (dx / dist) * self.speed
                self.y += (dy / dist) * self.speed
        else:
            return True  # Đến cuối đường
        return False
    
    def draw(self, screen):
        if self.is_boss:
            # Boss pixel art lớn hơn
            pygame.draw.rect(screen, PURPLE, (self.x - self.size//2, self.y - self.size//2, self.size, self.size))
            pygame.draw.rect(screen, PINK, (self.x - self.size//2 + 3, self.y - self.size//2 + 3, self.size-6, self.size-6))
            # Tên boss
            name_text = font_small.render(self.boss_name, True, WHITE)
            screen.blit(name_text, (self.x - 15, self.y - 25))
        else:
            pygame.draw.rect(screen, RED, (self.x - self.size//2, self.y - self.size//2, self.size, self.size))
        
        # Thanh máu dài hơn cho boss
        bar_width = 25 if self.is_boss else 20
        health_ratio = self.health / self.max_health
        pygame.draw.rect(screen, RED, (self.x - bar_width//2, self.y - 20, bar_width, 6))
        pygame.draw.rect(screen, GREEN, (self.x - bar_width//2, self.y - 20, bar_width * health_ratio, 6))
    
    def take_damage(self, damage):
        self.health -= damage
        return self.health <= 0

class Tower:
    def __init__(self, x, y, tower_type=0):
        self.x = x
        self.y = y
        self.tower_type = tower_type
        self.range = 160
        self.size = 28
        self.shoot_timer = 0
        
        # Stats theo loại tháp
        tower_stats = [
            {"damage": 30, "cooldown": 50, "color": GRAY, "name": "Basic"},  # 0
            {"damage": 50, "cooldown": 80, "color": BLUE, "name": "Sniper"},   # 1
            {"damage": 20, "cooldown": 20, "color": YELLOW, "name": "MachineGun"}, # 2
            {"damage": 80, "cooldown": 120, "color": ORANGE, "name": "Rocket"}     # 3
        ]
        self.stats = tower_stats[tower_type]
    
    def find_target(self, enemies):
        closest = None
        min_dist = float('inf')
        for enemy in enemies:
            dist = math.hypot(enemy.x - self.x, enemy.y - self.y)
            if dist <= self.range and dist < min_dist:
                min_dist = dist
                closest = enemy
        return closest
    
    def update(self, enemies, damage_mult):
        if self.shoot_timer > 0:
            self.shoot_timer -= 1
            return
        
        target = self.find_target(enemies)
        if target:
            # Bắn laser
            pygame.draw.line(screen, self.stats["color"], (self.x, self.y), (target.x, target.y), 4)
            pygame.draw.circle(screen, WHITE, (int(target.x), int(target.y)), 5)
            damage = self.stats["damage"] * damage_mult
            target.take_damage(damage)
            self.shoot_timer = self.stats["cooldown"]
    
    def draw(self, screen):
        # Vẽ tháp pixel art
        pygame.draw.rect(screen, self.stats["color"], (self.x - self.size//2, self.y - self.size//2, self.size, self.size))
        pygame.draw.circle(screen, BROWN, (self.x, self.y - 8), 10)
        # Icon loại tháp
        type_icons = [GRAY, BLUE, YELLOW, ORANGE]
        pygame.draw.circle(screen, type_icons[self.tower_type], (self.x, self.y), 6)

# Đường đi
path = [(0, SCREEN_HEIGHT//2), (250, SCREEN_HEIGHT//2), (250, 120), (600, 120), (600, 450), (SCREEN_WIDTH-50, 450)]

def draw_path(screen):
    for i in range(len(path) - 1):
        pygame.draw.line(screen, BROWN, path[i], path[i+1], 45)
    # Điểm spawn pixel art
    pygame.draw.circle(screen, ORANGE, path[0], 20)

# Shop
tower_prices = [50, 120, 180, 300]
upgrade_prices = {"lives": 80, "armor": 100, "damage": 150}

def draw_shop(screen, money, selected_tower=0):
    shop_rect = pygame.Rect(SCREEN_WIDTH - 220, 20, 200, 300)
    pygame.draw.rect(screen, GRAY, shop_rect)
    pygame.draw.rect(screen, WHITE, shop_rect, 3)
    
    # Tiêu đề
    shop_title = font.render("SHOP HTPC", True, BLACK)
    screen.blit(shop_title, (shop_rect.x + 10, shop_rect.y + 10))
    
    # Tháp
    y_offset = 50
    for i, price in enumerate(tower_prices):
        color = ["GRAY", "BLUE", "YELLOW", "ORANGE"][i]
        text = font_small.render(f"{i+1}: {['Basic','Sniper','MGun','Rocket'][i]} ${price}", True, BLACK)
        screen.blit(text, (shop_rect.x + 10, shop_rect.y + y_offset))
        if i == selected_tower:
            pygame.draw.rect(screen, YELLOW, (shop_rect.x + 5, shop_rect.y + y_offset - 2, 190, 22), 2)
        y_offset += 30
    
    # Nâng cấp
    y_offset += 10
    upgrade_text = font_small.render("UPGRADES:", True, BLACK)
    screen.blit(upgrade_text, (shop_rect.x + 10, shop_rect.y + y_offset))
    y_offset += 25
    
    lives_text = font_small.render(f"Lives +5: ${upgrade_prices['lives']}", True, BLACK)
    screen.blit(lives_text, (shop_rect.x + 10, shop_rect.y + y_offset))
    y_offset += 25
    
    armor_text = font_small.render(f"Armor +1: ${upgrade_prices['armor']}", True, BLACK)
    screen.blit(armor_text, (shop_rect.x + 10, shop_rect.y + y_offset))
    y_offset += 25
    
    damage_text = font_small.render(f"Damage x0.2: ${upgrade_prices['damage']}", True, BLACK)
    screen.blit(damage_text, (shop_rect.x + 10, shop_rect.y + y_offset))

# Vẽ cốt truyện
def draw_story(screen, wave):
    if wave == 1:
        story = [
            "Trên mãnh đất tên Chàng Béo...",
            "Team HTPC với lãnh đạo Hiếu TV & Nấm Gamer",
            "bỗng NYC tấn công: Ly, Trâm, Hân, Thảo, Sói!",
            "Họ hét: 'HTPC đầu hàng hoặc chết!'",
            "CHIẾN ĐẤU!"
        ]
        for i, line in enumerate(story):
            text = font_small.render(line, True, YELLOW)
            screen.blit(text, (50, 50 + i * 25))
    
    if wave % 10 == 0:
        boss_dialogue = [
            f"BOSS {random.choice(['Ly','Trâm','Hân','Sói'])}:",
            "'Các ngươi không đầu hàng sao?'",
            "'HÃY CHẾT ĐI!'"
        ]
        for i, line in enumerate(boss_dialogue):
            text = font.render(line, True, RED)
            screen.blit(text, (SCREEN_WIDTH//2 - 150, SCREEN_HEIGHT//2 + i * 30 - 30))

# Game variables
player = Player()
enemies = []
towers = []
wave = 0
money = 150
selected_tower = 0  # 0-3
wave_timer = 0
wave_cooldown = 900  # 15s
spawning = False
enemies_to_spawn = 0
boss_dialog_timer = 0

# Story popup timer
story_timer = 300

running = True
while running:
    mx, my = pygame.mouse.get_pos()
    screen.fill(DARK_GREEN)
    draw_path(screen)
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:  # Left click
                # Shop click
                shop_rect = pygame.Rect(SCREEN_WIDTH - 220, 20, 200, 300)
                if shop_rect.collidepoint(mx, my):
                    # Chọn tower type
                    if 70 <= my <= 190:  # Tower buttons
                        selected_tower = (my - 70) // 30
                    # Upgrade buttons
                    elif 230 <= my <= 340:
                        btn_index = (my - 230) // 25
                        if btn_index == 0 and money >= upgrade_prices["lives"]:
                            money -= upgrade_prices["lives"]
                            player.max_lives += 5
                            player.lives = min(player.lives + 5, player.max_lives)
                        elif btn_index == 1 and money >= upgrade_prices["armor"]:
                            money -= upgrade_prices["armor"]
                            player.armor += 1
                        elif btn_index == 2 and money >= upgrade_prices["damage"]:
                            money -= upgrade_prices["damage"]
                            player.damage_mult += 0.2
                
                else:  # Đặt tháp
                    too_close = False
                    for px, py in path:
                        if math.hypot(mx - px, my - py) < 40:
                            too_close = True
                            break
                    if not too_close and money >= tower_prices[selected_tower]:
                        towers.append(Tower(mx, my, selected_tower))
                        money -= tower_prices[selected_tower]
    
    # Wave system
    wave_timer += 1
    if wave_timer > wave_cooldown and not spawning and len(enemies) == 0:
        wave += 1
        enemies_to_spawn = 8 + wave * 2
        if wave % 10 == 0:
            enemies_to_spawn += 1  # Boss
        spawning = True
        wave_timer = 0
        boss_dialog_timer = 180  # 3s
    
    if spawning:
        spawn_interval = max(25, 60 - wave // 5)
        if wave_timer % spawn_interval == 0 and enemies_to_spawn > 0:
            enemies.append(Enemy(wave))
            enemies_to_spawn -= 1
        if enemies_to_spawn == 0:
            spawning = False
    
    # Update enemies
    for enemy in enemies[:]:
        if enemy.update():
            player.take_damage(15 if enemy.is_boss else 10)
            enemies.remove(enemy)
        else:
            enemy.draw(screen)
    
    # Update towers
    for tower in towers:
        tower.update(enemies, player.damage_mult)
        tower.draw(screen)
    
    # Cleanup dead enemies & reward money
    for enemy in enemies[:]:
        if enemy.health <= 0:
            money += enemy.money_reward
            enemies.remove(enemy)
    
    # UI
    lives_text = font.render(f"Lives: {player.lives}/{player.max_lives}", True, WHITE)
    screen.blit(lives_text, (10, 10))
    money_text = font.render(f"Money: ${money}", True, YELLOW)
    screen.blit(money_text, (10, 40))
    wave_text = font.render(f"Wave: {wave}", True, WHITE)
    screen.blit(wave_text, (10, 70))
    
    # Selected tower info
    tower_info = font_small.render(f"Selected: {['Basic','Sniper','MGun','Rocket'][selected_tower]}", True, WHITE)
    screen.blit(tower_info, (10, 100))
    
    draw_shop(screen, money, selected_tower)
    
    # Story & Boss dialogue
    if story_timer > 0:
        story_timer -= 1
        draw_story(screen, 1)
    if boss_dialog_timer > 0:
        boss_dialog_timer -= 1
        draw_story(screen, wave)
    
    # Game Over
    if player.lives <= 0:
        go_text = font_big.render("GAME OVER - NYC THẮNG!", True, RED)
        screen.blit(go_text, (SCREEN_WIDTH//2 - 200, SCREEN_HEIGHT//2))
        restart_text = font.render("Press R to restart", True, WHITE)
        screen.blit(restart_text, (SCREEN_WIDTH//2 - 80, SCREEN_HEIGHT//2 + 40))
        pygame.display.flip()
        keys = pygame.key.get_pressed()
        if keys[pygame.K_r]:
            # Restart (simple reset)
            player = Player()
            enemies = []
            towers = []
            wave = 0
            money = 150
            wave_timer = 0
            spawning = False
            story_timer = 300
    
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
