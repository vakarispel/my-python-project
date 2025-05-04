import random
import json
import unittest
from abc import ABC, abstractmethod

class GameManager:
    _instance = None

    def __new__(cls):
        if not cls._instance:
            cls._instance = super(GameManager, cls).__new__(cls)
        return cls._instance

    def __init__(self):
        if not hasattr(self, 'history'):
            self.history = []

    def log_action(self, action):
        self.history.append(action)

    def save_history_to_file(self, filename="game_history.json"):
        with open(filename, "w") as file:
            json.dump(self.history, file, indent=2)

    def export_history_txt(self, filename="game_history.txt"):
        with open(filename, "w") as file:
            for entry in self.history:
                file.write(entry + "\n")

    def load_history_from_file(self, filename="game_history.json"):
        try:
            with open(filename, "r") as file:
                self.history = json.load(file)
        except FileNotFoundError:
            self.history = []

class Ship:
    def __init__(self, name, size):
        self.name = name
        self.size = size
        self.positions = []
        self.hits = 0

    def is_sunk(self):
        return self.hits >= self.size

class Board:
    def __init__(self, size=5):
        self.size = size
        self.grid = [["." for _ in range(size)] for _ in range(size)]
        self.ships = []

    def place_ship(self, ship, start, orientation):
        x, y = start
        positions = []

        for i in range(ship.size):
            if orientation == "H":
                if y + i >= self.size or self.grid[x][y + i] != ".":
                    return False
                positions.append((x, y + i))
            elif orientation == "V":
                if x + i >= self.size or self.grid[x + i][y] != ".":
                    return False
                positions.append((x + i, y))

        for pos in positions:
            self.grid[pos[0]][pos[1]] = "S"

        ship.positions = positions
        self.ships.append(ship)
        return True

    def receive_attack(self, position):
        x, y = position
        if self.grid[x][y] == "S":
            self.grid[x][y] = "X"
            for ship in self.ships:
                if position in ship.positions:
                    ship.hits += 1
                    GameManager().log_action(f"Attack at ({x+1},{y+1}): Hit on {ship.name}")
                    return "Hit!" if not ship.is_sunk() else f"You sank the {ship.name}!"
        elif self.grid[x][y] == ".":
            self.grid[x][y] = "O"
            GameManager().log_action(f"Attack at ({x+1},{y+1}): Miss")
            return "Miss."
        elif self.grid[x][y] in ["X", "O"]:
            GameManager().log_action(f"Attack at ({x+1},{y+1}): Already attacked")
            return "Already attacked here."
        return "Invalid attack."

    def get_display_lines(self, reveal_ships=False):
        lines = ["   " + " ".join(str(i+1) for i in range(self.size))]
        for idx, row in enumerate(self.grid):
            display_row = [cell if reveal_ships or cell != "S" else "." for cell in row]
            lines.append(f"{idx+1:2} {' '.join(display_row)}")
        return lines

class Player(ABC):
    def __init__(self, name):
        self.name = name
        self.board = Board()

    def auto_place_ships(self, ships):
        for ship in ships:
            placed = False
            attempts = 0
            while not placed and attempts < 100:
                start = (random.randint(0, self.board.size - 1), random.randint(0, self.board.size - 1))
                orientation = random.choice(["H", "V"])
                placed = self.board.place_ship(Ship(ship.name, ship.size), start, orientation)
                attempts += 1

    @abstractmethod
    def take_turn(self, opponent_board):
        pass

class HumanPlayer(Player):
    def take_turn(self, opponent_board):
        while True:
            try:
                coords = input(f"{self.name}, enter attack coordinates (row col) [1-{opponent_board.size}]: ")
                x, y = map(int, coords.strip().split())
                x -= 1
                y -= 1
                if 0 <= x < opponent_board.size and 0 <= y < opponent_board.size:
                    result = opponent_board.receive_attack((x, y))
                    if result != "Already attacked here.":
                        GameManager().log_action(f"{self.name} attacks ({x+1},{y+1}): {result}")
                        print(f"{self.name} attacks ({x+1},{y+1}): {result}")
                        return
                    else:
                        print(result)
                else:
                    print("Coordinates out of bounds.")
            except ValueError:
                print("Invalid input. Please enter two integers separated by space.")

class AIPlayer(Player):
    def take_turn(self, opponent_board):
        while True:
            x = random.randint(0, opponent_board.size - 1)
            y = random.randint(0, opponent_board.size - 1)
            result = opponent_board.receive_attack((x, y))
            if result != "Already attacked here.":
                GameManager().log_action(f"{self.name} attacks ({x+1},{y+1}): {result}")
                print(f"{self.name} attacks ({x+1},{y+1}): {result}")
                return

class Game:
    def __init__(self):
        self.players = []

    def setup(self):
        player1 = HumanPlayer("Player")
        player2 = AIPlayer("Computer")
        self.players = [player1, player2]

        ships = [Ship("Destroyer", 2), Ship("Submarine", 3), Ship("Battleship", 4)]
        for player in self.players:
            player.auto_place_ships(ships)

    def display_boards(self):
        p1_lines = self.players[0].board.get_display_lines(reveal_ships=True)
        p2_lines = self.players[1].board.get_display_lines(reveal_ships=False)
        print("\nYour Board".ljust(20) + "Enemy Board")
        for l1, l2 in zip(p1_lines, p2_lines):
            print(f"{l1.ljust(20)} {l2}")

    def play(self):
        current = 0
        while True:
            opponent = (current + 1) % 2
            print(f"\n{self.players[current].name}'s turn!")
            self.display_boards()
            self.players[current].take_turn(self.players[opponent].board)

            if all(ship.is_sunk() for ship in self.players[opponent].board.ships):
                winner = self.players[current].name
                print(f"\n {winner} wins!")
                GameManager().log_action(f"{winner} wins the game!")
                GameManager().save_history_to_file()
                GameManager().export_history_txt()
                return
            current = opponent

if __name__ == "__main__":
    print("Welcome to Battleship!")
    game = Game()
    game.setup()
    game.play()

class TestBattleship(unittest.TestCase):
    def setUp(self):
        self.board = Board()
        self.ship = Ship("TestShip", 2)
        self.board.place_ship(self.ship, (0, 0), "H")

    def test_place_ship_success(self):
        new_board = Board()
        ship = Ship("Sub", 3)
        self.assertTrue(new_board.place_ship(ship, (1, 1), "V"))

    def test_attack_hit_and_sunk(self):
        result1 = self.board.receive_attack((0, 0))
        result2 = self.board.receive_attack((0, 1))
        self.assertIn("sank", result2)

    def test_attack_miss(self):
        result = self.board.receive_attack((2, 2))
        self.assertEqual(result, "Miss.")

    def test_repeated_attack(self):
        self.board.receive_attack((2, 2))
        result = self.board.receive_attack((2, 2))
        self.assertEqual(result, "Already attacked here.")

if __name__ == "__main__":
    unittest.main()