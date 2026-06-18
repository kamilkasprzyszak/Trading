# Specyfikacja Techniczna: Historical Quant AVP - S/D Zones V6 MAX [KDE]

## 1. Przeznaczenie i Przegląd Funkcjonalny

**Historical Quant AVP - S/D Zones V6 MAX [KDE]** to zaawansowany wskaźnik analizy przepływu zleceń (Order Flow) i handlu ilościowego, zaimplementowany w języku Pine Script v6. Skrypt służy do precyzyjnej identyfikacji instytucjonalnych stref popytu i podaży (Supply/Demand Zones) oraz węzłów wysokiego wolumenu (HVN - High Volume Nodes) na rynkach kontraktów terminowych (indeksy, kryptowaluty).

<img width="1261" height="775" alt="image" src="https://github.com/user-attachments/assets/c47f143f-6ad4-4eb6-84b2-9ed2fc3142dd" />


Narzędzie przetwarza surowe dane wolumenowe i cenowe w ujęciu wielookresowym (Historical Anchored Volume Profile), stosując aparat statystyczny do filtrowania szumu rynkowego oraz dynamicznego mapowania płynności rynkowej w czasie rzeczywistym.


### Przewodnik dla Tradera: Fundamenty działania i zastosowanie praktyczne

Ten rozdział jest przeznaczony dla osób, które wykorzystują wskaźnik wyłącznie do handlu podręcznego (discretionary trading) i chcą zrozumieć, jakie informacje płyną z wykresu oraz jak przekuć je na realne decyzje inwestycyjne, bez wchodzenia w szczegóły kodu źródłowego.

### A. Jaka jest główna filozofia tego narzędzia?
Większość klasycznych wskaźników (np. RSI, MACD czy zwykłe średnie kroczące) bazuje wyłącznie na historycznej cenie. Ignorują one najważniejszy czynnik poruszający rynkiem – **wolumen**, czyli informację o tym, ile kapitału zaangażowano na danym poziomie.

**Historical Quant AVP** działa na zasadzie Teorii Aukcji Rynkowej. Jego zadaniem jest znalezienie miejsc na wykresie, gdzie duże instytucje (tzw. "grubas" lub Smart Money) dokonywały masowej wymiany pozycji. Wskaźnik nie rysuje stref tam, gdzie cena po prostu zawróciła, ale tam, **gdzie doszło do rzeczywistego starcia popytu z podażą potwierdzonego ogromnym obrotem**.

---

### B. Co dokładnie widzisz na wykresie? (Interpretacja wizualna)

Wskaźnik automatycznie nanosi na wykres kolorowe bloki (strefy) oraz linie. Oto ich bezpośrednie znaczenie rynkowe:

1. **Złota Strefa / Złota Linia (POC - Point of Control):** * **Co oznacza:** To najważniejszy poziom cenowy w całym badanym okresie (np. z całej sesji dziennej lub tygodniowej). To tutaj handlowano najwięcej kontraktów.
   * **Znaczenie dla Tradera:** Jest to tzw. „cena sprawiedliwa” (Fair Value). Rynek ma naturalną tendencję do powracania do tego poziomu (działa jak magnes).

2. **Czerwone Strefy (SUPPLY - Podaż):**
   * **Co oznacza:** Są to klastry wysokiego wolumenu, które znajdują się **powyżej** obecnej ceny rynkowej, a algorytm wykrył w nich dominację zleceń sprzedających (negatywna Delta).
   * **Znaczenie dla Tradera:** Silny opór. To miejsca, gdzie instytucje dystrybuowały (sprzedawały) swoje aktywa. Spodziewaj się tam reakcji spadkowej.

3. **Morskie / Tealowe Strefy (DEMAND - Popyt):**
   * **Co oznacza:** Klastry wysokiego wolumenu zlokalizowane **poniżej** obecnej ceny rynkowej, gdzie przewagę zyskały zlecenia kupujących (pozytywna Delta).
   * **Znaczenie dla Tradera:** Silne wsparcie. Miejsca instytucjonalnej akumulacji (kupna). Cena w tych rejonach ma statystyczną skłonność do odbijania w górę.

4. **Pomarańczowa, schodkowa linia (dPOC - Developing POC):**
   * **Co oznacza:** Pokazuje, jak w miarę upływu dnia przemieszczał się punkt największego obrotu.
   * **Znaczenie dla Tradera:** Jeśli linia schodzi coraz niżej, kontrolę mają sprzedający. Jeśli skacze w górę, rynek akceptuje wyższe ceny (trend wzrostowy).

---

### C. W czym ten wskaźnik pomaga w codziennym handlu?

Stosując to narzędzie, zyskujesz cztery kluczowe przewagi rynkowe:

#### 1. Koniec z rysowaniem stref "na oko" (Pełny obiektywizm)
Zamiast zastanawiać się, czy poprowadzić linię wsparcia po cieniach świec, czy po korpusach, otrzymujesz strefę wyznaczoną czystą matematyką i wolumenem. Strefa pojawia się tylko wtedy, gdy obrót kapitału przekroczy normę statystyczną (Z-Score lub zadany Percentyl).

#### 2. Ochrona przed tzw. "łapaniem FOMO" (Złe wejścia)
Wskaźnik chroni Cię przed otwieraniem pozycji w najgorszych możliwych momentach. Jeśli widzisz gwałtowną, zieloną świecę wzrostową, ale cena właśnie uderza w historyczną, czerwoną strefę *Supply* (Podaży), wskaźnik ostrzega Cię: *„Nie kupuj tutaj, tuż nad Tobą stoją potężne zlecenia sprzedażowe instytucji”*.

#### 3. Precyzyjne planowanie punktów wyjścia (Take Profit)
Wiedząc, gdzie leżą klastry wolumenowe z poprzednich dni (dzięki funkcji projekcji stref w prawo), możesz idealnie zaplanować realizację zysków. Jeśli grasz pozycję długą (Long), Twoim logicznym celem (Take Profit) jest spód najbliższej czerwonej strefy lub historyczny poziom POC, ponieważ tam cena najprawdopodobniej wyhamuje.

#### 4. Filtracja szumu rynkowego (Czyszczenie wykresu)
Dzięki wbudowanemu algorytmowi wygładzania (KDE), wskaźnik ignoruje przypadkowe anomalie cenowe i pojedyncze, mniejsze zlecenia. Na Twoim wykresie zostają tylko te poziomy, które realnie będą bronione przez algorytmy funduszy inwestycyjnych.
---

## 2. Kluczowe Filary Matematyczno-Ilościowe

### A. Algorytm Pseudo-KDE (Kernel Density Estimation)
Tradycyjne profile wolumenowe cierpią na tzw. szum dyskretyzacji (błędy zaokrągleń wynikające z sztywnego podziału na rzędy/bins). Wskaźnik rozwiązuje ten problem poprzez implementację funkcji `smooth_profile`, która działa jako **Pseudo-KDE** przy użyciu centrowanego okna średniej ruchomej na histogramie wolumenu.
* Wygładza to rozkład wolumenu, eliminując fałszywe mikro-szczyty (Prominence).
* Pozwala na dokładniejsze wyznaczenie rzeczywistego punktu równowagi (POC) oraz granic stref płynności.

### B. Kwantyfikowalne Metody Filtrowania HVN
Wskaźnik odchodzi od subiektywnego wyznaczania stref na rzecz trzech rygorystycznych metod statystycznych, określających, który poziom cenowy kwalifikuje się jako strefa S/D:
* **Kwantyle (Percentyl):** Filtruje rzędy profilu, odcinając wartości poniżej zadanego progu (np. percentyl 85 wyłania top 15% poziomów o najwyższej koncentracji wolumenu).
* **Z-Score (Odchylenie Standardowe):** Wylicza średnią ($\mu$) oraz odchylenie standardowe ($\sigma$) wolumenu dla całego profilu. Poziom strefy aktywowany jest tylko wtedy, gdy wolumen rzędu przekracza wartość $\mu + z \cdot \sigma$.
* **Prominencja (Lokalne Szczyty):** Analizuje topografię profilu i identyfikuje lokalne maksima (rząd jest większy od swoich bezpośrednich sąsiadów), izolując kluczowe klastry płynności bez względu na ich bezwzględny wolumen.

### C. Matematyczna Aproksymacja Delty Rynkowej
Z uwagi na ograniczenia historycznych danych tikowych w TradingView, skrypt implementuje algorytm podziału wolumenu wewnątrz słupka na podstawie relacji ceny zamknięcia do jego rozpiętości (High-Low):

<img width="261" height="91" alt="image" src="https://github.com/user-attachments/assets/ff6b3aa8-4854-4c73-b5c2-57fe64fb7a81" />

Na tej podstawie wyliczana jest **Delta rzędu** ($\Delta$), która pozwala ocenić, czy w danym klastrze wolumenowym dominowała strona popytowa (Delta dodatnia, strefy *Demand* oznaczane kolorem morskim/teal), czy podażowa (Delta ujemna, strefy *Supply* oznaczane kolorem czerwonym).

---

## 3. Architektura Silnika i Funkcje Rynkowe

* **Anchored Multi-Period Engine:** Skrypt zapamiętuje historyczne punkty zwrotne (kotwice czasowe np. sesje dzienne, tygodniowe) i generuje profile wstecz (do 15 okresów historycznych), co pozwala na badanie pamięci rynkowej.
* **Dynamiczny POC (dPOC / Developing POC):** Za pomocą struktur `polyline`, wskaźnik śledzi migrację punktu kontrolnego (POC) wewnątrz trwającego okresu krok po kroku, wizualizując proces relokacji kapitału.
* **Multi-Timeframe Second POC:** Możliwość nałożenia drugiego, nadrzędnego punktu POC z wyższego interwału (np. tygodniowego na profilu dziennym) w celu identyfikacji kluczowych poziomów wsparcia/oporu wyższego rzędu (MTF).
* **Projekcja Stref (Zone Extension):** Historyczne strefy HVN, które nie zostały jeszcze zanegowane, są automatycznie przedłużane w prawo (projekcja), tworząc precyzyjne mapy poziomów reakcji dla obecnej ceny.
* **Dynamic Transparency Mapping:** Przezroczystość wypełnienia stref jest dynamicznie skalowana (funkcja `normalize_transp`) względem maksimum wolumenowego (POC). Im większa koncentracja wolumenu w danym rzędzie, tym bardziej wyrazista jest strefa na wykresie.

---

## 4. Przegląd Parametrów Wejściowych (Inputs)

### Ustawienia Bazowe (Core Settings)
* `Zasięg Profilu (Anchor)`: Definiuje interwał czasowy kotwicy (od 1 godziny do 12 miesięcy).
* `Ilość historycznych okresów`: Określa liczbę wstecznych profili rysowanych na wykresie.
* `Rozdzielczość`: Liczba poziomów cenowych (rzędów) wewnątrz jednego profilu (zakres 10-100).
* `Wygładzanie (Pseudo-KDE)`: Szerokość okna filtrującego (wartości nieparzyste 1-9).

### Metody Ilościowe (Quant Methods)
* `Metoda Obliczeń HVN`: Wybór matematycznego algorytmu odcinającego szum (Percentyl / Z-Score / Prominencja).
* `Kwantyle: Poziom Percentyla`: Próg odcięcia dla metody kwantylowej.
* `Z-Score`: Mnożnik odchylenia standardowego dla metody Z-Score.

---

## 5. Zastosowanie w Algorytmicznych Strategiach Inwestycyjnych

W kontekście handlu ilościowego i automatyzacji (np. integracji z Pythonem czy systemami typu Expert Advisors), wskaźnik dostarcza krytycznych danych wejściowych:
1. **Filtry Egzekucyjne:** Blokowanie pozycji długich (Long) bezpośrednio pod silnymi strefami *Supply* zidentyfikowanymi przez statystykę Z-Score.
2. **Kwantyfikacja Targetów (Take Profit):** Wyznaczanie poziomów docelowych w oparciu o linie MTF POC lub dPOC, gdzie prawdopodobieństwo wyhamowania pędu ceny jest statystycznie najwyższe.
3. **Generowanie Sygnałów Mean-Reversion:** Wykorzystanie anomalii cenowych (odchylenia od wygładzonych stref KDE) do handlu powrotnego do średniej.

---

## 6. Ograniczenia Środowiska, Wady i Uproszczenia Modelowe (Model Risk)

Z punktu widzenia matematyki finansowej oraz rygorystycznego handlu ilościowego, implementacja zaawansowanych profili wolumenowych w środowisku Pine Script (v6) wymaga przyjęcia istotnych kompromisów. Poniżej znajduje się krytyczna analiza ograniczeń technologicznych oraz wprowadzonych uproszczeń, które należy uwzględnić przy ocenie ryzyka modelowego (Model Risk).

### A. Syntetyczna Delta vs Rzeczywisty Order Flow (Market Microstructure)
* **Ograniczenie:** Środowisko Pine Script na standardowych interwałach historycznych nie ma bezpośredniego dostępu do surowych danych L1/L2 (Order Book / Time & Sales). True Order Flow wymaga rozliczania transakcji po cenie Ask/Bid w celu określenia agresji rynkowej.
* **Uproszczenie:** Wskaźnik stosuje matematyczną aproksymację delty na podstawie relacji ceny zamknięcia do rozpiętości słupka (OHLC Proxy). 
* **Wada handlowa:** Wprowadza to błąd śledzenia (tracking error) w momentach wysokiej zmienności i niskiej płynności. Syntetyczna delta może błędnie zaklasyfikować wolumen jako pro-popytowy, podczas gdy na poziomie mikrostruktury rynku (Tick-by-Tick) mogło dojść do zleceń ukrytych (Iceberg) po stronie podaży.

### B. Pseudo-KDE zamiast Ciągłego Estymatora Jądrowego Gęstości
* **Ograniczenie:** Pełna, ciągła estymacja jądrowa gęstości (Gaussian KDE) wymaga kosztownych obliczeniowo operacji zmiennoprzecinkowych (całkowanie numeryczne, funkcja Gaussa) dla każdego punktu cenowego. Wykonanie takiego algorytmu na historii 5000 barów w Pine Script skutkowałoby przekroczeniem limitu czasu wykonania skryptu (Script Timeout Error).
* **Uproszczenie:** Zastosowano dyskretną aproksymację KDE przy użyciu symetrycznego okna średniej ruchomej (SMA) na stabelaryzowanych rzędach wolumenu (`smooth_profile`).
* **Wada handlowa:** Wygładzanie dyskretne ma charakter kaskadowy i jest zależne od parametru `rows` (rozdzielczości). Zbyt niska rozdzielczość w połączeniu z dużym oknem wygładzania może doprowadzić do przesunięcia (shiftu) matematycznego punktu POC względem rzeczywistej, historycznej koncentracji kapitału.

### C. Sztywne Limity Pamięciowe i Graficzne (Runtime Constraints)
* **Ograniczenie:** Pine Script narzuca restrykcyjne limity architektoniczne: maksymalnie 5000 barów wstecz do analizy wolumenu, limit 500 obiektów typu `box` oraz 100 obiektów `polyline` wyświetlanych jednocześnie na wykresie.
* **Uproszczenie:** Maksymalna rozdzielczość profilu została ograniczona do 100 rzędów (`rows = 100`), a liczba monitorowanych okresów historycznych do 15 (`history_count = 15`).
* **Wada handlowa:** Uniemożliwia to prowadzenie ciągłej analizy struktury rynkowej High-Frequency (HFT) w długim horyzoncie czasowym na niskich interwałach (np. 1-minutowych). Algorytm zmuszony jest do agregacji danych, co zaciera mikro-klastry płynności, kluczowe dla precyzyjnego pozycjonowania zleceń typu Stop Loss.

### D. Brak Adaptacyjnej Zmienności (Fixed Step Binning)
* **Ograniczenie:** Podział profilu na strefy opiera się na sztywnej, liniowej interpolacji ceny (`(maxP - minP) / rows`). 
* **Uproszczenie:** Krok ceny (bin size) jest stały w obrębie jednego profilu i nie reaguje na dynamiczne zmiany zmienności rynkowej (np. poprzez implementację ATR - Average True Range).
* **Wada handlowa:** Podczas gwałtownych impulsów cenowych (np. publikacje danych makroekonomicznych), sztywny podział powoduje, że rzędy stają się zbyt szerokie, co drastycznie zmniejsza precyzję generowanych stref S/D, czyniąc je zbyt ogólnymi dla systemów automatycznych egzekwowanych z poziomu API (np. w Pythonie).

### E. Izolacja Wolumenu od Czasu (Time-Volume Bias)
* **Ograniczenie:** Profil wolumenowy mierzy wyłącznie wolumen skumulowany po konkretnej cenie, ignorując wymiar czasu, przez jaki cena przebywała na danym poziomie wewnątrz sesji.
* **Wada handlowa:** Poziom o wysokim wolumenie wygenerowany przez jedną potężną transakcję blokową (Block Trade) traktowany jest tak samo, jak poziom budowany przez wielogodzinną dystrybucję. Z perspektywy teorii aukcji rynkowej (Market Profile), poziomy te mają zupełnie inną wartość informacyjną dla tradingu pozycyjnego.

---

*Disclaimer: Materiały zawarte w tym repozytorium mają charakter wyłącznie badawczo-edukacyjny i nie stanowią rekomendacji inwestycyjnych.*
