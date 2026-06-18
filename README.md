# Quantitative Trading & Algorithmic Strategies

Cześć! To repozytorium to moja piaskownica (i jednocześnie portfolio) związana z **handlem ilościowym (Quantitative Trading)** i automatyzacją strategii. Łączę tutaj analizę danych rynkowych z kodowaniem gotowych systemów transakcyjnych. 

Skupiam się głównie na rynkach kontraktów terminowych (indeksy USA, krypto) oraz dynamicznym zarządzaniu ryzykiem. Repozytorium traktuję jako rozszerzenie mojego CV – znajdziesz tu żywy dowód na to, jak przekładam matematykę i logikę rynkową na działający kod.

---

## 🛠️ Co tu znajdziesz? (Struktura projektu)

Projekt podzieliłem na trzy główne filary technologiczne:

### 🐍 Python (`/python`)
Tutaj dzieje się cała "brudna robota" związana z analizą danych i backendem.
* **Integracje API & DeFi:** Skrypty pobierające i analizujące dane (np. historyczne funding rates, tracking portfeli wielorybów na platformach typu Hyperliquid).
* **Analiza Statystyczna:** Modele matematyczne, badanie zmienności (GARCH, regresja lokalna) i przetwarzanie surowych danych rynkowych za pomocą Pandas i NumPy.

### 🌲 Pine Script (`/pine-script`)
Szybkie prototypowanie i analiza wizualna na TradingView.
* **Autorskie wskaźniki:** Implementacje zaawansowanych konceptów statystycznych, które pomagają odsiać szum rynkowy.
* **Backtesting:** Gotowe strategie (v5) zoptymalizowane pod trading intraday, gdzie liczy się precyzja wejścia i sztywne zasady wyjścia.

### 📊 MQL5 (`/mql5`)
Czysta egzekucja i automatyzacja na platformie MetaTrader 5.
* **Expert Advisors (EAs):** Boty handlowe, które bez emocji realizują założenia strategii 24/7.
* **Risk Management:** Skrypty i biblioteki pomocnicze odpowiedzialne za dynamiczne wyliczanie wielkości pozycji i trailing stopy.

---

## 💻 Mój Stack

* **Języki:** Python, Pine Script (v5), MQL5
* **Analiza danych:** Pandas, NumPy, Statsmodels, SciPy
* **Inne:** REST/Websockets API, Git

---

## 📈 Jak podchodzę do tradingu?
Nie szukam "świętego graala". Moje podejście opiera się na **radykalnym zarządzaniu ryzykiem** i szukaniu powtarzalnych przewag statystycznych. Każda strategia lądująca w tym repozytorium przechodzi przez rygorystyczny backtesting (z uwzględnieniem kosztów transakcyjnych i poślizgów), gdzie kluczowe wskaźniki to dla mnie Drawdown, Profit Factor i Sharpe Ratio, a nie tylko sucha stopa zwrotu.

---

*Disclaimer: Kod zawarty w tym repozytorium służy wyłącznie celom badawczym i edukacyjnym.*
