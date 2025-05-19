# Battleship žaidimas – OOP Kursinis Darbas

## ***Įvadas:***

***Kas tai per programa?***

Tai yra "Jūrų mūšio" žaidimo (Battleship) versija, sukurta Python kalba, taikant objektinio programavimo (OOP) principus. Žaidimas vyksta paeiliui, laivai išdėstomi automatiškai, kiekvienas veiksmas yra užregistruojamas, o rezultatai išsaugomi faile.

***Kaip paleisti programą?***

Atidarykite terminalą arba komandų eilutę.

Paleisti žaidimą su komanda:

    python .\battleship-game.py
Paleisti testus su komanda:

    python -m unittest .\battleship-game.py

***Kaip naudotis programa?***

Žaidimas automatiškai išdėsto laivus kiekvienam žaidėjui.
Kiekvieno ėjimo metu prašoma įvesti taikinio koordinates (pvz., 2 3).
Ekrane rodoma, ar tai buvo pataikymas, nepataikymas, ar nuskandintas laivas.
Žaidimas baigiasi, kai vieno žaidėjo visi laivai yra sunaikinti.

## ***4 OOP principai***

***Enkapsuliacija (Encapsulation)***

Enkapsuliacija reiškia, kad duomenys ir funkcijos, kurios juos valdo, yra sudėti į vieną vienetą – klasę. Vidinė būsena yra paslėpta nuo išorės, o su ja dirbti galima tik per nurodytas funkcijas.
Čia klasė Ship valdo savo kintamuosius, o kiti objektai gali tik klausti, ar laivas nuskendo – tiesiogiai keisti „hits“ negalima.

    class Ship:
        def __init__(self, name, size):
            self.name = name
            self.size = size
            self.positions = []
            self.hits = 0

        def is_sunk(self):
            return self.hits >= self.size
        
***Abstrakcija (Abstraction)***

Abstrakcija slepia sudėtingą veikimo logiką ir pateikia tik tai, kas svarbiausia naudotojui. Vartotojui ar kitoms klasėms nebūtina žinoti, kaip kažkas veikia — svarbu tik kad veikia.
Čia Player klasė nenurodo, kaip tiksliai vykdomas ėjimas — tai turi įgyvendinti konkretūs žaidėjai (HumanPlayer ar AIPlayer). Taip paslepiamos detalės, bet išlaikomas bendras veikimo principas.

    from abc import ABC, abstractmethod

    class Player(ABC):
        @abstractmethod
        def take_turn(self, opponent_board):
            pass
        
***Paveldėjimas (Inheritance)***

Paveldėjimas leidžia sukurti naujas klases, kurios perima kitos klasės savybes ir funkcijas.
Abi šios klasės paveldi bendrą tėvinę klasę Player ir naudoja jos struktūrą, bet įgyvendina savo unikalią logiką.

    class HumanPlayer(Player):
        def take_turn(self, opponent_board):
            ...

    class AIPlayer(Player):
        def take_turn(self, opponent_board):
            ...
        
***Polimorfizmas (Polymorphism)***

Polimorfizmas reiškia, kad tas pats metodas gali veikti skirtingai, priklausomai nuo to, koks objektas jį naudoja. Tai leidžia naudoti bendrą sąsają skirtingoms klasėms.
Funkcija take_turn() veikia tiek žmogui, tiek dirbtiniam intelektui, bet elgiasi skirtingai. Žaidimo klasė nesirūpina, kuris žaidėjas tai yra – tiesiog kviečia tą patį metodą.

    self.players[current].take_turn(self.players[opponent].board)

## ***Design pattern***

***Singleton Pattern***

Šis šablonas užtikrina, kad egzistuotų tik vienas tam tikros klasės objektas — tai naudinga, kai reikia vieningos žaidimo būsenos ar istorijos saugojimo.

    class GameManager:
        _instance = None

        def __new__(cls):
            if not cls._instance:
                cls._instance = super(GameManager, cls).__new__(cls)
            return cls._instance

## ***Kompozicija ir agregacija (Composition and Aggregation)***

***Kompozicija***. Kai vienas objektas visiškai priklauso kitam — jei sunaikinsi tėvinį objektą, vidinis irgi išnyks.

    class Player:
        def __init__(self, name):
            self.board = Board()	
Žaidėjas turi savo lentą, kuri neegzistuoja be žaidėjo.

***Agregacija***. Tai silpnesnis ryšys — objektai susiję, bet vienas gali egzistuoti ir be kito.

    class Board:
        def __init__(self):
            self.ships = []
Lenta turi laivus, bet laivai galėtų egzistuoti ir be lentos.

## ***Failų skaitymas ir rašymas***
***Rašymas į failą::***

    def save_history_to_file(self, filename="game_history.json"):
        with open(filename, "w") as file:
            json.dump(self.history, file, indent=2)
Visa žaidimo istorija išsaugoma JSON formatu.

***Skaitymas iš failo:***

    def load_history_from_file(self, filename="game_history.json"):
        try:
            with open(filename, "r") as file:
                self.history = json.load(file)
        except FileNotFoundError:
            self.history = []
Jei istorijos failas egzistuoja – jis įkeliamas, kitaip sukuriama tuščia istorija.


## ***Unit Testing***
Testavimas atliekamas naudojant Python modulį unittest.

    class TestBattleship(unittest.TestCase):
        def test_attack_and_sinking(self):
            board = Board()
            ship = Ship("Destroyer", 2)
            board.place_ship(ship, (0, 0), "H")
            result1 = board.receive_attack((0, 0))
            result2 = board.receive_attack((0, 1))
            self.assertIn("sank", result2)
***Tikrinami aspektai:***
Laivų išdėstymas,
Atakos (pataikymas, nepataikymas, pakartotinis ėjimas),

## ***Rezultatai*** 

Žaidimo logika sukurta sėkmingai – du žaidėjai paeiliui atakuoja vienas kitą. Visi veiksmai (pataikymai, nepataikymai, nuskendimai) yra užregistruoti ir išsaugoti JSON formatu. Vieneto testai parodė, kad pagrindinės funkcijos veikia teisingai.


