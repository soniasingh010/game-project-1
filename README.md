# game-project-1

# IMPORTING LIBRARIES
import pygame
import random

# MAIN MENU
def menu(start):
    run=True
    while run:
        start.fill((0,0,0))
        tetris_front(start, 'TETRIS', 80, (204,0,0))
        front_text(start, 'Press Enter To Start', 50, (200,200,200))
        pygame.display.update()
        for i in pygame.event.get():
            if i.type == pygame.KEYDOWN:
                main(start)
            if i.type == pygame.QUIT:
                run = False
pygame.font.init()

# BLOCKS
I = [['XX0XX',
      'XX0XX',
      'XX0XX',
      'XX0XX',
      'XXXXX'],
     ['XXXXX',
      '0000X',
      'XXXXX',
      'XXXXX',
      'XXXXX']]

J = [['XXXXX',
      'X0XXX',
      'X000X',
      'XXXXX',
      'XXXXX'],
     ['XXXXX',
      'XX00X',
      'XX0XX',
      'XX0XX',
      'XXXXX'],
     ['XXXXX',
      'XXXXX',
      'X000X',
      'XXX0X',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'XX0XX',
      'X00XX',
      'XXXXX']]

L = [['XXXXX',
      'XXX0X',
      'X000X',
      'XXXXX',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'XX0XX',
      'XX00X',
      'XXXXX'],
     ['XXXXX',
      'XXXXX',
      'X000X',
      'X0XXX',
      'XXXXX'],
     ['XXXXX',
      'X00XX',
      'XX0XX',
      'XX0XX',
      'XXXXX']]

O = [['XXXXX',
      'XXXXX',
      'X00XX',
      'X00XX',
      'XXXXX']]

S = [['XXXXX',
      'XXXXX',
      'X00XX',
      'X00XXX',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'XX00X',
      'XXX0X',
      'XXXXX']]

T = [['XXXXX',
      'XX0XX',
      'X000X',
      'XXXXX',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'XX00X',
      'XX0XX',
      'XXXXX'],
     ['XXXXX',
      'XXXXX',
      'X000X',
      'XX0XX',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'X00XX',
      'XX0XX',
      'XXXXX']]

Z = [['XXXXX',
      'XXXXX',
      'X00XX',
      'XX00X',
      'XXXXX'],
     ['XXXXX',
      'XX0XX',
      'X00XX',
      'X0XXX',
      'XXXXX']]

blocks = [I, J, L, O, S, T, Z]
block_colors = [(255, 255, 0),(0, 255, 0),(0, 255, 255), (255, 128, 0), (0, 0, 255), (255, 0, 255),(255, 0, 0)]

def GRID(fix_pos={}):
    grid=[[(0, 0, 0) for _ in range(10)] for _ in range(20)]

    for i in range(len(grid)):
        for j in range(len(grid[i])):
            if (j, i) in fix_pos:
                c = fix_pos[(j, i)]
                grid[i][j] = c
    return grid

class Piece(object):
    def __init__(self, a, b, shape):
        self.a = a
        self.b = b
        self.shape = shape
        self.color = block_colors[blocks.index(shape)]
        self.rotation = 0

def change_format(shape):
    pos= []
    format = shape.shape[shape.rotation % len(shape.shape)]

    for i, line in enumerate(format):
        row = list(line)
        for j, column in enumerate(row):
            if column == '0':
                pos.append((shape.a + j, shape.b + i))

    for i, K in enumerate(pos):
        pos[i] = (K[0] - 2, K[1] - 4)

    return pos

def valid_space(shape, grid):
    allow_pos = [[(a, b) for a in range(10) if grid[b][a] == (0,0,0)] for b in range(20)]
    allow_pos = [a for res in allow_pos for a in res]

    designed = change_format(shape)

    for x in designed:
        if x not in allow_pos:
            if x[1] > -1:
                return False
    return True

# CHECKING FOR LOST POSITION
def check_lost(pos):
    for res in pos:
        num1, num2 = res
        if num2< 1:
            return True
    return False

# NEXT BLOCK 
def next_shape():
    return Piece(5, 0, random.choice(blocks))

# TEXT
def tetris_front(surface, text, size, color):
    font = pygame.font.SysFont("Times New Roman", size, bold=True)   
    label = font.render(text, 1, color)
    surface.blit(label,(tlx + W/2 - (label.get_width()/2), tly + H/4 - label.get_height()/2))

def front_text(surface, text, size, color):
    font = pygame.font.SysFont("Times New Roman", size, bold=True)   
    label = font.render(text, 1, color)
    surface.blit(label,(tlx + W/2 - (label.get_width()/2), tly + H/2 - label.get_height()/2))

# DESIGNING GRID
def draw_grid(surface, grid):
    sx = tlx
    sy = tly

    for i in range(len(grid)):
        pygame.draw.line(surface, (128,128,128), (sx, sy+ i*size_of_block), (sx+W, sy+ i*size_of_block))
        for j in range(len(grid[i])):
            pygame.draw.line(surface, (128, 128, 128), (sx + j*size_of_block, sy),(sx + j*size_of_block, sy + H))

# CLEARNING THE ROWS
def remove_blocks(grid, locked):
    flag1 = 0
    for _ in range (len(grid)-1, -1, -1):
        hori = grid[_]
        if (0, 0, 0) not in hori:
            flag1 += 1
            flag2 = _
            for o in range(len(hori)):
                try:
                    del locked[(o, _)]
                except:
                    continue
    if flag1 > 0:
        for m in sorted(list(locked), key=lambda x: x[1])[::-1]:
            x, y = m
            if y < flag2:
                new = (x, y + flag1)
                locked[new] = locked.pop(m)
    return flag1

# SHOWING NEXT BLOCK
def next_block(shape, surface):
    font = pygame.font.SysFont('Comic Sans MS', 30)
    label = font.render('Next Block', 1, (255,255,255))
    sx = tlx+W+28
    sy = tly+H/2-100

    format = shape.shape[shape.rotation % len(shape.shape)]

    for i, line in enumerate(format):
        row = list(line)
        for j, column in enumerate(row):
            if column == '0':
                pygame.draw.rect(surface, shape.color, (sx + j*size_of_block, sy + i*size_of_block, size_of_block, size_of_block), 0)

    surface.blit(label, (sx + 10, sy- 30))

# GAME WINDOW
def window(surface, grid, score=0, last_score=0):
    surface.fill((0, 0, 0))

    pygame.font.init()
    font = pygame.font.SysFont('Algerian', 60)
    label = font.render("Tetris", 1, (255, 255, 255))

    surface.blit(label, (tlx+(W/2)-(label.get_width()/2), 30))

    # CURRENT SCORE
    font=pygame.font.SysFont('Comic Sans MS', 27)
    label=font.render('Your Score: ' + str(score), 1, (255, 255, 255))
    sx= tlx+W+20
    sy=tly+H/2-100
    surface.blit(label, (sx + 20, sy + 160))

    # HIGH SCORE
    label = font.render('High Score: '+ last_score, 1, (255, 255, 255))
    sx = tlx - 248
    sy = tly + 20
    surface.blit(label, (sx + 20, sy + 160))

    for i in range(len(grid)):
        for j in range(len(grid[i])):
            pygame.draw.rect(surface, grid[i][j], (tlx + j*size_of_block, tly + i*size_of_block, size_of_block, size_of_block), 0)

    pygame.draw.rect(surface, (255,0,0), (tlx, tly,W,H), 5)
    draw_grid(surface, grid)

# MAXIMUM SCORE
def high_score():
    with open('game score.txt', 'r') as f:
        lines = f.readlines()
        score = lines[0].strip()
    return score

#SCORE UPDATE
def score_update(score):
    new_score = high_score()
    with open('game score.txt','w') as f:
        if (int(score) > int(new_score)):
            f.write(str(score))
        else:
            f.write(str(new_score))

# MAIN FUNCTION
def main(start):
    fix_pos= {}
    grid = GRID(fix_pos)
    last_score = high_score()

    change_piece = False
    run = True
    current_piece = next_shape()
    next_piece = next_shape()
    clock = pygame.time.Clock()
    time_of_falling = 0
    speed_of_falling = 0.27
    level_time = 0
    score = 0

    while run:
        grid = GRID(fix_pos)
        time_of_falling=time_of_falling+clock.get_rawtime()
        level_time += clock.get_rawtime()
        clock.tick()

        if (level_time/1000 > 5):
            level_time = 0
            if (speed_of_falling > 0.12):
                speed_of_falling=speed_of_falling-0.005

        if (time_of_falling/1000 > speed_of_falling):
            time_of_falling = 0
            current_piece.b+=1
            if not ((valid_space(current_piece, grid)) and current_piece.b > 0):
                current_piece.b-=1
                change_piece = True

        for event in pygame.event.get():
            if (event.type==pygame.QUIT):
                run=False
                pygame.display.quit()

            if (event.type==pygame.KEYDOWN):
                if (event.key==pygame.K_LEFT):
                    current_piece.a-=1
                    if not (valid_space(current_piece, grid)):
                        current_piece.a+=1

                if (event.key==pygame.K_RIGHT):
                    current_piece.a+=1
                    if not (valid_space(current_piece, grid)):
                        current_piece.a-=1

                if (event.key == pygame.K_UP):
                    current_piece.rotation+=1
                    if not (valid_space(current_piece, grid)):
                        current_piece.rotation-=1

                if (event.key==pygame.K_DOWN):
                    current_piece.b+=1
                    if not (valid_space(current_piece, grid)):
                        current_piece.b-=1

        shape_pos = change_format(current_piece)

        for i in range(len(shape_pos)):
            a, b = shape_pos[i]
            if (b > -1):
                grid[b][a] = current_piece.color

        if change_piece:
            for pos in shape_pos:
                p = (pos[0], pos[1])
                fix_pos[p] = current_piece.color
            current_piece = next_piece
            next_piece = next_shape()
            change_piece = False
            score += remove_blocks(grid, fix_pos) * 10

        window(start, grid, score, last_score)
        next_block(next_piece,start)
        pygame.display.update()

        if check_lost(fix_pos):
            front_text(start, "GAME OVER!!", 80, (200,200,200))
            pygame.display.update()
            pygame.time.delay(1000)

            run = False
            score_update(score)

# START

# screen width and height 
main_screen_width = 830
main_screen_height = 700

W = 300  
H = 600

# size of blocks
size_of_block=30
tlx=(main_screen_width - W) // 2
tly=(main_screen_height - H)

start= pygame.display.set_mode((main_screen_width, main_screen_height))
pygame.display.set_caption('TETRIS')
menu(start)
