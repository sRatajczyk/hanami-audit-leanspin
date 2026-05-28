> ## STATUS DOKUMENTU
> Niniejszy dokument jest HIPOTEZĄ założyciela — wstępną propozycją
> wyceny istniejącej IP do walidacji przez audytora zewnętrznego (LeanSpin).
> Wszystkie kwoty są szacunkowe i wymagają niezależnej weryfikacji.
> Nie jest to oferta ani zobowiązanie finansowe.

---

# WYCENA IP — SE Platform (Hanami)
**Wersja:** hipoteza robocza, 2026-05-28
**Przedmiot:** istniejąca IP na branchu `master`, HEAD `ff4208f`
**Autor:** Sebastian Ratajczyk

---

## 1. CO JEST PRZEDMIOTEM WYCENY

Wycena dotyczy aktywów IP istniejących w dniu dzisiejszym — kodu, wiedzy domenowej i algorytmów wbudowanych w działający produkt. **Nie obejmuje** przyszłej pracy deweloperskiej ani przyszłego rozwoju produktu (te pozycje są w USE_OF_FUNDS_HIPOTEZA.md).

### Aktywa IP na masterze

| Aktywo | Status | Unikalne w PL? | Czas odtworzenia |
|---|---|---|---|
| **Silnik rozliczeniowy art. 38c** (552 LOC, zweryfikowany 100% vs Excel SE Jaworze) | ✅ Działa na produkcji | **TAK** — brak publicznego open-source dla PL regulacji | 3–5 m-m (dev + prawnicy + dane OSD) |
| **Silnik optymalizatora PV** (1 870 LOC scripts + 1 619 LOC views) — analiza K1–K4, taryfy B/C, multi-profil, battery sizing, projekcja 5-letnia | ✅ Działa | **TAK** — analiza K1/K2/K3/K4 pod polskie taryfy dystrybucyjne | 2–3 m-m |
| **Parser OSD Tauron** (476 LOC) — produkcyjny, registry pattern, AI fallback | ✅ Działa na produkcji | Nie (parsery CSV istnieją) — ale wariant dla Tauron + metodologia | 0,5–1 m-m |
| **Parsery OSD PGE/ENEA/Energa** (szkielety, 197 LOC `pge_parser.py`) | ⚠️ W trakcie | Nie — szkielety, wymaga uzupełnienia | 1,5 m-m (3 OSD) |
| **Architektura multi-tenant** (OrgModel, OrgManager, middleware) | ✅ Działa | Nie (wzorzec Django) — ale implementacja + konwencja | 1–1,5 m-m |
| **Workflow 10-krokowy** (rozliczenie miesięczne, auto-detekcja kroków) | ✅ Działa | **TAK** — specyfika procesu SE jest wbudowana | 1 m-m |
| **Generator proform PDF** (WeasyPrint, Jinja2, bulk email Mailgun) | ✅ Działa | Nie (PDF generation standard) — ale integracja z rozliczeniem | 0,5–1 m-m |
| **Generator raportów optymalizatora PDF** (1 047 LOC) | ✅ Działa | Nie — ale treść jest unikalna (metodologia) | 0,5–1 m-m |
| **System uprawnień RBAC** (5 ról, zaproszenia tokenem, hierarchia) | ✅ Działa | Nie | 0,5 m-m |
| **Panel admina SE** (ECharts — Sankey, donut, historia 12m; dashboard, hourly, annual) | ✅ Działa | Nie (UI standard) — ale integracja z danymi SE | 2–2,5 m-m |
| **Panel członka SE** (/me/, oszczędności, dokumenty, historia) | ✅ Działa | Nie | 0,5–1 m-m |
| **Infrastruktura** (Docker Compose, Celery + Redis, GitHub Actions CI, Sentry) | ✅ Działa | Nie | 0,3–0,5 m-m |
| **Wiedza domenowa w kodzie** (interpretacja art. 38c, metodologia KOWR, specyfika OSD, taryfy dystrybucyjne B11/B12/C11) | ✅ Wbudowana | **TAK** — niemożliwa do oddzielenia od kodu | Nie odtwarzalna bez eksperta |

**Kluczowa obserwacja:** Trzy elementy są naprawdę unikalne w skali PL i nie mają odpowiedników open-source: silnik rozliczeniowy (art. 38c), implementacja K1–K4 dla polskich taryf dystrybucyjnych, wiedza domenowa zaszywa w algorytmach. Pozostałe komponenty to dobrze wykonana, ale standardowa inżynieria oprogramowania.

---

## 2. METODA 1: KOSZT ODTWORZENIA (Replacement Cost)

**Pytanie:** Ile kosztowałoby zbudowanie tego od zera przez zewnętrzną firmę?

Założenia: senior developer Django/Python w PL, **bez wiedzy domenowej** o art. 38c. Stawka dzienna: 1 750 zł (rynek 2026, B2B). Stawka miesięczna ekwiwalentna: ~35 000 zł (20 dni roboczych).

| Moduł | Dev (m-m) | Domena (m-m) | Test (m-m) | Razem |
|---|---|---|---|---|
| Silnik rozliczeniowy art. 38c (bateria, WI, korekta depozytowa) | 2,5 | 2,0 | 0,5 | **5,0** |
| Optymalizator PV (K1–K4, taryfy B/C, multi-profil, battery sizing) | 2,0 | 1,0 | 0,5 | **3,5** |
| Parsery OSD (Tauron prod. + PGE/ENEA/Energa) | 0,5 | 0,5 | 0,5 | **1,5** |
| Generator proform PDF + bulk email | 0,5 | 0,3 | 0,2 | **1,0** |
| Generator raportów optymalizatora PDF | 0,5 | 0,2 | 0,3 | **1,0** |
| Architektura multi-tenant (OrgModel, middleware) | 1,0 | 0,0 | 0,3 | **1,3** |
| Workflow 10-krokowy (auto-detekcja, KOWR) | 0,5 | 0,3 | 0,2 | **1,0** |
| Panel admina SE (dashboard, wykresy, hourly, annual) | 1,5 | 0,3 | 0,2 | **2,0** |
| Panel członka SE | 0,5 | 0,2 | 0,1 | **0,8** |
| RBAC (5 ról, zaproszenia, hierarchia) | 0,5 | 0,0 | 0,2 | **0,7** |
| Infrastruktura (Docker, Celery, CI/CD) | 0,3 | 0,0 | 0,2 | **0,5** |
| **ŁĄCZNIE** | **10,3** | **4,8** | **3,2** | **18,3** |

**Kalkulacja:**

| Pozycja | Kwota |
|---|---|
| 18,3 m-m × 35 000 zł/m-m (senior dev B2B) | 640 500 zł |
| Marża projektowa 30% (standardowa dla prac na zamówienie) | +192 150 zł |
| Koszt zarządzania projektem 15% (PM / koordynacja) | +96 075 zł |
| **KOSZT ODTWORZENIA BRUTTO** | **~928 000 zł** |

*Widełki: przy stawce 28 000–35 000 zł/m-m koszt odtworzenia wynosi **630 000–928 000 zł** przed marżą. Z marżą: **820 000–928 000 zł**.*

*Niepewność: estymacja czasu dla silnika rozliczeniowego (5 m-m) może być niedoszacowana — bez konkretnego prawnika energetycznego i dostępu do danych OSD czas może wzrosnąć do 7–8 m-m, co podniosłoby całość do 1,0–1,1 mln zł.*

---

## 3. METODA 2: SWEAT EQUITY (wkład pracy założyciela)

**Pytanie:** Ile Sebastian zarobiłby gdyby te godziny sprzedał na rynku?

**Czas budowy (realny):**

| Aktywność | Godziny |
|---|---|
| Kodowanie (15 dni roboczych × 12h — intensywny sprint) | 180h |
| Research regulacyjny (art. 38c, Dz.U. 2022 poz. 703, metodologia KOWR) | 50h |
| Projektowanie produktu (architektura, priorytety, decyzje techniczne) | 35h |
| Testowanie z realnymi danymi OSD (SE Jaworze) | 25h |
| **Łącznie** | **290h** |

*Kontekst: 290h w ~3 tygodnie = średnio ~14h/dzień. Możliwe dzięki AI-first workflow (Claude Code jako pair programmer). Ważne: godziny liczymy, bo audytor je sprawdzi — ale wartość IP nie wynika z czasu, lecz z unikalności wiedzy.*

**Trzy warianty wyceny:**

| Rola | Godziny | Stawka rynkowa (PL 2026) | Wartość |
|---|---|---|---|
| Senior Developer (Django/Python) | 180h | 175 zł/h | 31 500 zł |
| CTO / Product Owner (z wiedzą domenową SE) | 290h | 300 zł/h | **87 000 zł** |
| Konsultant regulacyjny art. 38c (takich ekspertów w PL jest <20) | 290h | 450 zł/h | 130 500 zł |

**Rekomendacja:** wariant CTO/PO = **87 000 zł** jako sweat equity value. Sebastian pełnił wszystkie trzy role jednocześnie — stawka 300 zł/h jest defensywna i weryfikowalna rynkowo.

*Obserwacja: sweat equity (87 000 zł) jest dramatycznie niższy niż koszt odtworzenia (820 000+ zł). Ta różnica — ~733 000 zł — to wartość stworzona przez efektywność AI-first buildu i wiedzę domenową, której zewnętrzna firma nie ma.*

---

## 4. METODA 3: FAIR MARKET VALUE (wartość rynkowa IP)

### 4A. Value-based (% z wartości dostarczanej klientowi)

| Parametr | Wartość |
|---|---|
| Pipeline: 100 SE × średnio 2 500 MWh/rok | 250 000 MWh/rok |
| Oszczędność z automatyzacji: 250 000 × 17,5 zł/MWh | 4 375 000 zł/rok |
| Oszczędność w modelu Greenlab: 100 SE × 0,5 etatu × 6 000 zł | 3 600 000 zł/rok |

*Użyto średniej 17,5 zł/MWh z przedziału 15–20 zł/MWh. Greenlab oszczędza dodatkowe ~3,6 mln zł na samych etatach niezatrudnionych przy ręcznej obsłudze.*

**Multiplier IP SaaS B2B** (1–3× rocznej wartości dostarczanej, zależnie od dojrzałości):

| Multiplier | Roczna wartość dostarczana | Wycena IP |
|---|---|---|
| 1× (MVP, 1 klient) | 4 375 000 zł | 4 375 000 zł |
| 2× (potwierdzone skalowanie) | 4 375 000 zł | 8 750 000 zł |
| 3× (dojrzały produkt, lock-in) | 4 375 000 zł | 13 125 000 zł |

Przy obecnym etapie (MVP, 1 klient, brak proven skalowania) uzasadniony jest multiplier **1×**, co daje wycenę IP na poziomie **~4,4 mln zł**.

### 4B. Porównawczy (comparable transactions)

**Exnaton (Szwajcaria):**
- ETH spinoff, 2020, platforma SaaS dla community energy (model B2B2C)
- 50+ utilities w 5 krajach europejskich, ~25–50 pracowników
- Series A (2025): kwota nieujawniona; na podstawie europejskich benchmarków energtech pre-scale ~3–8 mln EUR
- Implikowana wycena przy Series A: ~15–40 mln EUR

Korekty dla porównania z SE Platform:
- Discount pre-seed vs Series A: **−92%** (typowy dla oprogramowania na tym etapie)
- Discount: inny rynek (utilities vs SE), inny model, brak polskiej lokalizacji

| Scenariusz | Wycena Exnaton | Discount | Implikowana wycena SE Platform |
|---|---|---|---|
| Pesymistyczny | 15 mln EUR = 65 mln zł | −95% | 3 250 000 zł |
| Bazowy | 25 mln EUR = 108 mln zł | −93% | 7 560 000 zł |
| Optymistyczny | 40 mln EUR = 172 mln zł | −92% | 13 760 000 zł |

*Zastrzeżenie: to jest bardzo zgrubne porównanie. Exnaton działa na innych rynkach, w innym modelu prawnym, z innym zespołem i produktem. Audytor powinien zweryfikować lub odrzucić tę metodę.*

### 4C. DCF uproszczone (5-letnie)

**Założenia:**
- Przychód per SE: 18 000 zł/rok (1 500 zł/mies.) — konserwatywny, poniżej sufitu rynkowego (37 500–50 000 zł/rok)
- Koszty operacyjne: z Use of Funds rok 1 (1 810 000 zł), w kolejnych latach malejący % przychodu
- Stopa dyskontowa: 30% (early stage, single founder, rynek regulowany)

| Rok | SE aktywne | Przychód (PLN) | Koszty operac. | EBIT | Czynnik dysk. @30% | PV (PLN) |
|---|---|---|---|---|---|---|
| 1 | 50 śr. | 900 000 | 1 810 000 | −910 000 | ÷ 1,300 | −700 000 |
| 2 | 150 | 2 700 000 | 1 200 000 | 1 500 000 | ÷ 1,690 | 888 000 |
| 3 | 300 | 5 400 000 | 1 600 000 | 3 800 000 | ÷ 2,197 | 1 730 000 |
| 4 | 500 | 9 000 000 | 2 000 000 | 7 000 000 | ÷ 2,856 | 2 451 000 |
| 5 | 700 | 12 600 000 | 2 500 000 | 10 100 000 | ÷ 3,713 | 2 719 000 |
| **NPV (bez wartości końcowej)** | | | | | | **7 088 000 zł** |

*Przy stopie dyskontowej 35% NPV spada do ~6 100 000 zł. Bez wartości końcowej (exit). Dodanie wartości końcowej przy exit 3× przychód roku 5 podnosiłoby NPV do ~17–20 mln — liczby spekulatywne na tym etapie.*

**DCF VALUE: 6 100 000 – 7 100 000 zł** (przy 30–35% dyskonta, bez wartości końcowej)

---

## 5. TABELA PORÓWNAWCZA TRZECH METOD

| Metoda | Dolna granica | Górna granica | Komentarz |
|---|---|---|---|
| **Koszt odtworzenia** | 630 000 zł | 928 000 zł | Ile kosztuje zbudować to samo od zera (zewnętrzna firma, bez wiedzy domenowej) |
| **Sweat equity** | 87 000 zł | 130 500 zł | Ile founder włożył własnej pracy w cenach rynkowych |
| **Fair market value — value-based** | 4 375 000 zł | 13 125 000 zł | % wartości dostarczanej klientowi (1–3× annual value) |
| **Fair market value — comparable** | 3 250 000 zł | 13 760 000 zł | Discount od Exnaton pre-seed (spekulatywne) |
| **Fair market value — DCF** | 6 100 000 zł | 7 100 000 zł | NPV 5-letni @30–35% bez wartości końcowej |

**Obserwacja:** Trzy metody dają bardzo różne wyniki — to normalne i oczekiwane. Koszt odtworzenia to "podłoga" (minimalne uzasadnienie ceny), sweat equity to punkt odniesienia dla foundera, FMV to "sufit" (wartość potencjalna). License Setup Fee powinno mieścić się między podłogą a sufitem.

---

## 6. REKOMENDACJA: LICENSE SETUP FEE

### Logika ustalenia ceny

**Setup Fee powinno spełniać cztery warunki:**
1. **≥ sweat equity** — founder nie sprzedaje godzin, sprzedaje IP
2. **≈ koszt odtworzenia** — najbardziej audytowalny punkt odniesienia ("kupiłeś poniżej kosztu budowy")
3. **< fair market value** — pozostawia przestrzeń na ongoing royalty i aligned incentives
4. **Wytrzymać audyt NIK** — uzasadnione rynkowo, z dokumentacją

### Rekomendowana kwota

**License Setup Fee: 500 000 zł (jednorazowo)**

| Test | Wynik |
|---|---|
| Wyższe niż sweat equity (87 000 zł)? | ✅ TAK (+474%) |
| Poniżej kosztu odtworzenia brutto (928 000 zł)? | ✅ TAK (−46%) |
| Znacznie poniżej FMV (6,1–13 mln zł)? | ✅ TAK (−92% do −96%) |
| Możliwe do obronienia jako "zakup poniżej rynku" dla RSSI? | ✅ TAK |

**Uzasadnienie dla audytora NIK:** RSSI nabywa dostęp do IP, której koszt odtworzenia wynosi 820 000–928 000 zł. Cena 500 000 zł oznacza zakup z dyskontem ~46% do kosztu rynkowego budowy — co jest transakcją korzystną dla nabywcy (RSSI), a nie dla sprzedającego. Brak przesłanek do kwestionowania ceny jako zawyżonej.

### Pełna kompensacja za IP (Setup Fee to nie wszystko)

Setup Fee jest opłatą jednorazową za dostęp do istniejącej IP. Pełna kompensacja za IP w czasie składa się z:

| Element | Charakter | Kwota / stawka |
|---|---|---|
| **License Setup Fee** | jednorazowy | 500 000 zł |
| **Royalty od przychodu** | miesięczny / per SE | do uzgodnienia z audytorem |
| **Minimum gwarantowane** | roczne | do uzgodnienia |
| **Klauzula change-of-control** | przy exit / sprzedaży | do uzgodnienia |

**Własność IP pozostaje przy Sebastianie Ratajczyku** — umowa jest licencją wyłączną (lub niewyłączną — do negocjacji), nie sprzedażą. Klauzula change-of-control zabezpiecza przychód foundera przy exit Greenlab/RSSI.

*Niepewność: rekomendacja 500 000 zł jest oparta na metodzie kosztu odtworzenia — najbardziej konserwatywnej i audytowalnej. Audytor LeanSpin może zaproponować inną kwotę na podstawie FMV, co Sebastian zaakceptuje jako punkt wyjścia do negocjacji.*

---

## 7. CO TEN DOKUMENT NIE WYCENIA

Poniższe elementy są celowo wyłączone z niniejszej wyceny IP:

**Przyszła praca Sebastiana** — wynagrodzenie CTO/PO w Use of Funds (30 000 zł/mies.) pokrywa przyszły czas pracy. Nie jest to część wartości istniejącej IP.

**Przyszły rozwój produktu** — hardening (T1–T10), multi-OSD, E2E testy, refaktor views.py — to inwestycja z Use of Funds, nie istniejąca IP.

**Wartość relacji z klientem (Greenlab/RSSI)** — dostęp do pipeline 100 SE z KOWR, zaufanie prezesa Greenlab, pozycja first-mover — realna i znacząca wartość, ale niemierzalna metodami finansowymi.

**Wartość pozycji first-mover na rynku PL SE** — rynek rośnie >300% rocznie (19 SE w 2023 → 669 SE w marcu 2026). Bycie pierwszym działającym produktem na tym rynku ma wartość strategiczną, której nie oddaje żadna z trzech metod powyżej.

**Dane klienta (SE Jaworze)** — dane OSD i wyniki rozliczeń użyte do weryfikacji silnika mają wartość jako "ground truth" dla przyszłych testów. Nie są własnością Sebastian Ratajczyk — należą do SE Jaworze.
