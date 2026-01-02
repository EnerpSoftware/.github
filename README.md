# ENERP Software AI - CI/CD Pipeline

## Spis treści
1. [Co to jest CI/CD? (dla laika)](#co-to-jest-cicd-dla-laika)
2. [Jak działa nasz pipeline?](#jak-działa-nasz-pipeline)
3. [Workflow dla zespołu 3 osób + Claude Code](#workflow-dla-zespołu-3-osób--claude-code)
4. [Instrukcja krok po kroku](#instrukcja-krok-po-kroku)
5. [Reusable Workflows](#reusable-workflows)
6. [FAQ / Rozwiązywanie problemów](#faq--rozwiązywanie-problemów)

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
