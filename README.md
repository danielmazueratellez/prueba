import pygame
import random
from tkinter import messagebox, Tk

# Inicializar pygame
pygame.init()

# Configuración de la pantalla
TAMANO_CELDA = 30
FILAS, COLUMNAS = 10, 10
ANCHO, ALTO = COLUMNAS * TAMANO_CELDA, FILAS * TAMANO_CELDA + 50  # Espacio para la puntuación
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Pac-Man")

# Colores
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
ROJO = (255, 0, 0)
AMARILLO = (255, 255, 0)
AZUL = (0, 0, 255)

# Fuente para la puntuación
fuente = pygame.font.Font(None, 36)

def generar_laberinto():
    laberinto = [[0 for _ in range(COLUMNAS)] for _ in range(FILAS)]
    
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if random.random() < 0.2:
                laberinto[i][j] = 1  # Pared
    
    laberinto[1][1] = 3  # Pac-Man
    laberinto[FILAS-2][COLUMNAS-2] = 4  # Fantasma 1
    laberinto[FILAS-2][1] = 4  # Fantasma 2
    
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if laberinto[i][j] == 0 and random.random() < 0.5:
                laberinto[i][j] = 2  # Coco
    
    return laberinto

laberinto_matriz = generar_laberinto()
puntaje = 0

def encontrar_posiciones():
    global posicion_jugador, posiciones_fantasmas, contenido_fantasmas
    posiciones_fantasmas = []
    contenido_fantasmas = {}
    for i in range(FILAS):
        for j in range(COLUMNAS):
            if laberinto_matriz[i][j] == 3:
                posicion_jugador = (i, j)
            elif laberinto_matriz[i][j] == 4:
                posiciones_fantasmas.append((i, j))
                contenido_fantasmas[(i, j)] = 0  # Guardamos el contenido original de la celda

encontrar_posiciones()

def dibujar_laberinto():
    pantalla.fill(NEGRO)
    for i in range(FILAS):
        for j in range(COLUMNAS):
            x, y = j * TAMANO_CELDA, i * TAMANO_CELDA
            if laberinto_matriz[i][j] == 1:
                pygame.draw.rect(pantalla, BLANCO, (x, y, TAMANO_CELDA, TAMANO_CELDA))
            elif laberinto_matriz[i][j] == 3:
                pygame.draw.circle(pantalla, AMARILLO, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 2)
            elif laberinto_matriz[i][j] == 4:
                pygame.draw.circle(pantalla, ROJO, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 2)
            elif laberinto_matriz[i][j] == 2:
                pygame.draw.circle(pantalla, AZUL, (x + TAMANO_CELDA // 2, y + TAMANO_CELDA // 2), TAMANO_CELDA // 6)
    
    texto_puntaje = fuente.render(f'Puntaje: {puntaje}', True, BLANCO)
    pantalla.blit(texto_puntaje, (10, FILAS * TAMANO_CELDA + 10))
    pygame.display.flip()

def mover_jugador(dx, dy):
    global posicion_jugador, puntaje
    x, y = posicion_jugador
    nueva_x, nueva_y = x + dx, y + dy
    
    if 0 <= nueva_x < FILAS and 0 <= nueva_y < COLUMNAS:
        if laberinto_matriz[nueva_x][nueva_y] != 1:
            if laberinto_matriz[nueva_x][nueva_y] == 2:
                puntaje += 10
                laberinto_matriz[nueva_x][nueva_y] = 0
            elif laberinto_matriz[nueva_x][nueva_y] == 4:
                game_over()
                return
            
            laberinto_matriz[x][y] = 0
            laberinto_matriz[nueva_x][nueva_y] = 3
            posicion_jugador = (nueva_x, nueva_y)
            verificar_victoria()

def mover_fantasmas():
    global posiciones_fantasmas, contenido_fantasmas
    nuevas_posiciones = {}
    
    for x, y in posiciones_fantasmas:
        opciones = [(x-1, y), (x+1, y), (x, y-1), (x, y+1)]
        random.shuffle(opciones)
        
        for nueva_x, nueva_y in opciones:
            if 0 <= nueva_x < FILAS and 0 <= nueva_y < COLUMNAS and laberinto_matriz[nueva_x][nueva_y] not in [1, 4]:
                if laberinto_matriz[nueva_x][nueva_y] == 3:
                    game_over()
                
                # Restaurar la celda anterior del fantasma con su contenido original
                laberinto_matriz[x][y] = contenido_fantasmas[(x, y)]
                
                # Guardar el contenido de la nueva celda antes de mover el fantasma
                contenido_fantasmas[(nueva_x, nueva_y)] = laberinto_matriz[nueva_x][nueva_y]
                laberinto_matriz[nueva_x][nueva_y] = 4
                nuevas_posiciones[(nueva_x, nueva_y)] = contenido_fantasmas[(nueva_x, nueva_y)]
                break
        else:
            nuevas_posiciones[(x, y)] = contenido_fantasmas[(x, y)]
    
    posiciones_fantasmas = list(nuevas_posiciones.keys())
    contenido_fantasmas = nuevas_posiciones

def verificar_victoria():
    if all(2 not in fila for fila in laberinto_matriz):
        victoria()

def game_over():
    Tk().withdraw()
    if messagebox.askyesno("Game Over", "¿Quieres jugar de nuevo?"):
        reiniciar_juego()
    else:
        pygame.quit()
        exit()

def victoria():
    Tk().withdraw()
    if messagebox.askyesno("¡Ganaste!", "¿Jugar de nuevo?"):
        reiniciar_juego()
    else:
        pygame.quit()
        exit()

def reiniciar_juego():
    global laberinto_matriz, puntaje
    puntaje = 0
    laberinto_matriz = generar_laberinto()
    encontrar_posiciones()

clock = pygame.time.Clock()
tick_fantasmas = 0
ejecutando = True
while ejecutando:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            ejecutando = False
        elif evento.type == pygame.KEYDOWN:
            if evento.key == pygame.K_w:
                mover_jugador(-1, 0)
            elif evento.key == pygame.K_s:
                mover_jugador(1, 0)
            elif evento.key == pygame.K_a:
                mover_jugador(0, -1)
            elif evento.key == pygame.K_d:
                mover_jugador(0, 1)
    
    if tick_fantasmas % 5 == 0:  # Los fantasmas se mueven cada 5 ciclos
        mover_fantasmas()
    
    dibujar_laberinto()
    tick_fantasmas += 1
    clock.tick(10)  # Control de velocidad del juego

pygame.quit(
