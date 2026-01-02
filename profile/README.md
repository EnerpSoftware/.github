# ENERP Software AI

> Intelligent solutions for real-world problems

## O nas

Zespol 3 developerow tworzacych innowacyjne rozwiazania z wykorzystaniem AI, IoT i automatyzacji.
Pracujemy z **Claude Code** jako AI pair programmer.

## Nasze projekty

### Horse SafeCube (zarzadzanie stajniami)
- **horse** - Glowna aplikacja TypeScript
- **horse.safecube.me** - Wersja produkcyjna
- **analiza-danych-konia** - Analiza danych Python

### Web Applications
- **voiceorder.io** - Aplikacja glosowych zamowien
- **prawniczek-gpt-app** - AI marketplace (prawnik, ksiegowy, doradca)
- **enerp-audit-landing** - Landing page audyt energetyczny
- **enerp-magazyny-energii-landing** - Landing magazyny energii

### IoT & Embedded
- **ESP32-AI** - Integracja AI z mikrokontrolerami
- **TexasInstruments-*** - Projekty z czujnikami mmWave radar
- **LILYGO-T3** - Projekty z wyswietlaczami

## CI/CD Pipeline

Wszystkie projekty korzystaja z **centralnych reusable workflows**:

```
Push --> Build --> Test --> [Auto-Bug on fail] --> Deploy (k3s)
```

| Workflow | Opis |
|----------|------|
| `reusable-build-node.yml` | Node.js/TypeScript |
| `reusable-build-python.yml` | Python/Django |
| `reusable-test.yml` | Testy + auto-bug |
| `reusable-build-embedded.yml` | PlatformIO/ESP32 |
| `reusable-docker-build.yml` | Docker + ghcr.io |
| `reusable-deploy-k3s.yml` | Kubernetes deploy |

## Dokumentacja

- [Pelna dokumentacja CI/CD](./README.md) - instrukcja dla zespolu
- [CLAUDE.md](./CLAUDE.md) - wiedza dla Claude Code

## Tech Stack

![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat&logo=nodedotjs&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)

---

*Powered by Claude Code*
