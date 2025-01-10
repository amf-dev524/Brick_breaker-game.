# Brick_breaker-game.
#In this game you have to break bricks and you have to score high its a simple python game.
import pygame
import random

# Initialize pygame
pygame.init()

# Screen dimensions
width, height = 800, 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Brick Breaker")

# Colors
white = (255, 255, 255)
black = (0, 0, 0)
red = (255, 0, 0)
green = (0, 255, 0)
blue = (0, 0, 255)
yellow = (255, 255, 0)
orange = (255, 165, 0)
brick_colors = [yellow, orange, red]  # Colors representing brick strength

# Font styles
font = pygame.font.SysFont("comicsansms", 30)
large_font = pygame.font.SysFont("comicsansms", 50)

# Game clock
clock = pygame.time.Clock()
fps = 60

# High score tracker
high_score = 0

def create_bricks(rows, cols, strength):
    brick_width = width // cols
    brick_height = 30
    brick_padding = 5
    bricks = []
    for row in range(rows):
        for col in range(cols):
            brick_x = col * (brick_width + brick_padding) + brick_padding // 2
            brick_y = row * (brick_height + brick_padding) + brick_padding // 2
            direction = random.choice([-1, 1])  # Random movement direction
            bricks.append({
                "rect": pygame.Rect(brick_x, brick_y, brick_width, brick_height),
                "hits": strength,
                "direction": direction
            })
    return bricks, brick_width, brick_height

def move_bricks(bricks, min_y, max_y):
    for brick in bricks:
        brick["rect"].y += brick["direction"]
        if brick["rect"].top <= min_y or brick["rect"].bottom >= max_y:
            brick["direction"] *= -1  # Reverse direction

def draw_bricks(bricks):
    for brick in bricks:
        color = brick_colors[min(brick["hits"] - 1, len(brick_colors) - 1)]
        pygame.draw.rect(screen, color, brick["rect"])

def display_score(score, level, high_score):
    score_text = font.render(f"Score: {score}  Level: {level}  High Score: {high_score}", True, white)
    screen.blit(score_text, (10, 10))

def game_over_screen(score, high_score):
    screen.fill(black)
    over_text = large_font.render("GAME OVER", True, red)
    score_text = font.render(f"Final Score: {score}", True, white)
    high_score_text = font.render(f"High Score: {high_score}", True, green)
    restart_text = font.render("Press R to Restart or Q to Quit", True, yellow)

    screen.blit(over_text, (width // 2 - over_text.get_width() // 2, height // 3))
    screen.blit(score_text, (width // 2 - score_text.get_width() // 2, height // 2))
    screen.blit(high_score_text, (width // 2 - high_score_text.get_width() // 2, height // 1.7))
    screen.blit(restart_text, (width // 2 - restart_text.get_width() // 2, height // 1.4))

    pygame.display.update()

    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    waiting = False
                    game_loop()
                if event.key == pygame.K_q:
                    pygame.quit()
                    exit()

def game_loop():
    global high_score

    # Game variables
    paddle_width = 100
    paddle_height = 10
    paddle_color = blue
    paddle_speed = 10

    ball_radius = 10
    ball_color = red
    ball_speed_x = 4
    ball_speed_y = -4

    rows, cols = 5, 8  # Initial number of bricks
    brick_strength = 1  # Initial strength of bricks
    level = 1
    score = 0

    while True:
        # Create bricks for each level
        bricks, brick_width, brick_height = create_bricks(rows, cols, brick_strength)

        # Paddle setup
        paddle = pygame.Rect(width // 2 - paddle_width // 2, height - 20, paddle_width, paddle_height)

        # Ball setup
        ball = pygame.Rect(width // 2, height // 2, ball_radius * 2, ball_radius * 2)
        ball_dx, ball_dy = ball_speed_x, ball_speed_y

        # Level loop
        while True:
            screen.fill(black)

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    exit()

            # Paddle movement
            keys = pygame.key.get_pressed()
            if keys[pygame.K_LEFT] and paddle.left > 0:
                paddle.move_ip(-paddle_speed, 0)
            if keys[pygame.K_RIGHT] and paddle.right < width:
                paddle.move_ip(paddle_speed, 0)

            # Ball movement
            ball.move_ip(ball_dx, ball_dy)

            # Ball collision with walls
            if ball.left <= 0 or ball.right >= width:
                ball_dx = -ball_dx
            if ball.top <= 0:
                ball_dy = -ball_dy

            # Ball collision with paddle
            if ball.colliderect(paddle):
                ball_dy = -ball_dy

            # Ball collision with bricks
            for brick in bricks[:]:
                if ball.colliderect(brick["rect"]):
                    ball_dy = -ball_dy
                    brick["hits"] -= 1
                    if brick["hits"] <= 0:
                        bricks.remove(brick)
                        score += 10
                        if score > high_score:
                            high_score = score
                    break

            # Move bricks
            move_bricks(bricks, 0, height // 2)

            # Check if the ball is out of bounds
            if ball.bottom >= height:
                game_over_screen(score, high_score)

            # Draw everything
            pygame.draw.rect(screen, paddle_color, paddle)
            pygame.draw.ellipse(screen, ball_color, ball)
            draw_bricks(bricks)
            display_score(score, level, high_score)

            # Check win condition
            if not bricks:
                level += 1
                brick_strength += 1  # Increase brick strength
                rows += 1  # Add more bricks
                break  # Move to the next level

            pygame.display.update()
            clock.tick(fps)

if __name__ == "__main__":
    game_loop()
