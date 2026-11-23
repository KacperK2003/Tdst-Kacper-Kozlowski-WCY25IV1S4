# Sprawozdanie — Laboratorium 2

**Link do repozytorium:** [https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4](https://github.com/KacperK2003/Tdst-Kacper-Kozlowski-WCY25IV1S4)

**Antigravity · Gemini 4 Argon · Low**

---

### 1. Cel i zakres

Celem jest zaprojektowanie prototypu filtru antyspamowego. Prototyp ten ma otrzymywać treści wiadomości e-mail i na ich podstawie klasyfikować je do jednej z dwóch klas: `ham` i `spam`. Prototyp po przeanalizowaniu wiadomości szacuje prawdopodobieństwo tego, że wiadomość jest spamem. Jeśli prawdopodobieństwo przekroczy ustalony próg decyzyjny to wiadomość jest klasyfikowana jako spam i zwracana jest etykieta `spam`.  W przeciwnym wypadku zwracana jest etykieta `ham`. Zakresem projektu jest prototyp działający lokalnie, a uczony ma być wyłącznie na danych zawartych w zbiorze `Apache SpamAssassin` z 2003 roku.

---

### 2. Wybrane podejście i uzasadnienie

Dane są podzielone na zestaw treningowy (`data/train.csv` - 120 wiadomości ham oraz 60 spam) oraz walidacyjny (`data/validation.csv` - 40 ham i 20 spam). Każda wiadomość z tych zestawów ma identyfikator (`message_id`), treść (`text`) oraz etykietę (`label`). Dane pochodzą ze zbioru `Apache SpamAssassin`. W zbiorze dane nie zawierają duplikatów. Do modelu trafia tylko treść wiadomości, w której wszystkie duże litery są zamieniane na małe (po to, aby słowa były rozpoznawane tak samo niezależnie od tego, czy były napisane małymi, czy dużymi literami) oraz poddawana jest procesowi `tokenizacji` oraz podziałowi na `2-gramy znakowe`. Tokenizacja pozwoli na łatwy podział tekstu na pojedyncze słowa. 2-gramy pozwolą zachować informacje o znakach wskazujących na próbę wymuszenia jakichś emocji (np. duża ilość wykrzykników pod rząd). Po tym każdemu tokenowi i 2-gramowi można przydzielić wartość liczbową (`TF-IDF`), po czym są przekazywane do modelu w postaci liczb. Sam identyfikator wiadomości nie jest potrzebny do klasyfikacji. Etykieta służy do uczenia i sprawdzania podczas walidacji, natomiast podczas użycia modelu etykietę model ma sam podać. Wadą tej reprezentacji jest to, że tracone są kolejność zdań oraz szerszy kontekst gramatyczny i semantyczny. Reprezentacja ta zakłada też pomijanie słów, które nie pojawiły się w danych trenujących. Model będzie wykorzystywać `regresję logistyczną`, która pozwoli na zwrócenie prawdopodobieństwa tego, że wiadomość jest spamem. Powodem wybrania tej metody jest prostota implementacji, dobra współpraca z wybraną reprezentacją danych, odporność na przeuczenie oraz wysoka kontrola błędów dzięki łatwemu do ustawienia progowi decyzyjnemu.

Porównanie z alternatywą:

|Kryterium|Wybrany model: Regresja Logistyczna (z regularyzacją L2)|Alternatywa: Wielomianowy Naiwny Klasyfikator Bayesa|
|-|-|-|
|Współpraca z wybraną reprezentacją (tokeny + 2-gramy znakowe)|Bardzo dobra. Uwzględnia współwystępowanie cech. Nie zakłada, że token `"free"` i 2-gram `"ee"` lub `"!!"` są od siebie niezależne.|Ograniczona przez „naiwne” założenie niezależności. Ponieważ 2-gramy znakowe są fragmentami tokenów słownych, cechy są ze sobą silnie powiązane — Naive Bayes sztucznie wielokrotnie mnoży ten sam sygnał.|
|Zachowanie na małym zbiorze (`180` wiadomości w `train.csv`)|Odporna na przeuczenie dzięki regularyzacji L2, która ogranicza wpływ rzadkich słów występujących np. tylko w jednym e-mailu.|Bardzo szybka i stabilna na małych próbach, wymaga jedynie wygładzania (np. Laplace'a), aby słowa niewidziane w danej klasie nie zerowały prawdopodobieństwa.|
|Kontrola błędów (próg decyzyjny)|Wysoka. Zwraca wiarygodne prawdopodobieństwa (0–1), co ułatwia dostrojenie progu tak, by chronić wiadomości `ham` przed błędnym uznaniem za `spam`.|Słabsza. Zwracane wartości są skrajne (bliskie 0 lub 1 przez mnożenie wielu zależnych cech), co utrudnia intuicyjne sterowanie progiem.|
|Interpretowalność i wymagania sprzętowe|Bardzo wysoka i lekka. Można wprost odczytać listę tokenów i 2-gramów z najwyższymi wagami. Trenuje się w ułamku sekundy lokalnie na CPU.|Bardzo wysoka i jeszcze lżejsza. Uczenie polega na prostym zliczaniu częstości w tabeli; również działa natychmiastowo lokalnie.|

---

### 3. Kroki planu i ich zależności

| Krok | Czynność | Potrzebne dane (wejście) | Co powstaje i co trzeba zachować (wyjście) | Do czego jest potrzebne dalej? |
| --- | --- | --- | --- | --- |
| 1. Wczytanie i rozdzielenie danych | Odczyt plików CSV z zachowaniem oryginalnego podziału; oddzielenie treści `text` od etykiet `label` i odłożenie `message_id`. | Pliki `data/train.csv` (180 wierszy) oraz `data/validation.csv` (60 wierszy). | Osobne zestawy: teksty treningowe, etykiety treningowe, teksty walidacyjne oraz odłożone etykiety walidacyjne i `message_id`. | Teksty treningowe trafiają do Kroku 2, etykiety treningowe do Kroku 3, teksty walidacyjne do Kroku 4, a etykiety walidacyjne do Kroku 5. |
| 2. Budowa słownika i wektoryzacja (`train.csv`) | Zamiana liter na małe, podział na tokeny słowne i 2-gramy znakowe, budowa słownika i wyliczenie wag `TF-IDF`. | Wyłącznie teksty treningowe (`text` z `train.csv` z Kroku 1). | 1) Dopasowany przekształtnik tekstu / słownik cech z wagami IDF (do zachowania).<br>2) Macierz cech treningowych (wektory liczb). | Macierz cech treningowych jest potrzebna w Kroku 3 do uczenia modelu, a zachowany słownik w Kroku 4 (i przy nowych wiadomościach) do takiego samego przekształcenia tekstu. |
| 3. Uczenie modelu | Trenowanie regresji logistycznej (z regularyzacją L2) dopasowującej wagi cech do klas `spam` i `ham`. | Macierz cech treningowych (z Kroku 2) oraz prawdziwe etykiety treningowe (`label` z Kroku 1). | Wytrenowany model (wyuczone wagi cech, wyraz wolny i próg decyzyjny, np. 0.5 — do zachowania). | Potrzebny w Kroku 4 do wyliczenia przewidywań na zbiorze walidacyjnym oraz później do klasyfikowania nowych wiadomości bez ponownego uczenia. |
| 4. Klasyfikacja zbioru walidacyjnego | Przekształcenie tekstów walidacyjnych zamrożonym słownikiem na wektory `TF-IDF` i wyliczenie przez model prawdopodobieństwa oraz klasy (`spam`/`ham`). | Teksty walidacyjne (`text` z `validation.csv` z Kroku 1), zachowany słownik (z Kroku 2) i zachowany model (z Kroku 3). (Bez etykiet walidacyjnych!). | Lista przewidywanych etykiet i prawdopodobieństw dla 60 wiadomości walidacyjnych powiązana z `message_id` (do zachowania). | Potrzebna w Kroku 5 do porównania z prawdziwymi etykietami i analizy pomyłek modelu. |
| 5. Ocena odpowiedzi modelu | Porównanie przewidzianych klas z prawdziwymi etykietami walidacyjnymi i wyliczenie miar jakości. | Przewidziane etykiety (z Kroku 4) oraz prawdziwe etykiety walidacyjne (`label` z `validation.csv` z Kroku 1). | Macierz pomyłek (TP, TN, FP, FN) oraz miary Precyzja, Czułość i F1-score (do zachowania). | Pozwalają ocenić jakość prototypu (przeoczony spam i fałszywe alarmy) oraz zdecydować o ewentualnej korekcie planu (np. zmianie progu decyzyjnego). |

---

### 4. Uczenie i późniejsze użycie

Model jest uczony na podstawie danych z pliku `data/train.csv`. Zawarte w nim treści wiadomości są przekazywane do uczącego się modelu po wcześniejszym przygotowaniu. Do modelu podawana jest też etykieta, która jest przypisana do wiadomości, po to, aby model mógł się nauczyć, jak klasyfikować wiadomości. Różni się to od prawdziwej klasyfikacji z już nauczonym modelem tym, że podczas uczenia, etykieta jest znana, a podczas prawdziwego użycia to model ją próbuje przewidzieć. Podczas uczenia powstaje model (w tym wypadku model regresji logistycznej), a konkretnie jego parametry (wagi), dla których model ten potrafi poddać wiadomość klasyfikacji. Parametry te należy zachować, aby potem móc je podać do wdrożenia. W przypadku tego projektu, podczas uczenia powstaje również słownik cech, który przypisuje tokenom i 2-gramom, które wystąpiły w danych treningowych, wartości liczbowe. Słownik ten też należy zachować, aby móc go użyć do wdrożenia modelu.

---

### 5. Planowana ocena

Danymi do oceny są dane z pliku `data/validation.csv`. Znajdują się w nim wiadomości z identyfikatorem, treścią i poprawną etykietą. Treść wiadomości jest przekazywana (po wcześniejszym przygotowaniu) do już wytrenowanego modelu a on zwraca prawdopodobieństwo, które jeśli przekroczy ustalony próg jest wykorzystywane do określenia przewidywanej przez model etykiety. Jeśli przewidywana etykieta zgadza się z etykietą przypisaną do wiadomości w zbiorze walidacyjnym, model jest oceniany wyżej. Miarami wykorzystywanymi do oceny całego modelu są:
- macierz pomyłek, w której są zapisane przypadki: poprawnie zakwalifikowane do spamu (True Positive), niepoprawnie zakwalifikowane do spamu (False Positive), poprawnie niezakwalifikowane do spamu (True Negative) oraz niepoprawnie niezakwalifikowane do spamu (False Negative).
- precyzja, którą oblicza się używając wzoru: $\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}}$, pokazuje ona jaki procent wiadomości zakwalifikowanych do spamu przez model faktycznie jest spamem. Duża liczba oznacza, że pojawiło się mało wyników False Positive, dzięki czemu jest małe prawdopodobieństwo na utratę przez użytkownika ważnej wiadomości.
- czułość, którą oblicza się ze wzoru $\text{Recall} = \frac{\text{TP}}{\text{TP} + \text{FN}}$, pokazuje ona jaki procent spośród wszystkich rzeczywistych wiadomości typu spam model zdołał wyłapać. Duża czułość oznacza, że model nie przepuszcza zbyt dużej ilości spamu do skrzynki odbiorczej użytkownika.
- miara F1, którą oblicza się ze wzoru $\text{F1} = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$, pokazuje ona średnią harmoniczną precyzji i czułości. Jest ona wysoka tylko wtedy, gdy model jednocześnie wyłapuje dużą ilość prawdziwego spamu oraz nie oznacza błędnie zwykłych wiadomości jako spam.

---

### 6. Niewiadome i dalsze sprawdzenia

| Niewiadoma (założenie do sprawdzenia) | Jak to sprawdzić? | Jaki wynik skłoni do zmiany podejścia i co wtedy zmienimy? |
| --- | --- | --- |
| 1. Czy połączenie tokenów słownych z 2-gramami znakowymi nie tworzy zbyt wielu szumnych cech przy małym zbiorze (180 wiadomości)? | Porównać na zbiorze walidacyjnym (`validation.csv`) wynik modelu uczonego na pełnym słowniku tokenów + 2-gramów znakowych z modelem o ograniczonym słowniku (np. odrzucającym cechy występujące tylko w 1 wiadomości — `min_df=2` — oraz z samym TF-IDF słownym). | Warunek zmiany: Jeśli pełny słownik tokenów + 2-gramów znakowych pogarsza wynik F1 / Precyzję na `validation.csv` względem samych tokenów słownych lub powoduje przeuczenie (idealny wynik na `train.csv`, słaby na `validation.csv`).<br>Korekta planu: Odfiltrować najrzadsze 2-gramy znakowe (np. zostawić tylko cechy występujące w co najmniej 2–3 wiadomościach treningowych), zwiększyć siłę regularyzacji L2 lub zastąpić pełne 2-gramy znakowe kilkoma bezpośrednimi licznikami znaków specjalnych (`!`, `$`, `#`, wielkie litery). |
| 2. Czy zamiana całego tekstu na małe litery (lowercasing) nie usuwa cennej informacji o słowach pisanych `WIELKIMI LITERAMI`? | Sprawdzić, czy rozróżnianie wielkości liter w 2-gramach znakowych (lub dodanie jednej cechy: odsetek wielkich liter w wiadomości przed lowercasingiem) zmniejsza liczbę przeoczonych spamów (FN) na `validation.csv`. | Warunek zmiany: Jeśli wiadomości `spam` pisane wielkimi literami są przeoczane po zamianie na małe litery.<br>Korekta planu: Zachować oryginalną wielkość liter przy wycinaniu 2-gramów znakowych (lub dodać osobną cechę mierzącą udział wielkich liter przed lowercasingiem tokenów słownych). |
| 3. Czy domyślny próg decyzyjny regresji logistycznej (0.5) zapewnia wystarczającą ochronę prawidłowej poczty (`ham`) przed wrzuceniem do spamu (FP)? | Po wytrenowaniu modelu wygenerować macierz pomyłek na `validation.csv` dla progu 0.5 i sprawdzić liczbę fałszywych alarmów (FP — `ham` oznaczony jako `spam`) oraz przeoczonych spamów (FN). | Warunek zmiany: Jeśli przy progu 0.5 liczba błędów FP jest zbyt wysoka (niska Precyzja — prawidłowe wiadomości trafiają do spamu) albo przeciwnie — model jest zbyt ostrożny i przepuszcza większość spamu (wysokie FN, niska Czułość).<br>Korekta planu: Przesunąć próg decyzyjny prawdopodobieństwa (np. podnieść do 0.6-0.7, aby zredukować FP kosztem FN, lub dostosować wagi klas podczas uczenia regresji logistycznej). |
| 4. Czy powtarzające się wątki między `train.csv` a `validation.csv` (5 wspólnych tematów) sztucznie nie zawyżają oceny modelu? | Podczas analizy wyników na `validation.csv` sprawdzić osobno skuteczność modelu na wiadomościach należących do 5 wspólnych wątków oraz na pozostałych wiadomościach walidacyjnych, a także przejrzeć cechy o najwyższych wagach w modelu. | Warunek zmiany: Jeśli wśród najważniejszych cech modelu dominują nazwy specyficznych list dyskusyjnych lub tematów (np. `spambayes`, `secure`, `sofware`), a poza wspólnymi wątkami skuteczność wyraźnie spada.<br>Korekta planu: Uwzględnić to w ocenie końcowej (raportować wynik z wyłączeniem wspólnych wątków lub usunąć z tekstu techniczne prefiksy list dyskusyjnych w nawiasach kwadratowych typu `[Spambayes]`, aby model uczył się ogólnych cech poczty, a nie konkretnych nazw wątków). |

