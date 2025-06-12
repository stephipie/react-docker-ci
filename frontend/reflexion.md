# Reflexion zur erweiterten CI-Pipeline mit Docker und Docker Hub

## 1. Zweck der zwei Stages im Multi-Stage-Dockerfile (Builder und Runner)

In meinem Multi-Stage-Dockerfile habe ich zwei Stufen definiert:

- **Builder Stage**: Diese verwendet ein Node.js-basiertes Image und führt den Build-Prozess der React-App aus. Dazu gehören die Installation der Dependencies (`npm ci`) und das Erzeugen der Produktionsbuilds (`npm run build`). Dieser Schritt erzeugt den finalen statischen Code im `dist`-Ordner.

- **Runner Stage**: Diese basiert auf einem kleinen, sicheren nginx-Image (z. B. `nginx:alpine`) und übernimmt nur den ausgelagerten Build-Ordner. Sie dient dazu, die App möglichst schlank, performant und sicher auszuliefern.

Vorteile für CI/CD:
- Das endgültige Image ist kleiner, da Build-Werkzeuge wie `npm`, `node_modules` und Quellcode nicht enthalten sind.
- Trennung von Build und Runtime verbessert die Sicherheit und reduziert Angriffsflächen.
- Der Build ist reproduzierbar, unabhängig von der lokalen Umgebung.

## 2. Sichere Speicherung der Docker Hub Zugangsdaten in der CI-Plattform

Ich habe die Zugangsdaten (Benutzername und Access Token) nicht direkt im Code gespeichert, sondern als Secrets in GitHub Actions hinterlegt:

- `DOCKERHUB_USERNAME` → Docker Hub Benutzername
- `DOCKERHUB_TOKEN` → sicher generiertes Access Token mit Schreibrechten

Warum das sicherer ist:
- Secrets werden verschlüsselt gespeichert und sind nur zur Laufzeit verfügbar.
- Sie erscheinen nicht in Logs oder der Benutzeroberfläche.
- Direkt im Code eingebundene Zugangsdaten wären öffentlich sichtbar (z. B. bei Open Source) und könnten leicht missbraucht werden.

## 3. Abfolge der vier Hauptschritte in der CI-Pipeline

Die erweiterte CI-Pipeline führt folgende Schritte aus:

1. **Code holen (checkout)**  
   Holt den aktuellen Stand des Repositories, um damit zu arbeiten.

2. **Docker Image bauen**  
   Führt `docker build` mit dem Multi-Stage-Dockerfile aus und erstellt ein Image mit dem Produktionsbuild der React-App.

3. **Login bei Docker Hub**  
   Meldet sich mithilfe der Secrets bei Docker Hub an (`docker login`), um Schreibrechte für den Push zu erhalten.

4. **Image pushen**  
   Lädt das gebaute Image mit einem eindeutigen Tag in das Repository bei Docker Hub hoch (`docker push`).

## 4. Verwendeter Image-Tag und dessen Bedeutung

Ich verwende den Git Commit Hash (z. B. `abcdef1`) als Image-Tag:

```bash
docker build -t meindockeruser/react-docker-ci:abcdef1 .
```
Zusätzlich tagge ich das Image auf dem `main`-Branch auch mit `latest`.

Warum eindeutige Tags wichtig sind:
- Sie ermöglichen eine klare Rückverfolgbarkeit, welches Image zu welchem Code-Stand gehört.
- Bei Deployments lässt sich exakt ein bestimmtes Image nutzen.
- Verhindert Verwechslung und Probleme durch gecachte oder veraltete Images.
- Unterstützt Rollbacks, falls ein Fehler auftritt.

## 5. Vorgehen bei Fehlern (z. B. Image konnte nicht gepusht werden)

Wenn das Pushen des Docker Images fehlschlägt, würde ich folgende Schritte zur Fehlersuche unternehmen:

1. **Logs in GitHub Actions ansehen**  
   Welcher Schritt ist fehlgeschlagen? Welche Fehlermeldung erscheint (z. B. `unauthorized`, `denied`, `tag invalid`)?

2. **Secrets überprüfen**  
   In GitHub → Settings → Secrets:
   - Existieren die Secrets `DOCKERHUB_USERNAME` und `DOCKERHUB_TOKEN`?
   - Ist das Token korrekt und hat Schreibzugriff?

3. **Zugriffsrechte des Docker Hub Tokens prüfen**  
   Eventuell ein neues Access Token mit Rolle `Read/Write` erstellen.

4. **Tag-Format prüfen**  
   Docker Tags dürfen nur bestimmte Zeichen enthalten. Kein `/`, keine Umlaute, max. 128 Zeichen.
   

5. **Manuell einloggen und pushen (lokaler Test)**  
   Zum Ausschluss von Fehlern in der CI-Umgebung: Build und Push lokal testen.

