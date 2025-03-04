# proyectyo-programacion
proyecto programacion
import random
import os

FICHAS_POR_JUGADOR = 4  # Número de fichas por jugador

class Dados:
    def __init__(self):
        self.num_dados = 2  # Se lanzan dos dados
        self.pares_consecutivos = 0  # Contador de pares consecutivos

    def lanzar_dados(self):
        # Lanzamos los dos dados
        dado1 = random.randint(1, 6)
        dado2 = random.randint(1, 6)
        return dado1, dado2

    def es_par(self, dado1, dado2):
        # Verificamos si los dados son iguales (par)
        return dado1 == dado2

    def manejar_turno(self, posicion_actual, fichas_en_carcel, posiciones_jugadores, current_player):
        """
        Lanza los dados y aplica las reglas:
         - Doble 6 → libera ficha de la cárcel (si hay).
         - Tres pares consecutivos → la ficha vuelve a la cárcel y se termina el turno.
         - Si la ficha cae en la misma casilla que la ficha de otro jugador, esa ficha rival se va a la cárcel.
        """
        dado1, dado2 = self.lanzar_dados()
        print(f"Resultado del lanzamiento: {dado1}, {dado2}")

        # Si saca un par de 6 y tiene fichas en la cárcel, libera una ficha
        if dado1 == 6 and dado2 == 6 and fichas_en_carcel > 0:
            print("¡Doble 6! Liberas una ficha de la cárcel.")
            fichas_en_carcel -= 1

        # Verificamos si es par
        if self.es_par(dado1, dado2):
            self.pares_consecutivos += 1
            print("¡Dados iguales! Puedes lanzar de nuevo.")
            # Tres pares consecutivos: la ficha vuelve a la cárcel y el turno termina
            if self.pares_consecutivos == 3:
                print("¡Tres pares seguidos! Tu ficha vuelve a la cárcel y tu turno termina.")
                fichas_en_carcel += 1
                self.pares_consecutivos = 0
                return posicion_actual, fichas_en_carcel
        else:
            self.pares_consecutivos = 0

        # Movimiento en el tablero (68 casillas en total)
        nueva_posicion = (posicion_actual + dado1 + dado2) % 68

        # Verificar colisión: si la ficha cae en la misma casilla que una ficha de otro jugador,
        # se envía la ficha rival a la cárcel (casilla 10)
        for jugador, piezas in posiciones_jugadores.items():
            if jugador != current_player:
                for idx, pos in enumerate(piezas):
                    if pos == nueva_posicion:
                        print(f"¡El jugador {current_player} cayó en la casilla de una ficha del jugador {jugador}!")
                        print(f"La ficha del jugador {jugador} va a la cárcel.")
                        posiciones_jugadores[jugador][idx] = 10  # Casilla 10 = cárcel

        return nueva_posicion, fichas_en_carcel

class Tablero:
    def __init__(self, num_jugadores=4):
        self.num_jugadores = num_jugadores
        # Cada jugador tiene una lista con la posición de cada una de sus fichas, inicializadas en 0
        self.posiciones = {jugador: [0] * FICHAS_POR_JUGADOR for jugador in range(num_jugadores)}
        # Otras estructuras (internas, salida, seguro, llegada) se mantienen como en tu ejemplo
        self.casillas_internas = {jugador: {i: [] for i in range(1, 9)} for jugador in range(1, num_jugadores + 1)}
        self.casillas_salida = {jugador: [] for jugador in range(1, num_jugadores + 1)}
        self.casillas_seguro = {jugador: [] for jugador in range(1, num_jugadores + 1)}
        self.casillas_llegada = {jugador: [] for jugador in range(1, num_jugadores + 1)}

    def agregar_ficha(self, jugador, ficha, casilla):
        """
        Agrega una ficha a las estructuras del tablero (este ejemplo es solo ilustrativo).
        """
        if casilla in self.casillas_internas[jugador]:
            self.casillas_internas[jugador][casilla].append((jugador, ficha))
        elif casilla == 'salida':
            self.casillas_salida[jugador].append((jugador, ficha))
        elif casilla == 'seguro':
            self.casillas_seguro[jugador].append((jugador, ficha))
        elif casilla == 'llegada':
            self.casillas_llegada[jugador].append((jugador, ficha))
        else:
            print("Casilla no válida o no contemplada en este ejemplo.")

    def mover_ficha(self, jugador, ficha, desde, hacia):
        # Aquí programarías cómo mover la ficha de 'desde' a 'hacia'.
        pass

    def mostrar_tablero(self):
        """
        Muestra en consola la posición de cada ficha de cada jugador.
        """
        print("\n=== Tablero Actual ===")
        for j in range(self.num_jugadores):
            print(f"Jugador {j}: {self.posiciones[j]}")
        print("======================")

def mostrar_tablero_ascii(posiciones_jugadores, total_casillas=68):
    """
    Dibuja en una línea el tablero representado con puntos.
    Cada casilla vacía se muestra como '.' y si en una casilla hay una o más fichas se muestra el/los identificador(es) del jugador.
    """
    board = ['.' for _ in range(total_casillas)]
    for jugador, piezas in posiciones_jugadores.items():
        for pos in piezas:
            # Si ya hay algo en la casilla, se concatenan los identificadores
            if board[pos] == '.':
                board[pos] = str(jugador)
            else:
                board[pos] += str(jugador)
    print("Tablero ASCII:")
    print("".join(board))

def clear_console():
    os.system('cls' if os.name == 'nt' else 'clear')

def main():
    num_jugadores = 4  # Ahora hay 4 jugadores
    tablero = Tablero(num_jugadores)
    dados = Dados()

    # Inicializamos las posiciones de cada ficha para cada jugador
    posiciones_jugadores = {j: [0] * FICHAS_POR_JUGADOR for j in range(num_jugadores)}
    # Se mantiene un contador de fichas en cárcel para cada jugador (como número entero)
    fichas_en_carcel = {j: 0 for j in range(num_jugadores)}

    turno_actual = 0
    while True:
        clear_console()
        print(f"\n--- Turno del Jugador {turno_actual} ---")
        print("Tus fichas:")
        for idx, pos in enumerate(posiciones_jugadores[turno_actual]):
            print(f"  Ficha {idx}: casilla {pos}")

        try:
            ficha_index = int(input("¿Qué ficha deseas mover? (0-3): "))
            if ficha_index < 0 or ficha_index >= FICHAS_POR_JUGADOR:
                print("Índice de ficha no válido.")
                input("Presiona Enter para continuar...")
                continue
        except ValueError:
            print("Entrada no válida. Intenta de nuevo.")
            input("Presiona Enter para continuar...")
            continue

        pos_actual = posiciones_jugadores[turno_actual][ficha_index]
        en_carcel = fichas_en_carcel[turno_actual]

        # Ejecuta el turno para la ficha seleccionada, pasando además el id del jugador actual
        nueva_pos, en_carcel = dados.manejar_turno(pos_actual, en_carcel, posiciones_jugadores, turno_actual)
        posiciones_jugadores[turno_actual][ficha_index] = nueva_pos
        fichas_en_carcel[turno_actual] = en_carcel
        tablero.posiciones[turno_actual] = posiciones_jugadores[turno_actual]

        tablero.mostrar_tablero()
        mostrar_tablero_ascii(posiciones_jugadores, total_casillas=68)

        cont = input("\n¿Continuar? (s/n): ")
        if cont.lower() != 's':
            break

        turno_actual = (turno_actual + 1) % num_jugadores

if __name__ == "__main__":
    main()
