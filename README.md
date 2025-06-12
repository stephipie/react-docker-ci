# React Docker CI/CD Pipeline

Dieses Projekt demonstriert eine erweiterte CI/CD-Pipeline für eine React-Anwendung mit Docker und automatischem Push zu Docker Hub.

## Projekt-Überblick

- React-Frontend mit Vite
- Multi-Stage Docker Build
- Automatische CI/CD mit GitHub Actions
- Automatischer Push zu Docker Hub

## CI/CD Pipeline

Die Pipeline wird bei jedem Push auf den `main`-Branch ausgeführt und durchläuft folgende Schritte:

1. **Build der React-Anwendung**
   - Checkout des Codes
   - Installation der Dependencies
   - Ausführung des Linters
   - Build der Produktionsversion

2. **Docker Image Build**
   - Multi-Stage Build für optimale Image-Größe
   - Stage 1: Build der Anwendung
   - Stage 2: Nginx-basiertes Runtime Image

3. **Push zu Docker Hub**
   - Authentifizierung via GitHub Secrets
   - Push mit Git SHA als Tag (Secure Hash Algorithm/eindeutiger Hash-Wert)
   - Push mit 'latest' Tag

## Pipeline Beweise

### GitHub Actions CI Pipeline
![Erfolgreicher GitHub Actions Lauf](./screenshots/github-actions.png)

### Docker Hub Repository
![Docker Image in Docker Hub](./screenshots/docker-hub-image.png)

### GitHub Secrets Konfiguration
![GitHub Secrets (Werte verborgen)](./screenshots/docker-hub-secrets-token.png)

## Lokale Entwicklung

```bash
# Installation der Dependencies
npm install

# Entwicklungsserver starten
npm run dev

# Produktionsbuild erstellen
npm run build

# So würde man das Docker Image lokal bauen
docker build -t react-docker-ci .
```

## Docker Hub Repository

Das gebaute Image ist verfügbar unter:
`docker pull stephipie/react-docker-ci:2007a07`

## Technologie-Stack

- React + Vite
- Docker mit Multi-Stage Build
- GitHub Actions
- Docker Hub
- Nginx (Production Server)