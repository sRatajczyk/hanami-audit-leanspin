> ## STATUS DOKUMENTU
> Niniejszy dokument jest HIPOTEZĄ założyciela — wstępną propozycją
> struktury wydatków do walidacji przez audytora zewnętrznego (LeanSpin).
> Wszystkie kwoty są szacunkowe i wymagają niezależnej weryfikacji.
> Nie jest to oferta ani zobowiązanie finansowe.

---

# USE OF FUNDS — SE Platform (Hanami)
**Wersja:** hipoteza robocza, 2026-05-28
**Horyzont:** 12 miesięcy operacyjnych od uruchomienia finansowania

---

## 1. PODSUMOWANIE EXECUTIVE

**Łączna kwota finansowania: 2 300 000 PLN (ok. 530 000 EUR)**

Cel: przejście od MVP z 1 klientem produkcyjnym (SE Jaworze, 18 PPE) do platformy obsługującej 100–150 aktywnych SE, z architekturą i infrastrukturą gotową na 1 000+ tenantów. Plan dzieli się na trzy etapy o różnym charakterze: Etap 1 to hardening fundamentów (3 mies.), Etap 2 to pedał gazu i onboarding 100 SE z pipeline Greenlab (3 mies.), Etap 3 to stabilizacja i profesjonalizacja pod exit lub kolejną rundę (6 mies.). Największa pozycja budżetowa — 63% łącznych wydatków — to koszt zespołu. Rynek SE w Polsce rośnie ponad 300% rocznie (19 SE w 2023 → 669 SE w marcu 2026); produkt musi być gotowy na tę skalę od pierwszego dnia Etapu 2, nie dopiero po nim.

---

## 2. ETAP 1: PRODUCTION-READY + HARDENING NA 100 SE (miesiące 1–3)

**Cel:** Z MVP przejść do produkcji obsługującej pierwsze 5–10 SE z architekturą GOTOWĄ na 100 SE od dnia 1 Etapu 2. To nie jest "pilotaż na 5 SE" — to budowa fundamentu pod 100. Wszystkie pozycje technical debt zidentyfikowane w INWENTARYZACJA_PRODUKTU.md (T1–T10) muszą być zamknięte zanim zacznie się onboarding kolejnych klientów.

### Zespół — Etap 1

| Rola | Wymiar | Stawka mies. | 3 mies. |
|---|---|---|---|
| Sebastian Ratajczyk (CTO / Product Owner) | pełen etat | 30 000 zł B2B | 90 000 zł |
| Senior Python/Django Developer | pełen etat | 26 000 zł B2B | 78 000 zł |
| DevOps / SysAdmin | part-time (50%) | 15 000 zł B2B | 45 000 zł |
| **Razem zespół E1** | | | **213 000 zł** |

*Stawki B2B na poziomie rynku PL 2026 dla seniora z Django i doświadczeniem w SaaS. Niepewność: jeśli rekrutacja zajmie >4 tygodnie, koszty E1 rosną o ~26 000 zł za każdy miesiąc opóźnienia.*

### Hardening produktu — Etap 1

Wszystkie zadania z TECHNICAL DEBT (INWENTARYZACJA_PRODUKTU.md, §4). Stawka: 1 750 zł/dzień roboczy senior dev. Ważne: zadania wyceniane na skalę 100 SE — nie 5 SE.

| Task | Priorytet | Osobodni | Koszt |
|---|---|---|---|
| **T1 — Tenant isolation (PostgreSQL RLS)** | 🔴 Bloker | 12 | 21 000 zł |
| **T4 — Migracja file storage → S3** | 🔴 Bloker | 5 | 8 750 zł |
| **T3 — Email bulk send → Celery task** | 🟠 Wysoki | 3 | 5 250 zł |
| **T6 — E2E testy Playwright (5 critical flows)** | 🟠 Wysoki | 8 | 14 000 zł |
| Parsery multi-OSD (PGE, ENEA, Energa adaptery) | 🟠 Wysoki | 15 | 26 250 zł |
| Automatyczny IMAP pipeline per SE | 🟠 Wysoki | 8 | 14 000 zł |
| Automatyzacja onboardingu SE (<1 dzień roboczy) | 🟠 Wysoki | 10 | 17 500 zł |
| Per-tenant branding (logo, kolory, domena) | 🟡 Średni | 6 | 10 500 zł |
| **T2 — Refaktor views.py → warstwa serwisów** | 🟡 Średni | 10 | 17 500 zł |
| CI/CD deploy pipeline (staging → prod automatyczny) | 🟡 Średni | 5 | 8 750 zł |
| Panel zarządzania użytkownikami (100 SE × 50 os.) | 🟡 Średni | 8 | 14 000 zł |
| Security hardening (rate limiting, headers) | 🟡 Średni | 4 | 7 000 zł |
| Aktywacja monitoringu (Sentry DSN, alerty) | 🟡 Średni | 2 | 3 500 zł |
| Load testing baseline + optymalizacja | 🟡 Średni | 3 | 5 250 zł |
| **Razem hardening E1** | | **99 dni** | **173 250 zł** |

*Niepewność: T1 (RLS) może wymagać więcej czasu jeśli zdecydujemy się na django-tenants (osobne schematy DB) zamiast RLS — oszacowanie 12 dni może być niedoszacowane o 30–50%. To jest decyzja architektoniczna do podjęcia na początku Etapu 1.*

### Infrastruktura — Etap 1 (wymiarowana na 100 SE od dnia 1)

| Pozycja | Koszt mies. | 3 mies. |
|---|---|---|
| Serwer cloud (AWS/DigitalOcean — app + Celery worker, 8–16 vCPU) | 1 500 zł | 4 500 zł |
| Managed PostgreSQL (16, 8 GB RAM, daily backup) | 800 zł | 2 400 zł |
| Redis managed (persistent, HA) | 400 zł | 1 200 zł |
| S3 storage (pliki OSD, PDFy) | 200 zł | 600 zł |
| Mailgun (email transakcyjny, 50 000 mail/mies.) | 400 zł | 1 200 zł |
| Backup automatyczny (DB + storage) | 300 zł | 900 zł |
| Monitoring (Sentry Pro + uptime checks) | 400 zł | 1 200 zł |
| Domena + SSL + DNS (jednorazowo) | — | 500 zł |
| **Razem infra E1** | **4 000 zł** | **12 500 zł** |

*Uwaga: infra w Etapie 1 jest celowo wymiarowana na 100 SE z góry. Koszt skokowy przy przejściu E1 → E2 powinien być minimalny.*

### Prawnik — Etap 1

| Temat | Szac. godz. | Stawka | Koszt |
|---|---|---|---|
| Konsultacja IP/IT — struktura umowy licencyjnej SaaS | 5h | 550 zł/h | 2 750 zł |
| Przygotowanie umowy licencyjnej z Greenlab/RSSI | 12h | 550 zł/h | 6 600 zł |
| NDA + umowy z podwykonawcami (dev, DevOps) | 4h | 550 zł/h | 2 200 zł |
| Bieżące konsultacje (3 mies.) | 2h × 3 | 550 zł/h | 3 300 zł |
| **Razem prawnik E1** | | | **14 850 zł** |

### Audyt zewnętrzny — Etap 1

| Pozycja | Koszt |
|---|---|
| Audyt LeanSpin (techniczny + biznesowy) | 25 000 zł |
| Security audit / pen-test (przed onboardingiem 100 SE) | 15 000 zł |
| **Razem audyt E1** | **40 000 zł** |

### SUMA ETAP 1

| Kategoria | Kwota |
|---|---|
| Zespół | 213 000 zł |
| Hardening produktu | 173 250 zł |
| Infrastruktura | 12 500 zł |
| Prawnik | 14 850 zł |
| Audyt zewnętrzny | 40 000 zł |
| **SUMA ETAP 1** | **453 600 zł** |

---

## 3. ETAP 2: SKALOWANIE DO 100 SE (miesiące 4–6)

**Cel:** Onboarding 100 SE z pipeline Greenlab (40 potwierdzonych z KOWR + 60 deklaracji). Platforma musi działać stabilnie przy 100 tenantach × 20–50 członków × miesięczne rozliczenia × proformy. Fundament z Etapu 1 musi unieść tę skalę — jeśli nie uniósł, Etap 2 się nie zaczyna.

### Zespół — Etap 2

| Rola | Wymiar | Stawka mies. | 3 mies. |
|---|---|---|---|
| Sebastian (CTO/PO) | pełen etat | 30 000 zł | 90 000 zł |
| Senior Python/Django Developer | pełen etat | 26 000 zł | 78 000 zł |
| Drugi developer (mid/senior, nowy) | pełen etat | 20 000 zł | 60 000 zł |
| DevOps (rozbudowa z part-time na pełen etat) | pełen etat | 22 000 zł | 66 000 zł |
| Support / Customer Success | pełen etat | 13 000 zł | 39 000 zł |
| **Razem zespół E2** | | | **333 000 zł** |

*Drugi developer i CS muszą być zatrudnieni w połowie Etapu 1, żeby zdążyć z onboardingiem. Rekrutacja = osobne ryzyko kosztowe (headhunter ~10–15% rocznego wynagrodzenia, ~24 000 zł na 2 osoby).*

### Rozwój produktu — Etap 2

| Task | Osobodni | Stawka dzienna | Koszt |
|---|---|---|---|
| REST API (DRF) dla integracji z systemami OSD | 10 | 1 750 zł | 17 500 zł |
| Panel superadmin — metryki SaaS (MRR, churn, status SE) | 8 | 1 750 zł | 14 000 zł |
| Load testing przy 100 SE + optymalizacja bottlenecków | 5 | 1 750 zł | 8 750 zł |
| Raportowanie KOWR (sprawozdania roczne, formularze) | 10 | 1 750 zł | 17 500 zł |
| Self-service onboarding — formularz zamiast procesu 7-krokowego | 8 | 1 750 zł | 14 000 zł |
| **Razem dev E2** | **41 dni** | | **71 750 zł** |

### Infrastruktura — Etap 2

| Pozycja | Koszt mies. | 3 mies. |
|---|---|---|
| Cloud (app + workers, skalowanie pod 100 SE) | 2 500 zł | 7 500 zł |
| PostgreSQL managed (+ read replica, 100 tenantów) | 1 500 zł | 4 500 zł |
| Redis managed (scaled) | 600 zł | 1 800 zł |
| S3 (rosnące wolumeny OSD) | 500 zł | 1 500 zł |
| Mailgun (wyższy tier — proformy 100 SE × 50 os.) | 800 zł | 2 400 zł |
| Monitoring (upgrade) | 600 zł | 1 800 zł |
| **Razem infra E2** | **6 500 zł** | **19 500 zł** |

### Marketing i sprzedaż — Etap 2

| Pozycja | Koszt |
|---|---|
| Case study SE Jaworze (design + copywriting) | 5 000 zł |
| Strona www produktu (projekt + wdrożenie) | 12 000 zł |
| Materiały sprzedażowe (deck, one-pager) | 4 000 zł |
| **Razem marketing E2** | **21 000 zł** |

*Niepewność: jeśli 60 SE spoza KOWR wymaga aktywnego procesu sprzedaży (nie samo-onboarding), pojawi się potrzeba handlowca lub partnership managera. Kosztu tego nie ujęto — to decyzja do Etapu 1.*

### SUMA ETAP 2

| Kategoria | Kwota |
|---|---|
| Zespół | 333 000 zł |
| Rozwój produktu | 71 750 zł |
| Infrastruktura | 19 500 zł |
| Marketing / sprzedaż | 21 000 zł |
| **SUMA ETAP 2** | **445 250 zł** |

---

## 4. ETAP 3: INFRASTRUKTURA 1000+ I STABILIZACJA (miesiące 7–12)

**Cel:** Platforma obsługuje 100–150 aktywnych SE stabilnie. Infrastruktura gotowa na 1 000+ tenantów. Przygotowanie do exitu lub kolejnej rundy. Faza profesjonalizacji — product management, SLA, compliance, DR.

### Zespół — Etap 3

| Rola | Wymiar | Stawka mies. | 6 mies. |
|---|---|---|---|
| Sebastian (CTO/PO) | pełen etat | 30 000 zł | 180 000 zł |
| Senior Python/Django Developer | pełen etat | 26 000 zł | 156 000 zł |
| Drugi developer (mid/senior) | pełen etat | 20 000 zł | 120 000 zł |
| DevOps | pełen etat | 22 000 zł | 132 000 zł |
| Support / Customer Success | pełen etat | 13 000 zł | 78 000 zł |
| Product Manager (od m-ca 9, odciążenie Sebastiana) | pełen etat, 4 mies. | 22 000 zł | 88 000 zł |
| **Razem zespół E3** | | | **754 000 zł** |

### Rozwój produktu — Etap 3

| Task | Osobodni | Stawka dzienna | Koszt |
|---|---|---|---|
| Full self-service onboarding (bez udziału Greenlab) | 10 | 1 750 zł | 17 500 zł |
| Automatyczne aktualizacje taryf dystrybucyjnych (roczne) | 8 | 1 750 zł | 14 000 zł |
| GDPR compliance pełne (privacy by design, data residency) | 10 | 1 750 zł | 17 500 zł |
| SLA 99.9% — redundancja, failover, health checks | 8 | 1 750 zł | 14 000 zł |
| Rozbudowa marketplace integracji OSD | 15 | 1 750 zł | 26 250 zł |
| **Razem dev E3** | **51 dni** | | **89 250 zł** |

### Infrastruktura (1000+ ready) — Etap 3

| Pozycja | Koszt mies. | 6 mies. |
|---|---|---|
| Infrastruktura 1000+ (upgrade, tenant DB isolation) | 3 000 zł | 18 000 zł |
| Auto-scaling Celery workers | 1 500 zł | 9 000 zł |
| CDN (pliki statyczne, PDFy) | 400 zł | 2 400 zł |
| Disaster recovery (plan + implementacja, jednorazowo) | — | 5 000 zł |
| Penetration test pełny (przed exit / skalowaniem) | — | 20 000 zł |
| **Razem infra E3** | **4 900 zł** | **54 400 zł** |

### Prawnik — Etap 3

| Pozycja | Koszt |
|---|---|
| Bieżące konsultacje (6 mies. × 2h × 550 zł) | 6 600 zł |
| GDPR — konsultacja DPO | 4 000 zł |
| Rejestracja znaku towarowego Hanami | 3 000 zł |
| **Razem prawnik E3** | **13 600 zł** |

### SUMA ETAP 3

| Kategoria | Kwota |
|---|---|
| Zespół | 754 000 zł |
| Rozwój produktu | 89 250 zł |
| Infrastruktura | 54 400 zł |
| Prawnik | 13 600 zł |
| **SUMA ETAP 3** | **911 250 zł** |

---

## 5. HORYZONT 12–24 MIESIĄCE

Po osiągnięciu 100–150 aktywnych SE i infrastruktury 1 000+ otwierają się dwie ścieżki: exit (sprzedaż platformy lub licencji strategicznemu nabywcy — OSD, firma obsługująca SE w skali ogólnopolskiej lub europejskiej) albo dalszy rozwój (nowe ścieżki revenue: Hanami Connects, panel operacyjny dla KOWR, ekspansja do Czech/Słowacji gdzie podobne regulacje wchodzą w życie). Szczegółowy plan tej fazy zostanie przygotowany po stabilizacji Etapu 3 — jego zawartość zależy od dynamiki rynku i tempa onboardingu SE w Etapie 2.

---

## 6. TABELA ZBIORCZA

| Etap | Okres | Zespół | Dev / Hardening | Infra | Prawnik / Audyt | Inne | **SUMA** |
|---|---|---|---|---|---|---|---|
| 1 | M1–3 | 213 000 | 173 250 | 12 500 | 54 850 | — | **453 600** |
| 2 | M4–6 | 333 000 | 71 750 | 19 500 | — | 21 000 | **445 250** |
| 3 | M7–12 | 754 000 | 89 250 | 54 400 | 13 600 | — | **911 250** |
| **RAZEM** | **12 mies.** | **1 300 000** | **334 250** | **86 400** | **68 450** | **21 000** | **1 810 100** |

*(Kolumna "Prawnik / Audyt" w E1 łączy: prawnik 14 850 + audyt zewnętrzny 40 000 = 54 850)*

**Kwoty zaokrąglone do celów planowania:**

| | Kwota |
|---|---|
| Suma bez rezerwy | 1 810 000 zł |
| Rezerwa projektowa 25% | 453 000 zł |
| **Łączny ask** | **2 263 000 zł ≈ 2 300 000 zł** |

---

## 7. RYZYKO NIEDOSZACOWANIA

Standardowa rezerwa projektowa wynosi 20–30% łącznej kwoty. Przyjęto 25% (~453 000 zł), co daje łączny ask **2 300 000 PLN**. Uzasadnienie: nowy, szybko zmieniający się rynek regulowany (zmiany w Dz.U. mogą wymagać zmian w silniku rozliczeniowym), nieprzewidywalne tempo onboardingu SE (pipeline może być szybszy lub wolniejszy niż deklaracje), ryzyko rekrutacyjne (opóźnienie w pozyskaniu seniora dev przesuwa Etap 1 o 4–6 tygodni), ryzyko architektoniczne przy T1 (decyzja RLS vs django-tenants może podwoić szacowany czas). Rezerwa nie obejmuje kosztów exitu ani kolejnej rundy — te są poza horyzontem 12-miesięcznym.

*Największe źródło niepewności: stawki develope rów B2B w PL 2026 mogą być wyższe niż zakładane o 10–20% przy presji na rynek AI/energetyka. Jeśli rekrutacja wymaga agencji — dodatkowe 24 000–30 000 zł na dwie osoby.*
