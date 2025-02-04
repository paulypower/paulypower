import pygame
import random

# Initialize Pygame
pygame.init()

# Fullscreen dimensions
screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
screen_width, screen_height = screen.get_size()
pygame.display.set_caption("Interactive MONIAC Simulation with Political Compass")

# Colors
white = (255, 255, 255)
black = (0, 0, 0)
blue = (0, 0, 255)
red = (255, 0, 0)
green = (0, 255, 0)
light_blue = (173, 216, 230)
light_yellow = (255, 255, 224)
light_coral = (240, 128, 128)
light_green = (144, 238, 144)

# Fonts
font = pygame.font.SysFont('Arial', 30)

# Simulation variables
consumption = 200
investment = 100
government_spending = 150
net_exports = 50
savings = 80
imports = 60
tax_rate = 0.2  # Initial tax rate (20%)
total_income = consumption + investment + government_spending + net_exports - imports

# Water tanks
num_tanks = 6  # Total income will be a separate tank
tank_width = 100  # Reduced width to fit all tanks
tank_height = 300
tank_gap = (screen_width - (num_tanks * tank_width)) // (num_tanks + 1)  # Calculate equal gaps for centering

# Sliders
slider_width = screen_width // 2
slider_height = 75  # Increased height for better usability
slider_gap = 40

# Handle variables
handle_radius = 25  # Increased radius for better usability
handle_positions = {
    "consumption": (screen_width // 2 - slider_width // 2 + (consumption / 300) * slider_width, 100 + slider_height // 2),
    "investment": (screen_width // 2 - slider_width // 2 + (investment / 150) * slider_width, 200 + slider_height // 2),
    "government_spending": (screen_width // 2 - slider_width // 2 + (government_spending / 200) * slider_width, 300 + slider_height // 2),
    "net_exports": (screen_width // 2 - slider_width // 2 + (net_exports / 100) * slider_width, 400 + slider_height // 2),
    "savings": (screen_width // 2 - slider_width // 2 + (savings / 150) * slider_width, 500 + slider_height // 2),
    "imports": (screen_width // 2 - slider_width // 2 + (imports / 100) * slider_width, 600 + slider_height // 2),
    "tax_rate": (screen_width // 2 - slider_width // 2 + (tax_rate * slider_width), 700 + slider_height // 2)
}
dragging = None

# Political compass variables
compass_width = 600  # Increased size by 50%
compass_height = 600  # Increased size by 50%
compass_center = (screen_width // 2, screen_height - compass_height // 2 - 30)
marker_pos = compass_center

# Function to draw text
def draw_text(text, font, color, surface, x, y):
    text_obj = font.render(text, True, color)
    text_rect = text_obj.get_rect()
    text_rect.center = (x, y)
    surface.blit(text_obj, text_rect)

# Function to draw slider
def draw_slider(surface, color, x, y, width, height, level, max_level, text, handle_pos):
    pygame.draw.rect(surface, black, (x, y, width, height), 2)
    fill_width = handle_pos[0] - x
    pygame.draw.rect(surface, color, (x, y, fill_width, height))
    pygame.draw.circle(surface, black, handle_pos, handle_radius)
    draw_text(text, font, black, surface, x + width / 2, y - 20)

# Function to draw tank
def draw_tank(surface, x, y, width, height, level, max_level):
    pygame.draw.rect(surface, black, (x, y, width, height), 2)
    fill_height = height * (level / max_level)
    pygame.draw.rect(surface, blue, (x, y + height - fill_height, width, fill_height))

# Function to draw political compass with colored quadrants
def draw_compass(surface, x, y, width, height, marker_pos):
    # Draw quadrants
    pygame.draw.rect(surface, light_blue, (x - width // 2, y - height // 2, width // 2, height // 2))
    pygame.draw.rect(surface, light_yellow, (x, y - height // 2, width // 2, height // 2))
    pygame.draw.rect(surface, light_coral, (x - width // 2, y, width // 2, height // 2))
    pygame.draw.rect(surface, light_green, (x, y, width // 2, height // 2))
    # Draw axes
    pygame.draw.line(surface, black, (x - width // 2, y), (x + width // 2, y), 2)
    pygame.draw.line(surface, black, (x, y - height // 2), (x, y + height // 2), 2)
    # Draw marker
    pygame.draw.circle(surface, red, marker_pos, handle_radius)

# Function to map marker position to slider values with clamping
def map_marker_to_sliders():
    global consumption, investment, government_spending, net_exports, savings, imports, total_income
    consumption = max(0, min(int((marker_pos[0] - (compass_center[0] - compass_width // 2)) / compass_width * 300), 300))
    net_exports = max(0, min(int((marker_pos[0] - (compass_center[0] - compass_width // 2)) / compass_width * 100), 100))
    investment = max(0, min(int((marker_pos[1] - (compass_center[1] - compass_height // 2)) / compass_height * 150), 150))
    government_spending = max(0, min(int((marker_pos[1] - (compass_center[1] - compass_height // 2)) / compass_height * 200), 200))
    savings = max(0, min(int((marker_pos[1] - (compass_center[1] - compass_height // 2)) / compass_height * 150), 150))
    imports = max(0, min(int((marker_pos[0] - (compass_center[0] - compass_width // 2)) / compass_width * 100), 100))
    handle_positions["consumption"] = (screen_width // 2 - slider_width // 2 + (consumption / 300) * slider_width, handle_positions["consumption"][1])
    handle_positions["net_exports"] = (screen_width // 2 - slider_width // 2 + (net_exports / 100) * slider_width, handle_positions["net_exports"][1])
    handle_positions["investment"] = (screen_width // 2 - slider_width // 2 + (investment / 150) * slider_width, handle_positions["investment"][1])
    handle_positions["government_spending"] = (screen_width // 2 - slider_width // 2 + (government_spending / 200) * slider_width, handle_positions["government_spending"][1])
    handle_positions["savings"] = (screen_width // 2 - slider_width // 2 + (savings / 150) * slider_width, handle_positions["savings"][1])
    handle_positions["imports"] = (screen_width // 2 - slider_width // 2 + (imports / 100) * slider_width, handle_positions["imports"][1])
    total_income = consumption + investment + government_spending + net_exports - imports

# Function to map slider values to marker position with clamping
def map_sliders_to_marker():
    global marker_pos
    marker_x = max(compass_center[0] - compass_width // 2, min(((consumption + net_exports - imports) / 400) * compass_width + (compass_center[0] - compass_width // 2), compass_center[0] + compass_width // 2))
    marker_y = max(compass_center[1] - compass_height // 2, min(((investment + government_spending + savings) / 500) * compass_height + (compass_center[1] - compass_height // 2), compass_center[1] + compass_height // 2))
    marker_pos = (marker_x, marker_y)

# Main game loop
running = True
clock = pygame.time.Clock()

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            mouse_x, mouse_y = event.pos
            for key, pos in handle_positions.items():
                if (pos[0] - mouse_x) ** 2 + (pos[1] - mouse_y) ** 2 <= handle_radius ** 2:
                    dragging = key
            if (marker_pos[0] - mouse_x) ** 2 + (marker_pos[1] - mouse_y) ** 2 <= handle_radius ** 2:
                dragging = "marker"
        elif event.type == pygame.MOUSEBUTTONUP:
            dragging = None
        elif event.type == pygame.MOUSEMOTION:
            if dragging:
                mouse_x, mouse_y = event.pos
                if dragging == "marker":
                    marker_pos = (max(compass_center[0] - compass_width // 2, min(mouse_x, compass_center[0] + compass_width // 2)),
                                  max(compass_center[1] - compass_height // 2, min(mouse_y, compass_center[1] + compass_height // 2)))
                    map_marker_to_sliders()
                else:
                    handle_positions[dragging] = (max(screen_width // 2 - slider_width // 2, min(mouse_x, screen_width // 2 + slider_width // 2)), handle_positions[dragging][1])
                    if dragging == "consumption":
                        consumption = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 300)
                    elif dragging == "investment":
                        investment = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 150)
                    elif dragging == "government_spending":
                        government_spending = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 200)
                    elif dragging == "net_exports":
                        net_exports = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 100)
                    elif dragging == "savings":
                        savings = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 150)
                    elif dragging == "imports":
                        imports = int((handle_positions[dragging][0] - (screen_width // 2 - slider_width // 2)) / slider_width * 100)
                    elif dragging == "tax_rate":
                        tax_rate = (handle_positions["tax_rate"][0] - (screen_width // 2 - slider_width // 2)) / slider_width
                    map_sliders_to_marker()
                    total_income = consumption + investment + government_spending + net_exports - imports

    # Draw everything
    screen.fill(white)

    draw_text("MONIAC Simulation with Political Compass", font, black, screen, screen_width // 2, 30)

    # Draw sliders in 3 rows of 2
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 100, slider_width, slider_height, consumption, 300, "Consumption", handle_positions["consumption"])
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 200, slider_width, slider_height, investment, 150, "Investment", handle_positions["investment"])
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 300, slider_width, slider_height, government_spending, 200, "Gov. Spending", handle_positions["government_spending"])
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 400, slider_width, slider_height, net_exports, 100, "Net Exports", handle_positions["net_exports"])
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 500, slider_width, slider_height, savings, 150, "Savings", handle_positions["savings"])
    draw_slider(screen, blue, screen_width // 2 - slider_width // 2, 600, slider_width, slider_height, imports, 100, "Imports", handle_positions["imports"])
    draw_slider(screen, red, screen_width // 2 - slider_width // 2, 700, slider_width, slider_height, tax_rate, 1, "Tax Rate", handle_positions["tax_rate"])

    # Draw tanks with equal spacing in the middle of the screen
    tank_x_start = (screen_width - (num_tanks * tank_width + (num_tanks - 1) * tank_gap)) // 2
    tank_positions = [
        (tank_x_start + i * (tank_width + tank_gap), 800) for i in range(num_tanks)
    ]

    draw_tank(screen, tank_positions[0][0], tank_positions[0][1], tank_width, tank_height, consumption, 300)
    draw_text("Cons", font, black, screen, tank_positions[0][0] + tank_width // 2, tank_positions[0][1] + tank_height + 30)
    
    draw_tank(screen, tank_positions[1][0], tank_positions[1][1], tank_width, tank_height, investment, 150)
    draw_text("Inv", font, black, screen, tank_positions[1][0] + tank_width // 2, tank_positions[1][1] + tank_height + 30)
    
    draw_tank(screen, tank_positions[2][0], tank_positions[2][1], tank_width, tank_height, government_spending, 200)
    draw_text("Spend", font, black, screen, tank_positions[2][0] + tank_width // 2, tank_positions[2][1] + tank_height + 30)
    
    draw_tank(screen, tank_positions[3][0], tank_positions[3][1], tank_width, tank_height, net_exports, 100)
    draw_text("Exports", font, black, screen, tank_positions[3][0] + tank_width // 2, tank_positions[3][1] + tank_height + 30)
    
    draw_tank(screen, tank_positions[4][0], tank_positions[4][1], tank_width, tank_height, savings, 150)
    draw_text("Savings", font, black, screen, tank_positions[4][0] + tank_width // 2, tank_positions[4][1] + tank_height + 30)
    
    draw_tank(screen, tank_positions[5][0], tank_positions[5][1], tank_width, tank_height, imports, 100)
    draw_text("Imports", font, black, screen, tank_positions[5][0] + tank_width // 2, tank_positions[5][1] + tank_height + 30)

    # Draw total income tank in its original position with double width
    draw_tank(screen, screen_width // 2 - tank_width, 900 + tank_height, tank_width * 2, tank_height, total_income, 800)
    draw_text("Total Income", font, black, screen, screen_width // 2, 900 + tank_height + 30)

    # Draw political compass
    draw_compass(screen, compass_center[0], compass_center[1], compass_width, compass_height, marker_pos)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
