# Sprawozdanie — Laboratorium 1

**Link do repozytorium:** [https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4](https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4)  
**Link do pliku `exercise-summary.json`:** [https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4/blob/main/lab1/exercise-summary.json](https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4/blob/main/lab1/exercise-summary.json)

**Antigravity · Gemini 4 Argon · Low**

---

### 1. Jaki wynik zlecono, jakie zmiany były dozwolone i o jakie informacje do odbioru poproszono agenta?

W ramach etapu 1 zlecono wykonanie raportu dla danych zawartych w pliku `project/data/exercise.csv`. Zlecono agentowi zapisanie go w pliku `project/results/exercise-summary.json`. Zabroniona była zmiana kodu, danych oraz testów. Nakazano agentowi zachowanie zgodności z plikiem `README.md` dołączonym do projektu. Po wykonaniu raportu nakazano odczytanie jego zawartości. Dodatkowo zapytano o to, czy polecenia wykonywane są bezpośrednio na komputerze, na którym działa agent, czy wykonuje je inne środowisko udostępniane przez aplikację Antigravity.

Po wykonaniu raportu zlecono agentowi wykonanie testu zawartego w pliku `tests/test_report.py`.

W drugim etapie nakazano mu zmianę etykiety jednej z wiadomości. Zabroniono agentowi zmianę innych danych niż jednej etykiety. Następnie poproszono o sprawdzenie, czy stary raport zgadza się z nowymi, przewidywanymi wartościami. Następnie nakazano agentowi zaproponować operacje mające na celu aktualizację raportu wraz z opisem ich celu oraz podać spodziewaną zmianę liczebności. Po zapoznaniu się z propozycją zaakceptowano ją oraz nakazano jej wykonanie. Agent wykonał skrypt jeszcze raz. Nakazano agentowi odczytać nowy raport oraz porównać go z przewidywaniem.

---

### 2. Jak powstał raport? Wskaż wejście, polecenie, narzędzie, środowisko wykonania i wynik procesu. Dodaj link do końcowego raportu i wyjaśnij jego liczebności.

Raport powstał w wyniku uruchomienia skryptu `describe_data.py` przez agenta. Agent dostał polecenie, aby wykonać raport na podstawie danych wejściowych zawartych w pliku `project/data/exercise.csv`. Agent wykonał polecenie PowerShell:

```powershell
py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

Aby wykonać to polecenie agent użył narzędzia `run_command`, które uruchamia lokalny terminal systemu (w tym przypadku PowerShell) i wykonuje w nim komendę. Proces tworzenia raportu wykonuje się zatem lokalnie na systemie użytkownika. Środowisko wykorzystane do wykonania skryptu posiadało zainstalowanego Pythona w wersji 3.14. Agent po wykonaniu się skryptu podał:

* Wynik procesu (kod wyjścia: `0` - sukces)
* Zawartość utworzonego raportu (link: [https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4/blob/main/lab1/exercise-summary.json](https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4/blob/main/lab1/exercise-summary.json))

W raporcie po pierwszym etapie widniało, że:

* Liczba wiadomości `ham` wynosi **5** (tyle jest wiadomości z etykietą `ham`)
* Liczba wiadomości `spam` wynosi **3** (tyle jest wiadomości z etykietą `spam`)
* Łączna liczba wiadomości (`total_messages`) wynosi **8** (było 5 wiadomości `ham` oraz 3 `spam`, zatem 5 + 3 = 8)

Raport został wykonany ponownie w drugim etapie. Przez zmianę etykiety jednej wiadomości z `ham` na `spam` liczby wiadomości `ham` i `spam` się wyrównały (`ham`: 5 - 1 = 4; `spam`: 3 + 1 = 4). Łączna liczba wiadomości się nie zmieniła. Do ponownego wykonania raportu agent użył tej samej komendy, narzędzia, środowiska oraz wynik procesu był taki sam.

---

### 3. Który fragment dokumentacji lub kodu wyjaśniłeś? Podaj plik i miejsce, własne wyjaśnienie oraz mały przypadek: przewidywanie, sposób sprawdzenia i wniosek.

Kazano agentowi wyjaśnić funkcję `describe(input_path)` z pliku `describe_data.py` (linijki od 13 do 44). Funkcja ta przyjmuje argument `input_path`, który jest ścieżką do pliku z danymi wejściowymi. Funkcja najpierw czyta dane ze wskazanego pliku w formie surowych bajtów potrzebnych do obliczenie sumy kontrolnej przy użyciu algorytmu SHA-256 oraz w formie słowników Pythona zawierających `message_id` (identyfikator wiadomości), `text` (zawartość tekstowa wiadomości), `label` (etykieta). Po wczytaniu sprawdzane jest, czy pierwszy wiersz w pliku ma trzy kolumny. Dzięki temu wiadomo, czy wszystkie rekordy z pliku csv mają wszystkie 3 parametry. Jeśli którejś kolumny nie ma lub są one ułożone w złej kolejności to wyrzucany jest wyjątek a program przerywa działanie. Funkcja następnie inicjalizuje liczniki danych typów wiadomości oraz tworzy zbiór (`set`), który ma przechowywać już sprawdzone wiadomości. Funkcja w pętli sprawdzana jest każda wiadomość. Najpierw rekord jest sprawdzany pod kątem braku którejś wartości. Potem sprawdzane jest, czy id danego rekordu nie pojawiło się już w zbiorze sprawdzonych wcześniej wiadomości. Następnie inkrementowany jest odpowiedni licznik dla danej etykiety. Funkcja zwraca cały raport, czyli ścieżkę pliku z danymi wejściowymi, sumę kontrolną, łączną liczbę wiadomości, informację o środowisku Pythona, ścieżkę roboczą.

**Przykład:**

Niech w zbiorze danych znajdują się dwie wiadomości:

* `message_id`: `001`, `text`: `"Test1"`, `label`: `"ham"`
* `message_id`: `001`, `text`: `"Test2"`, `label`: `"ham"`

W tym przykładzie obie wiadomości mają taki sam identyfikator. Po analizie kodu ze skryptu wraz z agentem przewidywano, że kod zgłosi błąd `"Record X: duplicate message_id."`.

**Sposób sprawdzenia** opierał się na przeczytaniu fragmentu kodu oraz na analizie działania kodu wraz z agentem.

**Wniosek:** Skrypt zgłasza błąd o podwojonym rekordzie, jeśli identyfikator wiadomości jest powtórzony.

---

### 4. Jak uruchomiono gotowy test, jaki był wynik i czego dotyczyło sprawdzenie?

Agent uruchomił narzędzie `run_command` oraz użył komendy:

```powershell
py -3 -m unittest discover -s tests -v
```

Komenda została wykonana lokalnie w PowerShellu. Agent poinformował o wyniku procesu równym `0` oraz podał informacje wypisane w PowerShellu o tym, że test został zaliczony:

```text
test_cli_writes_expected_counts (test_report.ReportTest.test_cli_writes_expected_counts)
The CLI saves a report for the intended input with counts 3 / 2 / 1. ... ok

----------------------------------------------------------------------
Ran 1 test in 0.138s

OK
```

Test przekazuje do programu mały zbiór testowy z 3 wiadomościami, z których 2 mają etykietę `ham` a 1 `spam`. Test przy użyciu asercji sprawdza:

* czy proces `describe_data.py` zakończył się bez błędu (czy zwrócił `0`),
* czy plik JSON z raportem został utworzony na dysku,
* czy raport wskazuje poprawny plik wejściowy,
* czy liczba wszystkich wiadomości w raporcie wynosi 3,
* czy etykiety zostały poprawnie policzone (czy są 2 etykiety `ham` i 1 `spam`).

---

### 5. Jaką etykietę zmieniono w etapie 2? Jakiej zmiany liczebności oczekiwałeś, jakie operacje wybrałeś i dlaczego? Porównaj wynik przed zmianą i po nowym wykonaniu.

Pierwszą operacją zleconą agentowi była zmiana etykiety wiadomości z `ham` na `spam`. Agent wykonał zmianę bezpośrednio w pliku z danymi używając narzędzia `edit_file`. Agent zmienił etykietę wiadomości o ID `exercise-001`. Następnie poproszono agenta o sprawdzenie, czy raport zawiera aktualne dane oraz o to, jak powinny się zmienić liczebności etykiet. Agent odpowiedział, że dane się nie zgadzają z nowymi przewidywaniami oraz podał, jakie powinny być nowe wartości:

* Liczba wiadomości `ham` wynosi **4**
* Liczba wiadomości `spam` wynosi **4**
* Łączna liczba wiadomości (`total_messages`) wynosi **8**

Zgadzało się to z moimi przewidywaniami. Poproszono agenta o zaproponowanie operacji potrzebnych do uzyskania i potwierdzenia aktualnego wyniku. Po zapoznaniu się z propozycją zaakceptowano ją i nakazano ją wykonać. Wybrane operacje to:

* Ponowne uruchomienie skryptu na zmodyfikowanych danych po to, aby skrypt mógł wczytać nowe dane oraz je ponownie sprawdzić, przeliczyć, obliczyć nowy skrót SHA-256:

```powershell
py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

* Ponowne odczytanie nowo zapisanego pliku raportu w celu porównania nowego raportu z przewidywaniami.

**Porównanie wyników przed i po zmianie:**

| Pole w raporcie | Raport z Etapu 1 (przed zmianą) | Przewidywanie dla Etapu 2 | Końcowy raport z Etapu 2 (po zmianie) | Zgodność z przewidywaniem |
| :--- | :---: | :---: | :---: | :---: |
| `total_messages` | 8 | 8 | 8 | Zgodne |
| `label_counts.ham` | 5 | 4 | 4 | Zgodne |
| `label_counts.spam` | 3 | 4 | 4 | Zgodne |
