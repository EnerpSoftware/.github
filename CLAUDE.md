# ENERP Software AI - Knowledge Base

## Organizacja

- **GitHub Organization**: `EnerpSoftware`
- **Wyświetlana nazwa**: ENERP Software AI
- **Typ**: Firma software'owa / AI / IoT
- **Stack technologiczny**: Node.js, TypeScript, Python, React, Django, C (embedded)
- **Infrastruktura**: GitHub Actions (self-hosted runners), k3s cluster, ghcr.io

## Architektura CI/CD

### Reusable Workflows (w tym repo)

| Workflow | Przeznaczenie |
|----------|---------------|
| `reusable-build-node.yml` | Build projektów Node.js/TypeScript |
| `reusable-build-python.yml` | Build projektów Python/Django |
| `reusable-test.yml` | Uruchamianie testów + auto-bug na failure |
| `reusable-docker-build.yml` | Build i push do ghcr.io |
| `reusable-deploy-k3s.yml` | Deployment do klastra k3s |
| `reusable-build-embedded.yml` | Build firmware (PlatformIO) |

### Użycie w repozytoriach

```yaml
jobs:
  build:
    uses: EnerpSoftware/.github/.github/workflows/reusable-build-node.yml@main
    with:
      node-version: '20'
      runs-on: 'ubuntu-latest'  # lub 'self-hosted'
```

## Repozytoria

### Projekty Horse (System zarządzania stajniami)

| Repo | Język | Opis |
|------|-------|------|
| `horse` | TypeScript | Główna aplikacja |
| `horse.safecube.me` | TypeScript | Wersja produkcyjna SafeCube |
| `horse.safecube-2026` | JavaScript | System monitorowania dobrostanu koni |
| `horse-safe-cube-MVP` | JavaScript | MVP wersja |
| `analiza-danych-konia` | Python | Analiza danych o koniach |
| `apphorse-demo` | - | Demo aplikacji |

### Projekty Web/Landing

| Repo | Język | Opis |
|------|-------|------|
| `appretail.safecube.me` | JavaScript | Aplikacja retail |
| `enerp-magazyny-energii-landing` | TypeScript | Landing page magazyny energii |
| `enerp-audit-landing` | JavaScript | Landing page audyt |
| `voiceorder.io` | JavaScript | System zamówień głosowych |
| `NORAD` | JavaScript | Projekt NORAD |
| `prawniczek-gpt-app` | TypeScript | AI marketplace: prawnik, księgowy, doradca (PUBLICZNE) |

### Projekty IoT/Embedded

| Repo | Język | Opis |
|------|-------|------|
| `TexasInstruments-IWRL6432FSPEVM` | C | Radar mmWave + Vital Signs |
| `TexasInstruments-IWR6843AOPEVM` | Python | Radar mmWave Demo |
| `TexasInstruments-IWR6843AOPEVM-v2` | XS | Rozszerzenie z Vital Signs |
| `ESP32-AI` | Python | ESP32 z AI |
| `LILYGO-T3` | C | LILYGO T3 (w NotAngrySoftware) |

### Projekty AI/ML

| Repo | Język | Opis |
|------|-------|------|
| `Contact-Crawler` | Python | Crawler kontaktów |
| `whisper` | - | OpenAI Whisper (PUBLICZNE) |

### Inne

| Repo | Język | Opis |
|------|-------|------|
| `3DModels` | - | Modele 3D |
| `EduGames` | - | Portal edukacyjny |
| `TEXAS` | - | Projekt Texas |
| `SINGU2026` | - | Projekt Singu |
| `ci-test-repo` | JavaScript | Testowe repo CI/CD |

## GitHub Project

- **Nazwa**: ENERP Development Board
- **Custom Fields**:
  - `Priority`: Critical, High, Medium, Low
  - `TaskType`: Bug, Feature, Task, CI-Failure
  - `Claude-Ready`: boolean - czy zadanie gotowe dla Claude Code

## Konwencje

### Branches
- `main` - produkcja
- `develop` - development
- `feature/*` - nowe funkcje
- `fix/*` - poprawki

### Commits
```
feat: Nowa funkcja
fix: Poprawka buga
chore: Zadania techniczne
ci: Zmiany CI/CD
docs: Dokumentacja
```

### CI Workflow Pattern
Każde repo powinno mieć `.github/workflows/ci.yml` wywołujący reusable workflows z tego repo.

## Kontakt

- **Organizacja GitHub**: https://github.com/EnerpSoftware
- **Runners**: Self-hosted na dedykowanych VM
- **Cluster**: k3s dla deploymentu
