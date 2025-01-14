# python11328252

import pygame
import random
import time
import sys

# 初始化 Pygame
pygame.init()

# 設定視窗大小和遊戲參數
INITIAL_GRID_SIZE = 10  # 初始迷宮大小
CELL_SIZE = 50
FPS = 60
WINDOW_WIDTH, WINDOW_HEIGHT = 800, 800  # 遊戲視窗大小

# 顏色定義
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)

# 初始化 Pygame 繪圖
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("迷宮逃脫 with Pygame")
clock = pygame.time.Clock()

def generate_maze(grid_size):
    """生成可解的迷宮，使用深度優先搜尋法並確保終點可達"""
    maze = [["#" for _ in range(grid_size)] for _ in range(grid_size)]
    visited = [[False for _ in range(grid_size)] for _ in range(grid_size)]

    def carve_path(x, y):
        """使用 DFS 刻畫迷宮路徑"""
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]  # 上下左右
        random.shuffle(directions)  # 隨機排列方向
        for dx, dy in directions:
            nx, ny = x + dx * 2, y + dy * 2
            if 1 <= nx < grid_size - 1 and 1 <= ny < grid_size - 1 and not visited[nx][ny]:
                visited[nx][ny] = True
                maze[nx][ny] = " "
                maze[x + dx][y + dy] = " "
                carve_path(nx, ny)

    # 設置起點
    maze[1][1] = "S"
    visited[1][1] = True
    carve_path(1, 1)

    # 設置終點
    end_x, end_y = grid_size - 2, grid_size - 2
    maze[end_x][end_y] = "E"

    # 確保終點周圍有至少一條通路
    for dx, dy in [(0, -1), (-1, 0), (0, 1), (1, 0)]:
        nx, ny = end_x + dx, end_y + dy
        if 1 <= nx < grid_size - 1 and 1 <= ny < grid_size - 1:
            maze[nx][ny] = " "

    return maze

def draw_maze(maze, player_pos, offset):
    """繪製迷宮和玩家"""
    for x in range(len(maze)):
        for y in range(len(maze[x])):
            cell = maze[x][y]
            rect = pygame.Rect(y * CELL_SIZE - offset[0], x * CELL_SIZE - offset[1], CELL_SIZE, CELL_SIZE)
            
            if cell == "#":  # 牆壁
                pygame.draw.rect(screen, BLACK, rect)
            elif cell == "S":  # 起點
                pygame.draw.rect(screen, GREEN, rect)
            elif cell == "E":  # 出口
                pygame.draw.rect(screen, RED, rect)
            
            # 玩家
            if (x, y) == player_pos:
                pygame.draw.rect(screen, BLUE, rect)

def move_player(player_pos, direction, maze):
    """移動玩家並檢查牆壁"""
    x, y = player_pos
    if direction == "UP":
        new_pos = (x - 1, y)
    elif direction == "DOWN":
        new_pos = (x + 1, y)
    elif direction == "LEFT":
        new_pos = (x, y - 1)
    elif direction == "RIGHT":
        new_pos = (x, y + 1)
    else:
        return player_pos  # 無效指令

    # 檢查是否撞牆
    if maze[new_pos[0]][new_pos[1]] == "#":
        return player_pos  # 撞牆

    return new_pos

def adjust_camera(player_pos, grid_size):
    """計算攝影機的偏移值，使玩家保持在視窗中心"""
    max_offset_x = max(0, grid_size * CELL_SIZE - WINDOW_WIDTH)
    max_offset_y = max(0, grid_size * CELL_SIZE - WINDOW_HEIGHT)

    offset_x = max(0, min(player_pos[1] * CELL_SIZE - WINDOW_WIDTH // 2, max_offset_x))
    offset_y = max(0, min(player_pos[0] * CELL_SIZE - WINDOW_HEIGHT // 2, max_offset_y))
    return offset_x, offset_y

def main():
    grid_size = INITIAL_GRID_SIZE
    level = 1
    player_pos = [1, 1]  # 確保玩家從起點 (1, 1) 開始

    last_move_time = 0  # 上次移動的時間
    move_delay = 200  # 設定移動的延遲時間 (毫秒)
    can_move = True  # 用來控制是否可以移動

    while True:
        maze = generate_maze(grid_size)
        player_pos = [1, 1]  # 每次生成新迷宮時，將玩家位置重置為起點
        start_time = time.time()

        running = True
        while running:
            screen.fill(WHITE)  # 清空畫面

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()

            # 持續按下方向鍵移動
            keys = pygame.key.get_pressed()

            # 檢查是否可以移動（按住方向鍵並處於移動間隔內）
            current_time = pygame.time.get_ticks()  # 獲取當前時間（毫秒）

            if keys[pygame.K_UP]:
                if current_time - last_move_time > move_delay:  # 如果已過延遲時間
                    player_pos = move_player(player_pos, "UP", maze)
                    last_move_time = current_time
            if keys[pygame.K_DOWN]:
                if current_time - last_move_time > move_delay:
                    player_pos = move_player(player_pos, "DOWN", maze)
                    last_move_time = current_time
            if keys[pygame.K_LEFT]:
                if current_time - last_move_time > move_delay:
                    player_pos = move_player(player_pos, "LEFT", maze)
                    last_move_time = current_time
            if keys[pygame.K_RIGHT]:
                if current_time - last_move_time > move_delay:
                    player_pos = move_player(player_pos, "RIGHT", maze)
                    last_move_time = current_time

            # 計算攝影機的偏移值
            offset = adjust_camera(player_pos, grid_size)

            # 繪製迷宮
            draw_maze(maze, tuple(player_pos), offset)

            # 檢查是否到達出口
            if maze[player_pos[0]][player_pos[1]] == "E":
                end_time = time.time()
                total_time = round(end_time - start_time, 2)
                print(f"恭喜！你完成了第 {level} 關！用時 {total_time} 秒。")
                level += 1
                grid_size += 2  # 增加迷宮大小
                running = False

            pygame.display.flip()
            clock.tick(FPS)

if __name__ == "__main__":
    main()
