# Propozycja zakresu audytu techniczno-biznesowego

**Adresat:** LeanSpin
**Produkt:** SE Platform (Hanami) — SaaS do zarządzania spółdzielniami energetycznymi
**Zakres kodu:** branch `master`, tag `v0.3.0-etap2-stable` (aktualny HEAD: `ff4208f`, 2026-05-28)
**Tryb:** audyt ekspresowy

---

## 1. KONTEKST

SE Platform to pionowe oprogramowanie SaaS do zarządzania i rozliczania polskich spółdzielni energetycznych (SE) na podstawie art. 38c–38e ustawy OZE oraz Dz.U. 2022 poz. 703. Produkt obsługuje cały cykl miesięczny: import danych pomiarowych z OSD (operatora sieci dystrybucyjnej), algorytmiczne rozliczenie energii zbiorowej, generowanie proform i raportów finansowych, wysyłkę do członków. Druga usługa komercyjna to moduł optymalizatora SE — narzędzie doradcze do doboru mocy źródeł OZE z pełnym raportem PDF.

**Klient docelowy:** Greenlab Sp. z o.o. — spółka z udziałem podmiotów państwowych powiązanych z KOWR, powołana z misją zakładania i operacyjnej obsługi spółdzielni energetycznych dla lokalnych oddziałów KOWR w skali ogólnopolskiej. Zakres: rejestracja SE (KRS, KOWR), uruchomienie operacyjne, rozliczenia miesięczne, komunikacja z członkami i OSD. Greenlab pełni rolę centralnego operatora — jedno środowisko technologiczne dla wszystkich obsługiwanych SE. Bez automatyzacji rozliczeń program zatrzymuje się operacyjnie przy 2–3 SE (fizyczna granica Excela), co czyni platformę SE warunkiem technicznym istnienia tego biznesu w skali.

**Inwestor:** SSRI — spółka inwestycyjna powiązana ze środowiskiem KOWR, dysponująca kapitałem przeznaczonym na inwestycje w rozwój OZE. Pełna decyzyjność operacyjna po stronie jednego managera.

**Pierwszy klient produkcyjny:** SE Jaworze (Tauron OSD), 18 PPE, dane pomiarowe kwiecień 2026. Platforma działa — pierwsze rozliczenie uruchomione, proformy wygenerowane.

**Kontekst transformacji u klienta:** Obecna prezes Greenlab (od marca 2026, prawnik z wykształcenia, pełna decyzyjność operacyjna) przejęła spółkę po kilkunastoosobowym zespole, który przez 3 lata zajmował się rozwijaniem programu SE — głównie formalnościami, rejestracjami i próbami uruchomienia spółdzielni, z nakładami rzędu 4 mln zł i ograniczonymi efektami. Poprzednia ekipa w ogóle nie podeszła do budowy narzędzia do rozliczeń — obecnie identyfikowane jako kluczowa luka blokująca skalowanie programu.

**Pipeline:** ~100 SE w fazie rejestracji w KOWR, planowane podpięcie do platformy w pierwszym półroczu od uruchomienia.

**Etap produktu:** MVP funkcjonalny z jednym klientem produkcyjnym i działającym silnikiem rozliczeniowym (zweryfikowany 100% vs Excel klienta). Architektura multi-tenant od początku, ale produkt wymaga prac dostosowawczych przed obsługą 5+ SE (zarządzanie użytkownikami, branding, parametry regulacyjne, automatyzacja onboardingu).

---

## 2. STAN OBECNY (do walidacji przez audytora)

Dane z INWENTARYZACJA_PRODUKTU.md (wygenerowanej z aktualnego kodu mastera):

**Metryki kodu:**

| Metryka | Wartość |
|---|---|
| Python LOC (apps/ + scripts/) | ~17 568 |
| HTML templates LOC (szacunek) | ~3 500–4 000 |
| Aktywne Django apps | 12 |
| Modele Django | 18 |
| HTTP endpoints | ~57 |
| Testy (pytest, master) | 158 passed, 1 skipped, 1 xfailed |
| Czas przejścia testów | ~51s |
| CI/CD | GitHub Actions (push na dev + master) |

**Funkcjonalności zaimplementowane:**

| Obszar | Status |
|---|---|
| Silnik rozliczeniowy (art. 38c) | ✅ Produkcja, zweryfikowany |
| Parser danych OSD (Tauron) | ✅ Produkcja |
| Optymalizator SE (PDF report) | ✅ Produkcja |
| Proformy PDF + bulk email send | ✅ Produkcja |
| Panel admina SE (dashboard, wykresy) | ✅ Produkcja |
| Panel członka SE | ✅ Produkcja |
| Workflow miesięczny 10-krokowy | ✅ Produkcja |
| Multi-tenant + RBAC (5 ról) | ✅ Produkcja |
| Onboarding wizard nowej SE | ✅ Produkcja |
| Parametry regulacyjne (runtime) | ✅ Produkcja |
| API publiczne | ❌ Brak |
| E2E testy (Playwright) | ❌ Brak |
| Per-SE branding (white-label) | ⚠️ Częściowe |
| Email parser IMAP (auto-import) | ⚠️ Niezintegrowany |

**Czego brakuje przed pierwszym skalowaniem (własna ocena):**

| Obszar | Problem | Bloker skalowania? |
|---|---|---|
| Izolacja tenant | Brak PostgreSQL RLS — tylko dyscyplina kodu | **TAK** przy >1 developer |
| File storage | Lokalny filesystem kontenera | **TAK** przy multi-instance |
| Email bulk send | Synchroniczny w widoku (timeout przy 100+ odbiorców) | TAK przy dużych SE |
| E2E testy | Brak Playwright — krytyczne flow niezweryfikowane | Tak (ryzyko regresji) |
| views.py monolity | optimizer/views.py 1 619 LOC, dashboard/views.py 1 062 LOC | Nie teraz, tak przy >2 devs |
| White-label | Brak per-SE brandingu dla klientów platformy | Zależy od modelu B2B |
| API publiczne | Brak REST API — blokuje integracje OSD, portale | Nie teraz |
| Legacy w repo | FastAPI app/ + enso/settings.py — zaśmiecają strukturę | Nie (kosmetyczne) |

---

## 3. ZAKRES AUDYTU

### 3.1 Audyt techniczny (priorytet: wysoki)

**A. Ocena ogólna jakości kodu i architektury**

- Czy OrgModel pattern (`.for_org()`) jest konsekwentnie stosowany we wszystkich widokach — czy są miejsca z niezabezpieczonym `Model.objects.filter()`?
- Czy `apps/optimizer/views.py` (1 619 LOC) i `apps/dashboard/views.py` (1 062 LOC) zawierają logikę, która powinna być w serwisach/domenach — i jak pilna jest ta refaktoryzacja?
- Jak oceniasz jakość `scripts/settlement/internal_engine.py` (552 LOC) jako implementacji regulacji Dz.U. 2022 poz. 703 — czy wzorce są defensywne, czy są ukryte założenia, które mogą zawieść na innych danych OSD?
- Czy legacy kod (`app/`, `enso/settings.py`, plik z rotowanym SECRET_KEY) stanowi jakiekolwiek ryzyko bezpieczeństwa czy tylko dług porządkowy?

**B. Production-readiness checklist**

| Obszar | Do weryfikacji |
|---|---|
| Secrets management | Czy `.env` / zmienne środowiskowe są poprawnie oddzielone od kodu? |
| HTTPS / HSTS | Czy nagłówki bezpieczeństwa (HSTS, CSP, X-Frame) są ustawione? |
| Database connection pooling | Czy `CONN_MAX_AGE` jest ustawiony? |
| Celery error handling | Co dzieje się gdy task Celery padnie? Czy są retries? |
| File upload security | Czy walidacja rozszerzenia + rozmiaru jest wystarczająca? Path traversal? |
| Rate limiting coverage | Które endpointy nie mają rate limitu? |
| Session security | Czy ciasteczka sesji mają `Secure`, `HttpOnly`, `SameSite`? |
| Email delivery | Czy Mailgun jest skonfigurowany z DKIM/SPF? |
| Backup / recovery | Czy jest plan backupu PostgreSQL? |
| Logging | Czy błędy trafiają do Sentry? Czy są widoczne w środowisku produkcyjnym? |

**C. TOP 5 technical debt (do weryfikacji przez audytora)**

1. **Izolacja tenant** — brak RLS, tylko konwencja kodu → ryzyko wycieku danych
2. **File storage** — lokalny filesystem → blokuje skalowanie horyzontalne
3. **Email sync w view** — blokuje request, partial failure nieobsłużone
4. **Brak E2E testów** — krytyczne flow niepokryte automatycznie
5. **Monolity views.py** — logika domenowa w HTTP handlerach

**D. Estymacja kosztu odtworzenia (hipoteza: 21–26 m-m, do walidacji)**

Nasza wewnętrzna estymacja: 21–26 osobomiesięcy dla zespołu 3 senior developerów bez wiedzy domenowej (marża 30% na ryzyko domeny energetycznej). Prośba o niezależną walidację tej liczby i ocenę, czy estymacja uwzględnia wszystkie składniki trudności.

**E. Tenant isolation — rzeczywiste ryzyko**

Produkt adresuje rynek regulowany (spółdzielnie energetyczne z danymi pomiarowymi i finansowymi). Wyciek danych między organizacjami byłby poważnym incydentem prawnym i reputacyjnym. Prośba o ocenę: czy obecna implementacja OrgModel jest wystarczająca dla MVP z <10 SE? Czy wymaga przebudowy (PostgreSQL RLS / osobne schematy) przed skalowaniem do 50+ SE, i jaki jest realny koszt tej zmiany?

---

### 3.2 Roadmap skalowania (priorytet: wysoki)

Poniżej własna ocena kroków skalowania oparta na kodzie mastera. Prośba o weryfikację kompletności i realności estymacji.

| Etap | Skala | Co zrobić (nasze priorytety) | Czas | Koszt szac. (PLN) | Kto |
|---|---|---|---|---|---|
| **Etap 1 — Pre-scale** | 5–10 SE | (1) S3 file storage, (2) Celery email send, (3) E2E Playwright, (4) RLS lub audyt OrgModel, (5) CI deploy stage | 2–3 m-m | 40–60k | 1 senior dev |
| **Etap 2 — Growth** | 10–50 SE | (1) White-label per-SE branding, (2) API REST dla integracji OSD, (3) refaktor views.py → services, (4) multi-OSD (PGE, Energa, Enea adaptery), (5) automatyczny import email IMAP, (6) dashboard KOWR | 4–6 m-m | 80–120k | 2 senior devs |
| **Etap 3 — Scale** | 50–150 SE | (1) Kubernetes / auto-scaling, (2) PostgreSQL RLS lub schematy per-tenant, (3) REST API publiczne, (4) panel KOWR (sprawozdania roczne), (5) marketplace taryf OSD, (6) SLA 99.9% | 6–9 m-m | 150–250k | 3–4 devs + DevOps |

*Liczby do walidacji przez audytora. Prosimy o ocenę co pominęliśmy i co jest przewymiarowane.*

---

### 3.3 Wycena (priorytet: wysoki)

**A. Rynkowa cena odtworzenia**

Na podstawie własnej estymacji (21–26 m-m) i stawek rynkowych dla senior developerów w PL (15–20k PLN/m-m brutto + narzut):

- Koszt odtworzenia bez wiedzy domenowej: **1,8–2,8 mln PLN**
- Z marżą na wiedzę domenową i ryzyko compliance: **2,3–3,5 mln PLN**

Prośba o niezależną ocenę tej kalkulacji.

**B. Value-based pricing — dane rynkowe do kalkulacji**

> **Dane rynkowe do kalkulacji value-based:**
> Rynek obsługi SE w PL działa w modelu opłaty od wolumenu
> przebilansowanej energii — średnio 50 zł/MWh (proformy,
> rozliczenia, sprawozdania, obsługa prawna). Z tego ok. 30 zł/MWh
> to praca ręczna, 15–20 zł/MWh to narzut operacyjny, który
> automatyzacja realnie redukuje.
>
> Średnia SE: 1 000–10 000 MWh/rok, średnia ~2 500 MWh/rok.
> Pierwszy klient (SE Jaworze): 2 200 MWh/rok = 110 tys. zł/rok
> aktualnej obsługi rynkowej.
>
> Pipeline klienta: ~100 SE w pierwszych 6 miesiącach.
> Łączny wolumen: ~275 000 MWh/rok.
> TAM obsługi tego portfela: ~13,75 mln zł/rok.
> Oszczędność delivered przez platformę: 275 000 × 15–20 zł
> = 4,1–5,5 mln zł/rok.
>
> Koszt ręcznej obsługi 1 SE: ~0,5 etatu pracownika = ~3 000 zł/mies.
> (samo wynagrodzenie, bez marży firmy obsługującej). Przy 50 zł/MWh
> i średniej SE ~2 500 MWh/rok = ~10 400 zł/mies. obsługi rynkowej —
> z czego ~3 000 zł to czysta praca ręczna zastępowana przez platformę.
>
> Hipoteza: pricing oparty na 30–50% wartości oszczędności.
> Prośba o walidację tej proporcji.
>
> Dynamika rynku SE w Polsce:
> - 2023: 19 SE
> - styczeń 2025: 60 SE
> - grudzień 2025: >250 SE
> - luty 2026: 500 SE (potwierdzone przez KOWR)
> - marzec 2026: ~669 SE
> Wzrost >300% rocznie. TAM rośnie z każdym kwartałem.

**C. Rekomendowana struktura licencyjna (do wypełnienia przez audytora)**

| Element | Hipoteza | Rekomendacja audytora |
|---|---|---|
| Model | SaaS miesięczny od SE | |
| Opłata bazowa per SE/mies. | _______ PLN | |
| Składnik wolumenowy (per MWh) | _______ PLN | |
| Licencja optymalizatora (jednorazowo) | _______ PLN | |
| Onboarding fee (setup nowej SE) | _______ PLN | |
| SLA premium (99.9% uptime) | +_____% | |
| White-label dla resellera | _______ PLN | |
| Minimalna wartość kontraktu rocznego | _______ PLN | |

**D. Minimalna cena akceptowalna**

Prośba o ocenę: jaka jest minimalna cena licencji, która jest:
1. Uzasadniona rynkowo (weryfikowalna przez audyt NIK)
2. Daje zwrot z inwestycji w budowę w horyzoncie 3-letnim
3. Nie zaniża wartości poniżej benchmarku europejskiego (Exnaton, Sympower)

---

## 4. ANALIZA RYZYK — TOP 5

### 4.1 Key person risk [priorytet: KRYTYCZNY]

**Opis:** Sebastian Ratajczyk = jedyny developer + jedyna osoba z pełną wiedzą domenową o art. 38c + autor i maintainer silnika rozliczeniowego + autor 160 testów regresji. Żaden inny developer nie zna codebase ani specyfiki polskiego prawa energetycznego na tym poziomie szczegółowości.

**Konsekwencja:** 3-6 miesięcy przerwy w dostępności = brak możliwości utrzymania ani rozwoju. Nowy developer potrzebuje kilku miesięcy onboardingu tylko do poziomu "nie psuje produkcji".

**Prośba do audytora:**
- Ocena prawdopodobieństwa i wpływu (H/M/L)
- Jakie minimalne środki mitigation (dokumentacja, transfer wiedzy, escrow kodu) są akceptowalne dla inwestora?
- Czy struktura kontraktu powinna zawierać klauzulę o minimalnej dokumentacji jako warunek płatności?
- Jaki poziom pokrycia testami i dokumentacji traktowany jest jako "ubezpieczony" w tego typu inwestycji?

---

### 4.2 Ryzyko relacji z osobą decyzyjną [priorytet: WYSOKI]

**Opis:** Greenlab to spółka z udziałem podmiotów powiązanych z podmiotem publicznym (KOWR). Jedna osoba decyzyjna (prezes od marca 2026). Historia spółki: poprzednia ekipa przez 3 lata i 4 mln zł nie wybudowała platformy rozliczeniowej — nowa prezes zdecydowała się na zewnętrznego buildera.

**Ryzyka strukturalne:**
- Zmiana prezesa / restrukturyzacja zarządu → nowa osoba może nie rozumieć lub nie akceptować istniejącej umowy
- Zmiana priorytetów politycznych wobec sektora OZE / SE → zmiana mandatu KOWR dla Greenlab
- Procedury zakupowe spółek z udziałem skarbu państwa → potencjalny wymóg przetargu retroaktywnie
- Audyt NIK lub kontrola wewnętrzna → umowa musi być uzasadniona rynkowo i udokumentowana

**Prośba do audytora:**
1. Jakie klauzule chroniące kontrakt przy rotacji kierownictwa są standardowe w tego typu umowach?
2. Czy struktura licencyjna powinna być umową ramową z automatycznym rozszerzeniem na nowe SE — i jak to zabezpieczyć prawnie?
3. Jakie ryzyka wynikają z procedur zakupowych podmiotów z udziałem skarbu państwa (próg przetargowy, dokumentacja)?
4. Jaka struktura wynagrodzenia jest "audyt NIK safe" — udokumentowana rynkowo i proporcjonalna do dostarczanej wartości?

---

### 4.3 Ryzyko koncentracji klienta [priorytet: WYSOKI]

**Opis:** 1 klient (Greenlab) = 100% przychodu na start. Produkt zbudowany pod specyficzne potrzeby tego klienta (KOWR, procedury, format Tauron OSD).

**Czynnik łagodzący:** Rynek SE rośnie >300% rocznie (19 SE w 2023 → 669 SE w marcu 2026). Produkt jest wystarczająco generyczny (multi-tenant, parametryzowany) żeby obsługiwać inne podmioty obsługujące SE.

**Prośba do audytora:**
- Ocena H/M/L czy Greenlab może zablokować pozyskanie innych klientów (klauzule wyłączności?)
- Rekomendacja: kiedy aktywnie szukać drugiego klienta (presja na Greenlab vs dywersyfikacja)
- Czy istnieje model "białej etykiety" (white-label dla podmiotów obsługujących SE), który zmniejsza zależność?

---

### 4.4 Tenant isolation / security [priorytet: WYSOKI]

**Opis:** Multi-tenant izolacja oparta na dyscyplinie programistycznej (`OrgModel.for_org()`), nie mechanizmie automatycznym (PostgreSQL RLS, osobne schematy). Szczegóły w sekcji T1 INWENTARYZACJA_PRODUKTU.md.

**Kontekst regulacyjny:** Dane SE zawierają dane pomiarowe (profil energetyczny) i finansowe (kwoty rozliczeń) — potencjalnie objęte RODO jako dane wrażliwe w kontekście identyfikacji zachowań.

**Prośba do audytora:**
- Ocena: czy obecna implementacja jest wystarczająca dla MVP z <10 SE klientem enterprise?
- Kiedy (przy jakiej skali lub typie klienta) staje się blokerem i wymaga przebudowy na RLS?
- Szacunek kosztu migracji na PostgreSQL RLS (czy to tygodnie czy miesiące pracy)?

---

### 4.5 Ryzyko wejścia globalnego gracza [priorytet: ŚREDNI]

**Opis:** Exnaton (Szwajcaria, Series A, ~15 mln CHF finansowania) buduje platformę dla spółdzielni energetycznych na rynkach europejskich. Operuje w CH, DE, AT. Nie ma aktywności w PL (brak polskiej lokalizacji, brak art. 38c compliance).

**Bariera wejścia:** Polskie prawo energetyczne (art. 38c–38e ustawy OZE + Dz.U. 2022 poz. 703) jest specyficzne dla PL — wymaga 6–12 miesięcy pracy prawno-technicznej dla gracza nieobecnego w PL. Rynek jest stosunkowo mały w skali europejskiej — atrakcyjny dla lokalnego gracza, mało atrakcyjny dla venture-backed startupu celującego w skalę 10x.

**Prośba do audytora:**
- Ocena H/M/L w horyzoncie 2-letnim
- Czy istniejący pipeline (~100 SE w 6m) i lock-in przez integrację z OSD jest wystarczającą obroną?
- Czy warto budować moat przez certyfikacje/partnerstwa z OSD (Tauron, PGE, Energa)?

---

## 5. DELIVERABLE

Proponowany zakres dokumentów końcowych audytu:

| # | Dokument | Zawartość | Maks. stron |
|---|---|---|---|
| 1 | **Raport techniczny** | Ocena architektury, jakości kodu, tenant isolation, security review | 4 |
| 2 | **Production-readiness checklist** | Status każdego punktu (OK/Issue/Blocker) z rekomendacją | 2 |
| 3 | **Roadmap skalowania** | Zwalidowane 3 etapy z kosztami i priorytetami | 2 |
| 4 | **Wycena** | Odtworzeniowa + value-based + rekomendowana struktura licencji | 2 |
| 5 | **Risk register** | TOP 5 ryzyk z oceną H/M/L i rekomendacją mitigation | 2 |
| 6 | **Executive summary** | 1 strona dla inwestora — verdict + kluczowe liczby + rekomendacja | 1 |
| **RAZEM** | | | **~13 stron** |

---

## UWAGI LOGISTYCZNE

- **Zakres kodu:** branch `master`, tag `v0.3.0-etap2-stable` (aktualny HEAD zawiera 18 dodatkowych commitów UI — do uzgodnienia czy audyt obejmuje aktualny HEAD)
- **Branch `dev`** (prace rozwojowe w toku) nie wchodzi w zakres niniejszego audytu
- **Dostęp:** read-only repo + sesja demo (umówić termin)
- **Materiały wstępne:** niniejszy dokument + `INWENTARYZACJA_PRODUKTU.md`
- **Ograniczenie:** ocena architektury i wzorców, nie line-by-line code review
- **Oczekiwany czas audytu:** 3–5 dni roboczych
