# Handel Algorytmiczny i Rynki finansowe - Strategie, wskaźniki, zarządzanie ryzykiem

Repozytorium zawiera autorskie skrypty, wskaźniki oraz kompletne systemy transakcyjne dedykowane dla rynków kontraktów terminowych (indeksy USA oraz kryptowaluty). Projekt stanowi portfolio moich kompetencji w zakresie handlu ilościowego (Quantitative Trading), analizy danych oraz automatyzacji procesów decyzyjnych.

Od 5 lat aktywnie łączę handel na rynkach finansowych z programowaniem własnych narzędzi. Moje podejście opiera się na eliminacji czynnika uznaniowego na rzecz przewagi statystycznej, rygorystycznej kontroli ryzyka oraz automatyzacji egzekucji zleceń.

---

## 🛠️ Architektura Projektu

Kod został podzielony strukturalnie według wykorzystywanych technologii i środowisk:

### 🐍 Python (`/python`)
Warstwa analityczna i badawcza. Python stanowi główne narzędzie w moim workflow do poszukiwania przewagi rynkowej (market edge).
* **Zaawansowany Backtesting:** Autorskie środowiska testowe pozwalające na symulację strategii z uwzględnieniem kosztów transakcyjnych, poślizgów cenowych (slippage) oraz płynności.
* **Obliczenia Statystyczne:** Przetwarzanie dużych zbiorów danych rynkowych, badanie dystrybucji zwrotów, korelacji oraz implementacja modeli ilościowych.
* **Integracje API (DeFi/CeFi):** Skrypty do obsługi danych w czasie rzeczywistym i historycznych (np. analiza funding rates, tracking wolumenu i pozycji).

### 🌲 Pine Script (`/pine-script`)
Prototypowanie, analiza wizualna oraz środowisko testowe na platformie TradingView.
* **Wskaźniki Statystyczne (v5):** Narzędzia filtrujące szum rynkowy na podstawie zaawansowanych wyliczeń matematycznych.
* **Systemy Algorytmiczne:** Strategie intraday zaimplementowane w celu wstępnej weryfikacji hipotez rynkowych.

### 📊 MQL5 (`/mql5`)
Warstwa egzekucyjna i produkcyjna dla platformy MetaTrader 5.
* **Expert Advisors (EAs):** W pełni autonomiczne roboty handlowe realizujące założenia systemów matematycznych w handlu na żywo.
* **Moduły Zarządzania Ryzykiem:** Biblioteki odpowiedzialne za dynamiczne kalkulowanie wielkości pozycji, kontrolę obsunięć (drawdown) oraz trailing stopy.

---

## 💻 Stack Technologiczny

* **Języki programowania:** Python, Pine Script (v5), MQL5
* **Analiza danych & Ekonometria:** Pandas, NumPy, Statsmodels, SciPy
* **Komunikacja & Narzędzia:** REST API, WebSockets, Git

---

## 📈 Metodologia Badawcza

Każda koncepcja rynkowa przed wdrożeniem produkcyjnym przechodzi rygorystyczną ścieżkę walidacji – od analizy statystycznej w Pythonie, przez wizualny backtest, aż po testy stabilności kodu egzekucyjnego. Kluczowymi metrykami efektywności są dla mnie Profit Factor, Sharpe Ratio oraz maksymalny Drawdown (MaxDD), a nie wyłącznie nominalna stopa zwrotu.

---

*Disclaimer: Materiały zawarte w tym repozytorium mają charakter wyłącznie badawczo-edukacyjny i nie stanowią rekomendacji inwestycyjnych.*
