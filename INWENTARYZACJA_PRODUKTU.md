> **STAN PRODUKTU:** branch `master`, HEAD `ff4208f` (2026-05-28)
> Tag `v0.3.0-etap2-stable` wskazuje na commit `4c7f03e` — 18 commitów wstecz od HEAD.
> Niniejsza inwentaryzacja opisuje **aktualny HEAD mastera** — stan pełniejszy niż tag.
> Branch `dev` (prace rozwojowe w toku) NIE jest uwzględniony w niniejszej inwentaryzacji.
> Data generowania: 2026-05-28

---

# INWENTARYZACJA PRODUKTU — SE Platform (Hanami)

## 0. METODOLOGIA

Inwentaryzacja wygenerowana na podstawie:
- `git log` i `git diff` na branchu master
- `find apps scripts -name "*.py" | xargs wc -l` — pomiar LOC per moduł
- `pytest --collect-only -q` + `pytest -q` — aktualne wyniki testów (uruchomione dziś)
- ręczna analiza plików `apps/*/urls.py`, `apps/*/models.py`, `apps/*/views.py`
- przegląd `requirements-django.txt`, `docker-compose*.yml`, `.github/workflows/ci.yml`

**Pominięte katalogi:**
- `.claude/` — worktree i cache AI tooling, nie kod produktu
- `enso/` — legacy settings z poprzedniej architektury (FastAPI → Django), zawiera `README_LEGACY.md`
- `app/` — pozostałość FastAPI (router `se_router.py`, ~1 132 LOC), nieaktywny kod
- `docs/` — dokumentacja strategiczna i prawna
- `scripts/ingest/` — parser emailów IMAP, niezintegrowany z Django
- `scripts/domain/`, `scripts/simulation/`, `scripts/validate/`, `scripts/ai/` — utilities/eksperymenty pomocnicze

---

## 1. STAN PRODUKTU NA DZIŚ (2026-05-28)

| Moduł | Status | Krótki opis | Pliki / lokalizacja |
|---|---|---|---|
| **Silnik rozliczeniowy** | ✅ Produkcja | Pełna implementacja art. 38c–38e i Dz.U. 2022 poz. 703. Bateria wirtualna, saldo WI, wytwórcy/odbiorcy, korekty depozytowe. Zweryfikowany 100% vs Excel klienta SE Jaworze. | `scripts/settlement/internal_engine.py` (552 LOC) |
| **Adapter danych OSD** | ✅ Produkcja (Tauron) | Parser CSV Tauron z registry pattern gotowym do rozszerzenia. Multi-file + ZIP upload do 100 MB. AI fallback dla nieznanych formatów. | `scripts/etl/parse_osd.py` (476 LOC), `apps/readings/` (838 LOC) |
| **Optymalizator SE** | ✅ Produkcja | Oblicza optymalną moc OZE dla nowych SE. Multi-profil konsumpcji (max 3 profile, merge intersection), porównanie taryf B/C, raport PDF. | `scripts/se_optimizer/` (1 870 LOC), `apps/optimizer/` (1 633 LOC) |
| **Fakturowanie / proformy** | ✅ Produkcja | Generowanie PDF (WeasyPrint), per-proforma workflow zatwierdzania, bulk send Mailgun, ZIP eksport. Idempotentne generowanie. | `apps/documents/` (656 LOC) |
| **Panel admina SE** | ✅ Produkcja | Dashboard ECharts (Sankey energii, donut udziałów, historia 12m), widok godzinowy 24h, analiza roczna, oszczędności KPI. | `apps/dashboard/` (1 983 LOC) |
| **Panel członka SE** | ✅ Produkcja | Własne oszczędności, profil energetyczny, lista dokumentów/proform, historia rozliczeń. Osobne URL `/me/*`. | `apps/members/` (824 LOC), ścieżki `/me/` |
| **Workflow miesięczny** | ✅ Produkcja | 10-krokowy checklist (import → rozliczenie → proformy → wysyłka). Auto-detekcja 3 kroków. Sekcja roczna z countdown KOWR. | `apps/workflow/` (341 LOC) |
| **Rozliczenia SE** | ✅ Produkcja | Uruchamianie async (Celery), historia, walidacja, what-if analysis, widok szczegółowy z polling UI. | `apps/settlement/` (993 LOC) |
| **Multi-tenant / onboarding** | ✅ Produkcja | Wizard 3-krokowy: dane SE → PPE/członkowie → admin SE. OrgModel z wymuszeniem izolacji per-query. | `apps/wizard/` (348 LOC), `apps/accounts/models.py` |
| **Uprawnienia RBAC** | ✅ Produkcja | 5 ról z hierarchią. Zaproszenia tokenem (24h TTL). Dekorator `require_role`. Audit log. | `apps/accounts/` (1 161 LOC) |
| **Parametry regulacyjne** | ✅ Produkcja | django-constance: Wi, ceny energii, VAT, koef. KOWR (bez restartu). OrgSettings: per-SE override cen + szablony email. | `apps/settings_se/` (566 LOC) |
| **Baza taryf OSD** | ✅ Produkcja | Taryfy Tauron/PGE: B11, B12, C11 z aktywnym rokiem. Seed idempotent. | `scripts/se_optimizer/tariffs.py` (676 LOC) |
| **Portfolio SE** | ✅ Produkcja | Przegląd wszystkich SE dla superadmin/kowr_admin. Status rozliczeń, wskaźnik progresu workflow per SE. | `apps/portfolio/` (171 LOC) |
| **Generowanie raportów PDF** | ✅ Produkcja | Raport admina + 18 faktur proforma per SE. Jinja2 templates, WeasyPrint. | `scripts/report/` (1 047 LOC) |
| **White-label / branding** | ⚠️ Częściowe | Logo i nazwa produktu (Hanami) konfigurowalne. Brak per-SE brandingu dla klientów końcowych. | `static/images/`, `templates/base.html` |
| **API publiczne** | ❌ Brak | Brak REST/GraphQL API. Są wewnętrzne AJAX endpoints (daily charts, live, whatif, member daily). | — |
| **Parser email IMAP** | ⚠️ Niezintegrowany | Skrypt parsuje attachmenty OSD z maila. Nie podłączony do Django — wymaga uruchamiania ręcznie. | `scripts/ingest/` |
| **Push/SMS powiadomienia** | ❌ Brak | Alerty widoczne w panelu. Brak push/SMS. Email tylko dla proform. | `apps/alerts/` |

---

## 2. ARCHITEKTURA TECHNICZNA

**Stack:**

| Warstwa | Technologia | Wersja |
|---|---|---|
| Backend framework | Django | 5.x |
| Baza danych | PostgreSQL | 16 |
| Cache / kolejka wiadomości | Redis | 7 |
| Task queue | Celery + django-celery-beat | 5.3 |
| Auth | django-allauth | 0.61 |
| PDF rendering | WeasyPrint | 62.3 |
| Email transakcyjny | Mailgun via django-anymail | 15.x |
| Parametry runtime | django-constance | 4.3 |
| Rate limiting | django-ratelimit | 4.1 |
| Audit log | django-auditlog | 3.4 |
| Wykresy frontend | ECharts v5 (lokalny plik statyczny) | 5.5.1 |
| Data processing | NumPy, pandas, openpyxl | — |
| CI/CD | GitHub Actions | — |
| Konteneryzacja | Docker Compose (dev + prod wariant) | — |
| Monitoring | Sentry SDK | 2.x |

**Struktura repozytoriów:**

```
enso/         — Django project settings (base/local/prod) + celery.py
apps/         — 12 aktywnych Django applications
scripts/      — domain logic poza Django DI (settlement, optimizer, etl, report, tests)
templates/    — globalne szablony HTML (base.html, nav, error pages)
static/       — CSS design system, JS (ECharts), obrazy
```

**Multi-tenant:**

Każda instancja SE to obiekt `Organization`. Izolacja realizowana przez:
- `OrgModel` — abstrakcyjna klasa bazowa dla wszystkich modeli z danymi SE
- `OrgManager.for_org(org)` — każde query musi przejść przez ten manager
- `NotImplementedError` przy `Model.objects.all()` bez scope'u (wymuszenie runtime)
- Middleware `OrganizationMiddleware` — ustawia `request.organization` z sesji użytkownika

Schemat bazy: **single database**, izolacja na poziomie wierszy (`organization_id` FK na każdym modelu). Brak PostgreSQL row-level security ani osobnych schematów.

**Hierarchia ról:**

```
superadmin           → pełny dostęp, wszystkie SE
  └─ kowr_admin      → własne SE-children (relacja parent_org)
       └─ admin_se   → zarządza jedną SE
            └─ se_staff   → operacyjny dostęp do danych
                 └─ se_member   → tylko własne dane (panel /me/)
```

**Workflow onboardingu nowej SE:**

1. `/wizard/onboarding/` → krok 1: nazwa, slug, OSD, adres
2. Krok 2: import listy PPE + przypisanie ról prosumer/consumer
3. Krok 3: konto admin_se + wysyłka zaproszenia tokenem

**Dane regulacyjne:**

Algorytm rozliczeniowy implementuje Dz.U. 2022 poz. 703, §4–§8 (podział energii zbiorowej, bateria wirtualna, korekta WI, saldo depozytowe). Parametry konfigurowane przez django-constance (bez restartu serwera): wskaźnik korekcyjny Wi, ceny energii (wytwórca/odbiorca), VAT, próg autokonsumpcji 70% (KOWR). Każda SE może mieć własny override cen.

---

## 3. KLUCZOWE LICZBY (KOD NA MASTERZE)

**LOC per moduł (Python, `wc -l`, wynik z dziś):**

| Moduł | LOC Python |
|---|---|
| apps/dashboard | 1 983 |
| apps/optimizer | 1 633 |
| apps/accounts | 1 161 |
| apps/settlement | 993 |
| apps/readings | 838 |
| apps/members | 824 |
| apps/documents | 656 |
| apps/settings_se | 566 |
| apps/wizard | 348 |
| apps/workflow | 341 |
| apps/portfolio | 171 |
| **Razem apps/** | **~9 514** |
| scripts/tests | 3 660 |
| scripts/se_optimizer | 1 870 |
| scripts/report | 1 047 |
| scripts/settlement | 1 001 |
| scripts/etl | 476 |
| **Razem scripts/** | **~8 054** |
| **ŁĄCZNIE Python** | **~17 568** |

*Nie uwzględnia: HTML templates ~3 500–4 000 LOC, CSS ~800 LOC, legacy `app/` FastAPI ~1 132 LOC (nieaktywny).*

**Testy (wynik z dziś, master):**

```
$ pytest scripts/tests/ -q --ignore=scripts/tests/test_app.py
158 passed, 1 skipped, 1 xfailed — 160 zebranych — czas: ~51s
```

| Plik testów | Liczba testów | Zakres |
|---|---|---|
| test_settlement_regression.py | 15 | regresja silnika vs dane Excel klienta |
| test_se_optimizer.py | 22 | optymalizator — profil, taryfy, ekonomia |
| test_invitations.py | 12 | RBAC, zaproszenia, izolacja |
| test_multitenancy.py | 5 | izolacja multi-tenant, URL manipulation |
| test_economics.py | 6 | kalkulacje finansowe |
| test_workflow.py | 8 | workflow checklist |
| test_wizard.py | 6 | onboarding |
| test_adapters.py | 5 | parser OSD Tauron |
| test_proforma_generation.py | 4 | generowanie proform, idempotencja |
| test_tariff_db.py | 6 | baza taryf |
| test_orgsettings.py | 5 | OrgSettings, price override |
| test_user_admin.py | 5 | zarządzanie użytkownikami |
| test_constance.py | 5 | parametry runtime |
| test_portfolio.py | 5 | portfolio view |
| test_security_blockers.py | 2 | signup zablokowany, invitation auth |
| test_ingest.py | 7 | parser, path traversal, IMAP injection |
| pozostałe | ~7 | config, ops hardening, password reset |

**HTTP endpoints:** ~57 URL patterns (wliczając AJAX i akcje POST)

**Modele Django (aktywne, 18):**
`Organization`, `OrgSettings`, `User` (custom), `Invitation`, `Member`, `PPE`, `Membership`, `Measurement`, `MeasurementCorrection`, `ImportBatch`, `SettlementRun`, `SettlementResult`, `MonthlyWorkflow`, `WorkflowStep`, `Tariff`, `Proforma`, `Alert`, `EnergyMatrix`

**CI/CD:** GitHub Actions przy każdym push na `dev` i `master` — PostgreSQL 16 + Redis 7 jako serwisy testowe. Brak automatycznego deploy (brak stages staging/prod w pipeline).

---

## 4. TECHNICAL DEBT

Poniżej lista problemów, które audytor zidentyfikuje samodzielnie. Lepiej wymienić wprost.

**T1 — Izolacja tenant: dyscyplina kodu, nie mechanizm bazy [ryzyko: WYSOKIE]**
`OrgModel` wymusza `.for_org()` przez `NotImplementedError`, ale jest to kontrola runtime, nie enforce na poziomie PostgreSQL. Jeden niezauważony `Model.objects.filter(...)` bez scope'u daje wyciek danych między organizacjami. Brak row-level security, brak osobnych schematów. Przy skalowaniu zespołu to ryzyko rośnie — jeden nowy developer może zepsuć izolację nie wiedząc o konwencji.

**T2 — Monolity widokowe [ryzyko: ŚREDNIE]**
`apps/optimizer/views.py` — 1 619 LOC. `apps/dashboard/views.py` — 1 062 LOC. Logika domenowa, przygotowanie danych i renderowanie w jednym pliku. Brak warstwy serwisowej / use cases. Trudne w testowaniu bez HTTP requestu.

**T3 — Email wysyłany synchronicznie w widoku [ryzyko: ŚREDNIE]**
`proforma_send_bulk` wywołuje `EmailMessage.send()` w pętli w widoku Django. Przy 100+ odbiorców grozi timeout HTTP lub partial failure bez informacji dla użytkownika. Powinno być w Celery task z progresem.

**T4 — File storage: lokalny filesystem [ryzyko: WYSOKIE przy skalowaniu]**
`default_storage.save()` zapisuje pliki na lokalny dysk kontenera. Multi-instance deployment (load balancer, skalowanie horyzontalne) — jeden node nie widzi plików drugiego. Wymaga migracji na S3/GCS przed jakimkolwiek skalowaniem.

**T5 — Legacy kod w repozytorium [ryzyko: NISKIE operacyjnie, ŚREDNIE reputacyjnie]**
`app/` (FastAPI, ~1 132 LOC) i `enso/settings.py` (komentarz o rotowanym SECRET_KEY) w repozytorium. Zaśmiecają strukturę, mylą nowych developerów i audytorów.

**T6 — Brak E2E testów [ryzyko: ŚREDNIE]**
Playwright w requirements, ale brak działających testów E2E. Krytyczne flow (login → import danych → rozliczenie → proforma → send) nie jest automatycznie testowane end-to-end. Testy jednostkowe nie zastępują integracji.

**T7 — `scripts/` poza Django DI [ryzyko: NISKIE operacyjnie]**
Domain logic w `scripts/` to czyste moduły Python — brak dependency injection, brak bezpośredniego dostępu do ORM. Warstwa adaptera w `apps/settlement/` bridguje oba światy. Architektura działa, ale utrudnia rozbudowę (np. logowanie do bazy z `internal_engine.py` wymaga osobnej warstwy).

**T8 — Brak historii konfiguracji taryf [ryzyko: NISKIE]**
Zmiana taryfy dla PPE (B11 → C11) nie ma historii zmian w DB — nie wiadomo które rozliczenie historyczne używało której taryfy.

**T9 — Monitoring: skonfigurowany, niezweryfikowany [ryzyko: NISKIE]**
`sentry-sdk` w requirements, ale brak potwierdzenia że DSN jest ustawiony i alerty trafiają do Sentry w środowisku produkcyjnym.

**T10 — Rate limiting: niepełne pokrycie [ryzyko: NISKIE]**
Rate limiting jest na upload pliku i zaproszeniach. Bulk email send, what-if, download ZIP — bez limitu. Celowo nie zaznaczono jako krytyczny.

---

## 5. ESTYMACJA ODTWORZENIOWA

Założenie: **3 senior developerów** (Django + data engineering + frontend), **bez wiedzy domenowej** o art. 38c ustawy OZE.

| Komponent | Czas (m-m) | Uwagi |
|---|---|---|
| Silnik rozliczeniowy (art. 38c compliance) | 3–4 | Najtrudniejszy element — wymaga analizy prawnej + weryfikacji vs dane OSD |
| Optymalizator SE (profile, taryfy, PDF) | 2–3 | Skomplikowana matematyka energetyczna + baza taryf |
| Parser OSD (adaptery, AI fallback, ZIP) | 1–1,5 | Format Tauron + registry pattern |
| Multi-tenant Django + RBAC + invitations | 1,5–2 | OrgModel, hierarchia ról, tokeny |
| Panel admina (UI/UX, ECharts, 12+ widoków) | 2,5–3 | Dużo widoków, wykresy, responsywność |
| Panel członka SE | 1 | Prostsze widoki, read-only |
| Workflow miesięczny (10 kroków, auto-det.) | 1 | |
| Fakturowanie + PDF + bulk email | 1,5 | WeasyPrint, Mailgun, workflow zatwierdzania |
| Generowanie raportów | 1 | Jinja2 + template design |
| Onboarding wizard | 0,5 | 3 kroki |
| CI/CD, Docker, konfiguracja prod | 0,5 | GitHub Actions, secrets |
| Testy (160 przypadków + regresja vs Excel) | 1 | Unikalny — wymaga danych klienta |
| **Razem bez wiedzy domenowej** | **16–20 m-m** | |
| **Marża 30% (ryzyko wiedzy domenowej)** | **+5–6 m-m** | |
| **Szacunek całkowity** | **21–26 m-m** | ok. 7–9 miesięcy dla zespołu 3 os. |

**Wartość niematerialna nieuwzględniona w estymacji:** testy regresji silnika są napisane pod konkretne dane klienta SE Jaworze. Odtworzenie ich od zera wymagałoby dostępu do danych OSD i wiedzy jak je interpretować — nie jest to czas pracy develope rów, to wiedza domenowa właściciela.

---

## 6. HISTORIA ITERACJI

Projekt startował jako skrypt Python (pandas/Jupyter) weryfikujący algorytm art. 38c na danych SE Jaworze — prototyp potwierdzający wykonalność. Na przełomie Q1/Q2 2026 podjęto decyzję o budowie platformy SaaS: zreplikowano algorytm w `internal_engine.py`, powstał pierwszy backend FastAPI (`app/`). W maju 2026 nastąpiła pełna migracja do Django (commit `4977ed3`, 2026-05-07) z architekturą multi-tenant, ORM, Celery i systemem auth. W ciągu 3 tygodni (7–28 maja 2026, ~55 commitów na master) wykonano intensywny build: silnik → multi-tenant → RBAC → dashboard → proformy → workflow → wysyłka email → multi-upload. Aktualna wersja (HEAD `ff4208f`) ma 158 przechodzących testów, jedno środowisko produkcyjne (SE Jaworze, 18 PPE, kwiecień 2026) i tag `v0.3.0-etap2-stable` wskazujący na stan z przed ostatniego tygodnia intensywnych prac UI.
