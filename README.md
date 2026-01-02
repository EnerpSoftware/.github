# ENERP Software AI - CI/CD Pipeline

## Spis treści
1. [Co to jest CI/CD? (dla laika)](#co-to-jest-cicd-dla-laika)
2. [Jak działa nasz pipeline?](#jak-działa-nasz-pipeline)
3. [Workflow dla zespołu 3 osób + Claude Code](#workflow-dla-zespołu-3-osób--claude-code)
4. [Instrukcja krok po kroku](#instrukcja-krok-po-kroku)
5. [Reusable Workflows](#reusable-workflows)
6. [GitHub Projects - Zarządzanie zadaniami](#github-projects---zarządzanie-zadaniami)
7. [FAQ / Rozwiązywanie problemów](#faq--rozwiązywanie-problemów)

---

## Co to jest CI/CD? (dla laika)

### Analogia: Linia produkcyjna w fabryce

Wyobraź sobie fabrykę samochodów:

```
   PROGRAMISTA                    CI/CD PIPELINE                      PRODUKCJA
   ___________                    ______________                      __________
  |           |                  |              |                    |          |
  |  Pisze    |  -- wysyła -->   |  Sprawdza    |  -- jeśli OK -->   |  Działa  |
  |   kod     |     kod          |  i testuje   |                    |  u ludzi |
  |___________|                  |______________|                    |__________|
```

**CI (Continuous Integration)** = Ciągłe sprawdzanie
- Każdy kod który wysyłasz jest AUTOMATYCZNIE sprawdzany
- Jak kontrola jakości na linii produkcyjnej
- Wykrywa błędy ZANIM trafią do użytkowników

**CD (Continuous Delivery/Deployment)** = Ciągłe dostarczanie
- Po przejściu testów, kod automatycznie trafia na serwer
- Jak gotowy samochód zjeżdżający z linii produkcyjnej

### Co się dzieje gdy wysyłasz kod?

```
1. Piszesz kod na swoim komputerze
        |
        v
2. Robisz "git push" (wysyłasz na GitHub)
        |
        v
3. GitHub AUTOMATYCZNIE uruchamia pipeline:
   +------------------------------------------+
   |  BUILD: Czy kod się kompiluje?           |
   |    |                                     |
   |    v                                     |
   |  TEST: Czy testy przechodzą?             |
   |    |                                     |
   |    v (jeśli FAIL)                        |
   |  AUTO-BUG: Tworzy Issue z opisem błędu   |
   +------------------------------------------+
        |
        v (jeśli OK)
4. Kod jest gotowy do deployu (opcjonalnie automatyczny)
```

---

## Jak działa nasz pipeline?

### Architektura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ENERP SOFTWARE AI                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PROGRAMISTA 1          PROGRAMISTA 2          PROGRAMISTA 3               │
│   (+ Claude Code)        (+ Claude Code)        (+ Claude Code)             │
│        |                      |                      |                       │
│        └──────────────────────┼──────────────────────┘                       │
│                               |                                              │
│                               v                                              │
│                    ┌─────────────────────┐                                  │
│                    │   GitHub Repo       │                                  │
│                    │   (np. horse)       │                                  │
│                    └──────────┬──────────┘                                  │
│                               |                                              │
│                               v                                              │
│              ┌────────────────────────────────┐                             │
│              │      GitHub Actions            │                             │
│              │  (automatyczny pipeline)       │                             │
│              └───────────────┬────────────────┘                             │
│                              |                                               │
│           ┌──────────────────┼──────────────────┐                           │
│           v                  v                  v                            │
│     ┌──────────┐      ┌──────────┐      ┌──────────┐                       │
│     │  BUILD   │ ---> │   TEST   │ ---> │  DEPLOY  │                       │
│     │          │      │          │      │  (k3s)   │                       │
│     └──────────┘      └─────┬────┘      └──────────┘                       │
│                             |                                               │
│                             v (jeśli FAIL)                                  │
│                    ┌─────────────────────┐                                  │
│                    │  AUTO-BUG ISSUE     │                                  │
│                    │  na GitHub Projects │                                  │
│                    └─────────────────────┘                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Co robi każdy krok?

| Krok | Co robi? | Czas | Co jeśli fail? |
|------|----------|------|----------------|
| **BUILD** | Instaluje zależności, kompiluje kod | ~30s | Pipeline się zatrzymuje |
| **TEST** | Uruchamia testy jednostkowe | ~30s | Tworzy AUTO-BUG Issue |
| **DEPLOY** | Wysyła na serwer (opcjonalnie) | ~60s | Powiadamia zespół |

---

## Workflow dla zespołu 3 osób + Claude Code

### Codzienny rytm pracy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DZIEŃ PRACY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  RANO (przed pracą)                                                         │
│  ─────────────────                                                          │
│  1. Sprawdź GitHub Projects - jakie są zadania?                             │
│  2. Sprawdź czy są AUTO-BUG issues (czerwone!)                              │
│  3. Przypisz się do zadania                                                 │
│                                                                              │
│  PRACA (z Claude Code)                                                      │
│  ────────────────────                                                       │
│  4. Otwórz terminal w katalogu projektu                                     │
│  5. Uruchom: claude                                                         │
│  6. Powiedz Claude co chcesz zrobić                                         │
│  7. Claude pisze kod, ty weryfikujesz                                       │
│  8. Commit + Push                                                           │
│  9. Pipeline automatycznie sprawdza                                         │
│                                                                              │
│  KONIEC DNIA                                                                │
│  ──────────────                                                             │
│  10. Sprawdź czy wszystkie pipeline'y przeszły (zielone ✓)                  │
│  11. Zaktualizuj status zadań na GitHub Projects                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Scenariusze z Claude Code

#### Scenariusz 1: Nowe zadanie

```bash
# Terminal
cd ~/projekty/horse
claude

# Rozmowa z Claude:
> "Mam zadanie: dodać przycisk eksportu danych do CSV.
>  Issue #42 na GitHub Projects."

# Claude:
# - Przeczyta kod
# - Zaproponuje rozwiązanie
# - Napisze kod
# - Utworzy commit
# - Zapyta czy pushować

> "tak, pushuj"

# Po pushu - pipeline automatycznie:
# ✓ BUILD - sprawdza czy się kompiluje
# ✓ TEST - uruchamia testy
# Jeśli OK - gotowe!
```

#### Scenariusz 2: Test nie przeszedł (AUTO-BUG)

```bash
# Dostajesz powiadomienie: "Test failure in abc123"
# Na GitHub Projects pojawia się nowy BUG

# Terminal
claude

> "Mamy bug z testów, issue #45. Sprawdź logi i napraw."

# Claude:
# - Pobierze logi z GitHub Actions
# - Zidentyfikuje problem
# - Zaproponuje fix
# - Napisze poprawkę
# - Commit + Push

# Pipeline ponownie sprawdza...
# ✓ Tym razem przechodzi!
# Issue #45 można zamknąć
```

#### Scenariusz 3: Code Review (praca zespołowa)

```bash
# Osoba A: tworzy branch i PR
git checkout -b feature/nowy-raport
claude
> "Dodaj generowanie raportu PDF"
# ... Claude pisze kod ...
> "utwórz PR"

# Osoba B: robi review
claude
> "Sprawdź PR #15, zrób code review"
# Claude analizuje zmiany i komentuje

# Osoba A: poprawia po review
> "Popraw uwagi z review PR #15"
# ... poprawki ...
> "pushuj poprawki"

# Po merge - pipeline automatycznie deployuje
```

### Podział odpowiedzialności

```
┌────────────────────────────────────────────────────────────────┐
│                    ZESPÓŁ 3 OSÓB                                │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OSOBA 1 (np. Frontend)         OSOBA 2 (np. Backend)          │
│  ─────────────────────          ────────────────────           │
│  - Repozytoria: horse,          - Repozytoria: horse,          │
│    horse.safecube.me              analiza-danych-konia         │
│  - Focus: UI/UX                 - Focus: API, bazy danych      │
│                                                                 │
│  OSOBA 3 (np. IoT/DevOps)                                      │
│  ────────────────────────                                      │
│  - Repozytoria: ESP32-AI,                                      │
│    TexasInstruments-*                                          │
│  - Focus: Embedded, CI/CD                                      │
│                                                                 │
│  CLAUDE CODE - wspólny dla wszystkich                          │
│  ────────────────────────────────────                          │
│  - Pomaga pisać kod                                            │
│  - Czyta i rozumie istniejący kod                              │
│  - Tworzy commity i PR                                         │
│  - Analizuje błędy z CI                                        │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## Instrukcja krok po kroku

### Dla nowego programisty (setup)

#### 1. Zainstaluj wymagane narzędzia

```bash
# macOS
brew install gh node python3

# Zaloguj się do GitHub
gh auth login

# Zainstaluj Claude Code
npm install -g @anthropic-ai/claude-code
```

#### 2. Sklonuj repozytorium

```bash
# Przykład dla projektu "horse"
gh repo clone EnerpSoftware/horse
cd horse
```

#### 3. Zainstaluj zależności (ważne dla CI!)

```bash
# Node.js projekty
npm install
# To utworzy package-lock.json - MUSI być w repo!

# Python projekty
pip install -r requirements.txt
```

#### 4. Sprawdź czy CI działa lokalnie

```bash
# Node.js
npm test

# Python
pytest
```

### Codzienna praca

#### Rozpoczęcie pracy

```bash
# 1. Wejdź do projektu
cd ~/projekty/horse

# 2. Pobierz najnowsze zmiany
git pull

# 3. Uruchom Claude Code
claude
```

#### Tworzenie zmian

```bash
# W Claude Code:
> "Dodaj funkcję X do pliku Y"

# Claude napisze kod, potem:
> "zapisz zmiany i utwórz commit"

# Claude utworzy commit z opisem
> "pushuj na GitHub"

# Automatycznie uruchomi się pipeline!
```

#### Sprawdzanie statusu CI

```bash
# W terminalu (bez Claude)
gh run list --limit 5

# Lub w Claude:
> "sprawdź status ostatnich buildów"
```

#### Gdy test nie przejdzie

```bash
# 1. Sprawdź logi
gh run view <run-id> --log

# 2. Lub poproś Claude:
> "ostatni build nie przeszedł, sprawdź dlaczego i napraw"
```

---

## Reusable Workflows

### Dostępne workflow (w tym repo)

| Workflow | Użycie | Opis |
|----------|--------|------|
| `reusable-build-node.yml` | Node.js/TypeScript | Buduje projekt, cache npm |
| `reusable-build-python.yml` | Python/Django | Buduje, lint (ruff), type check |
| `reusable-test.yml` | Wszystkie | Testy + AUTO-BUG na failure |
| `reusable-build-embedded.yml` | ESP32/IoT | PlatformIO build |
| `reusable-docker-build.yml` | Kontenery | Build + push do ghcr.io |
| `reusable-deploy-k3s.yml` | Kubernetes | Deploy do klastra k3s |

### Jak dodać CI do nowego repo?

Utwórz plik `.github/workflows/ci.yml`:

```yaml
# Dla projektu Node.js/TypeScript
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: EnerpSoftware/.github/.github/workflows/reusable-build-node.yml@main
    with:
      node-version: '20'

  test:
    needs: build
    uses: EnerpSoftware/.github/.github/workflows/reusable-test.yml@main
    with:
      test-command: 'npm test --if-present -- --passWithNoTests'
      project-type: 'node'
```

```yaml
# Dla projektu Python
name: CI Pipeline

on:
  push:
    branches: [main, develop]

jobs:
  build:
    uses: EnerpSoftware/.github/.github/workflows/reusable-build-python.yml@main
    with:
      python-version: '3.11'

  test:
    needs: build
    uses: EnerpSoftware/.github/.github/workflows/reusable-test.yml@main
    with:
      test-command: 'pytest -v || echo "No tests"'
      project-type: 'python'
```

---


---

## GitHub Projects - Zarządzanie zadaniami

### Co to jest GitHub Projects?

GitHub Projects to tablica kanban zintegrowana z GitHub - jak Trello, ale połączona bezpośrednio z kodem.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ENERP Development Board                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  📋 BACKLOG      📝 TO DO        🔄 IN PROGRESS     ✅ DONE                  │
│  ┌──────────┐   ┌──────────┐    ┌──────────┐      ┌──────────┐             │
│  │ Task #25 │   │ Task #22 │    │ Task #19 │      │ Task #17 │             │
│  │ Task #26 │   │ Task #23 │    │ (Jan)    │      │ Task #18 │             │
│  │ Task #27 │   │ Bug #24  │    │          │      │ Bug #20  │             │
│  └──────────┘   └──────────┘    └──────────┘      └──────────┘             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Jak task przechodzi przez pipeline?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     CYKL ŻYCIA ZADANIA                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. UTWORZENIE TASKA                                                        │
│     ────────────────                                                        │
│     - Ręcznie przez programistę                                             │
│     - Automatycznie przez CI (gdy test FAIL → AUTO-BUG)                     │
│                                                                              │
│                          │                                                   │
│                          ▼                                                   │
│                                                                              │
│  2. BACKLOG / TO DO                                                         │
│     ───────────────                                                         │
│     - Task czeka na podjęcie                                                │
│     - Widoczny dla całego zespołu                                           │
│     - Claude Code może go odczytać                                          │
│                                                                              │
│                          │                                                   │
│                          ▼                                                   │
│                                                                              │
│  3. IN PROGRESS (z Claude Code)                                             │
│     ──────────────────────────                                              │
│     - Programista mówi: "wezmę task #22"                                    │
│     - Claude przypisuje task i zmienia status                               │
│     - Claude pisze kod zgodnie z opisem                                     │
│     - Commit + Push                                                         │
│                                                                              │
│                          │                                                   │
│                          ▼                                                   │
│                                                                              │
│  4. CI PIPELINE (automatycznie)                                             │
│     ──────────────────────────                                              │
│     ┌─────────┐    ┌─────────┐    ┌─────────┐                              │
│     │  BUILD  │───▶│  TEST   │───▶│ DEPLOY? │                              │
│     └─────────┘    └────┬────┘    └─────────┘                              │
│                         │                                                    │
│              ┌──────────┴──────────┐                                        │
│              │                     │                                        │
│              ▼                     ▼                                        │
│         ✅ PASS               ❌ FAIL                                       │
│              │                     │                                        │
│              ▼                     ▼                                        │
│                                                                              │
│  5a. DONE (sukces)           5b. AUTO-BUG (porażka)                        │
│      ─────────────               ─────────────────                          │
│      - Claude zamyka task        - Pipeline tworzy nowy Bug Issue           │
│      - Dodaje komentarz          - Przypisuje do autora commita             │
│      - Task → kolumna DONE       - Bug → kolumna TO DO                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Claude Code + GitHub Projects

Claude Code jest **w pełni zintegrowany** z GitHub Projects. Może:

| Akcja | Jak powiedzieć Claude | Co się stanie |
|-------|----------------------|---------------|
| Lista tasków | "Jakie mam taski?" | Wyświetli otwarte issues z projektu |
| Szczegóły | "Pokaż task #22" | Wyświetli pełny opis i kryteria |
| Podjęcie taska | "Wezmę task #22" | Przypisze Cię i zmieni status |
| Praca nad taskiem | "Zrób task #22" | Przeczyta opis, napisze kod |
| Zamknięcie | "Zamknij task #22" | Doda komentarz i zamknie issue |
| Nowy task | "Utwórz task: dodać X" | Utworzy issue w odpowiednim repo |

### Przykładowa sesja z Claude Code

```bash
# Uruchamiasz Claude Code
$ claude

# Pytasz o taski
> Jakie taski mam do zrobienia w projekcie horse?

# Claude odpowiada:
🔍 Sprawdzam GitHub Projects dla EnerpSoftware/horse...

Znalazłem 4 otwarte taski:

📋 TO DO:
  #22 [TASK] Dodać filtrowanie po rasie konia
  #23 [TASK] Eksport do PDF

🐛 BUGS:
  #24 [BUG] Błąd walidacji daty urodzenia
  #25 [AUTO-BUG] Test failure in abc123

Którym chcesz się zająć?

# Wybierasz task
> Wezmę #24, bug z walidacją

# Claude działa:
✅ Przypisałem Cię do #24
📖 Czytam szczegóły buga...

Problem: Walidacja akceptuje daty z przyszłości.
Lokalizacja: src/utils/validation.ts:45

Naprawiam...

[... Claude pisze kod ...]

✅ Poprawka gotowa. Zmiany:
   - src/utils/validation.ts (dodano sprawdzenie daty)
   - __tests__/validation.test.ts (nowy test)

Utworzyć commit i push?

> tak

📤 Pushing to origin/main...
🔄 CI Pipeline uruchomiony: https://github.com/EnerpSoftware/horse/actions/runs/123

Poczekać na wynik CI i zamknąć issue #24?

> tak

⏳ Czekam na CI...
✅ CI PASSED (Build: 15s, Test: 22s)

Zamykam issue #24 z komentarzem:
"Fixed date validation - now rejects future dates.
 Added test case for edge cases."

✅ Issue #24 zamknięty!
```

### Automatyczne tworzenie bugów (AUTO-BUG)

Gdy test nie przejdzie, pipeline **automatycznie** tworzy bug issue:

```
┌─────────────────────────────────────────────────────────────────┐
│                    AUTO-BUG WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Programista pushuje kod                                     │
│                    │                                             │
│                    ▼                                             │
│  2. CI uruchamia testy                                          │
│                    │                                             │
│                    ▼                                             │
│  3. Test FAIL! ❌                                                │
│                    │                                             │
│                    ▼                                             │
│  4. Pipeline AUTOMATYCZNIE tworzy Issue:                        │
│     ┌─────────────────────────────────────────────────────┐    │
│     │ [AUTO-BUG] Test failure in abc1234                  │    │
│     │                                                      │    │
│     │ ## Informacje o buildzie                            │    │
│     │ - Commit: abc1234                                   │    │
│     │ - Branch: main                                      │    │
│     │ - Autor: @jan_kowalski                              │    │
│     │ - Workflow run: [Link do logów]                     │    │
│     │                                                      │    │
│     │ ## Wymagane działania                               │    │
│     │ - [ ] Przeanalizować logi testów                    │    │
│     │ - [ ] Zidentyfikować przyczynę                      │    │
│     │ - [ ] Naprawić i utworzyć PR                        │    │
│     │                                                      │    │
│     │ Labels: bug, auto-generated, ci-failure             │    │
│     │ Assignee: @jan_kowalski                             │    │
│     └─────────────────────────────────────────────────────┘    │
│                    │                                             │
│                    ▼                                             │
│  5. Issue pojawia się na GitHub Projects w kolumnie "TO DO"     │
│                    │                                             │
│                    ▼                                             │
│  6. Programista (lub Claude) naprawia i zamyka                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Tworzenie tasków dla zespołu

#### Ręczne tworzenie (przez GitHub UI)

1. Wejdź na https://github.com/EnerpSoftware/horse/issues
2. Kliknij "New Issue"
3. Wybierz szablon (Bug Report lub Task)
4. Wypełnij formularz
5. Issue automatycznie trafi do Projects

#### Przez Claude Code

```bash
> Utwórz task w repo horse: "Dodać eksport do Excel"
  z opisem: "Użytkownik chce eksportować dane koni do formatu .xlsx"

# Claude utworzy:
✅ Utworzono Issue #26: [TASK] Dodać eksport do Excel
   Repo: EnerpSoftware/horse
   URL: https://github.com/EnerpSoftware/horse/issues/26
```

#### Przez terminal (gh CLI)

```bash
gh issue create \
  --repo EnerpSoftware/horse \
  --title "[TASK] Dodać eksport do Excel" \
  --body "Użytkownik chce eksportować dane koni do formatu .xlsx"
```

### Współpraca zespołu 3 osób

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CODZIENNY WORKFLOW                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  🌅 RANO (Daily standup - opcjonalnie)                                      │
│  ─────────────────────────────────────                                      │
│                                                                              │
│  Każdy sprawdza GitHub Projects:                                            │
│                                                                              │
│  OSOBA 1:                    OSOBA 2:                    OSOBA 3:           │
│  "Biorę #22 (frontend)"      "Biorę #23 (API)"          "Biorę #24 (IoT)"  │
│         │                          │                          │             │
│         └──────────────────────────┼──────────────────────────┘             │
│                                    │                                        │
│                                    ▼                                        │
│                                                                              │
│  💻 PRACA (każdy ze swoim Claude Code)                                      │
│  ─────────────────────────────────────                                      │
│                                                                              │
│  Terminal 1:                 Terminal 2:                 Terminal 3:        │
│  $ claude                    $ claude                    $ claude           │
│  > "Zrób task #22"           > "Zrób task #23"           > "Zrób task #24" │
│         │                          │                          │             │
│         │                          │                          │             │
│         ▼                          ▼                          ▼             │
│                                                                              │
│  🔄 CI PIPELINE (równolegle dla każdego)                                    │
│  ───────────────────────────────────────                                    │
│                                                                              │
│  Push #22 → CI ✅             Push #23 → CI ✅             Push #24 → CI ❌ │
│         │                          │                          │             │
│         │                          │                          │             │
│         ▼                          ▼                          ▼             │
│                                                                              │
│  🌆 KONIEC DNIA                                                             │
│  ─────────────                                                              │
│                                                                              │
│  #22 → DONE ✅               #23 → DONE ✅               #24 → nowy BUG 🐛 │
│                                                           (Osoba 3 naprawi │
│                                                            jutro)           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Dobre praktyki

#### ✅ TAK (rób to)

- Jeden task = jedna funkcjonalność
- Opisuj task jasno (Claude musi zrozumieć co zrobić)
- Sprawdzaj CI przed zamknięciem taska
- Używaj labels: `bug`, `task`, `feature`, `urgent`
- Przypisuj się do tasków które robisz

#### ❌ NIE (unikaj)

- Nie twórz gigantycznych tasków (dziel na mniejsze)
- Nie zostawiaj tasków "In Progress" na noc bez commita
- Nie ignoruj AUTO-BUG issues (są pilne!)
- Nie zamykaj tasków bez działającego CI

### Komendy gh CLI dla tasków

```bash
# Lista Issues w repo
gh issue list --repo EnerpSoftware/horse

# Szczegóły Issue
gh issue view 22 --repo EnerpSoftware/horse

# Utworzenie Issue
gh issue create --repo EnerpSoftware/horse --title "Tytuł" --body "Opis"

# Przypisanie się
gh issue edit 22 --repo EnerpSoftware/horse --add-assignee @me

# Dodanie komentarza
gh issue comment 22 --repo EnerpSoftware/horse --body "Komentarz"

# Zamknięcie Issue
gh issue close 22 --repo EnerpSoftware/horse

# Lista projektów organizacji
gh project list --owner EnerpSoftware

# Items w projekcie
gh project item-list 1 --owner EnerpSoftware
```

## FAQ / Rozwiązywanie problemów

### "Build failed - package-lock.json not found"

```bash
# Rozwiązanie: wygeneruj package-lock.json
npm install
git add package-lock.json
git commit -m "chore: Add package-lock.json"
git push
```

### "Test failed" - jak zobaczyć szczegóły?

```bash
# Opcja 1: GitHub CLI
gh run view <run-id> --log

# Opcja 2: Przeglądarka
# Wejdź na: github.com/EnerpSoftware/<repo>/actions

# Opcja 3: Claude Code
> "pokaż logi ostatniego nieudanego buildu"
```

### "Jak sprawdzić czy mój kod przeszedł CI?"

```bash
# Po pushu poczekaj ~1 minutę, potem:
gh run list --limit 1

# Powinieneś zobaczyć:
# completed  success  ✓
# lub
# completed  failure  ✗
```

### "Chcę przetestować lokalnie przed pushem"

```bash
# Node.js
npm test

# Python
pytest

# Jeśli testy przechodzą lokalnie, powinny przejść też na CI
```

### "Jak dodać nowy test?"

```bash
# Poproś Claude:
> "Dodaj test jednostkowy dla funkcji calculateTotal w pliku math.js"

# Claude utworzy plik w __tests__/ lub tests/
```

### "Pipeline trwa za długo"

Możliwe przyczyny:
1. Dużo zależności npm/pip
2. Wolne testy
3. Brak cache

Rozwiązanie: Użyj self-hosted runners (szybsze)

---

## Kontakt i pomoc

- **GitHub Organization**: [EnerpSoftware](https://github.com/EnerpSoftware)
- **GitHub Projects**: [ENERP Development Board](https://github.com/orgs/EnerpSoftware/projects/1)
- **Problemy z CI**: Utwórz Issue w tym repo (.github)

---

*Ostatnia aktualizacja: 2026-01-02*
