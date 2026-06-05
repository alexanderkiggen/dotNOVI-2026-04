# DevOps Plan - dotNOVI

Dit document beschrijft de DevOps-strategie en de gekozen tools voor de hele levenscyclus van de dotNOVI applicatie. Het doel is een geautomatiseerde, veilige en betrouwbare pijplijn op te zetten van code tot productie.

## Branching Strategie: GitHub Flow
Voor dit project is gekozen voor de **GitHub Flow** strategie.
* **Waarom:** Als individuele ontwikkelaar (of in een klein team) zorgt deze strategie voor een snelle doorlooptijd zonder de onnodige overhead van complexe release-branches (zoals bij Git Flow).
* **Hoe:** De `main` branch is altijd stabiel en "production-ready". Voor elke nieuwe feature of bugfix wordt een korte `feature/` of `bugfix/` branch aangemaakt. Zodra de code via automatische tests is goedgekeurd, wordt deze via een Pull Request gemerged naar `main` en direct gedeployed. Er wordt gewerkt met *Conventional Commits* voor een heldere historie.

## DevOps Lifecycle & Tooling

### 1. Plan
* **Tool:** GitHub Issues & GitHub Projects
* **Beschrijving:** Nieuwe features, refactor-taken en bugs worden vastgelegd als Issues. Dit geeft een duidelijk en transparant overzicht van het werk dat nog moet gebeuren en de voortgang van de applicatie.

### 2. Code
* **Tool:** Git & Visual Studio Code
* **Beschrijving:** De broncode wordt lokaal ontwikkeld en via Git versiebeheer opgeslagen in GitHub. Om de codekwaliteit consistent te houden, draait er lokaal een linter (ESLint).

### 3. Build & Test (Continuous Integration)
* **Tool:** GitHub Actions, Jest & Docker
* **Beschrijving:** Bij elke push of Pull Request start er automatisch een CI-pijplijn in GitHub Actions. Deze installeert de dependencies, draait de automatische tests (Jest) op meerdere Node.js versies (Matrix testing) en controleert de code. Vervolgens wordt de applicatie geïsoleerd verpakt in een geoptimaliseerde *multi-stage* Docker image.

### 4. Release (Continuous Deployment)
* **Tool:** GitHub Container Registry (GHCR)
* **Beschrijving:** Na een succesvolle test op de `main` branch, pusht de CD-pijplijn de nieuwe Docker image automatisch naar de GitHub Container Registry. Hier krijgt de image een unieke versie-tag (SemVer) en een `latest` tag, waarna een automatische veiligheidsscan (Trivy/npm audit) plaatsvindt.

### 5. Deploy
* **Tool:** Render.com
* **Beschrijving:** De productie-omgeving draait in de cloud bij Render. Vanuit GitHub Actions wordt er een geautomatiseerde web-hook naar Render gestuurd. Render haalt vervolgens de nieuwste image uit GHCR op en start de nieuwe containers veilig op met de gekoppelde PostgreSQL productie-database.

### 6. Monitor & Operate
* **Tool:** Render Logs, Dependabot & `/health` endpoint
* **Beschrijving:** De gezondheid van de applicatie en database-connectie wordt continu in de gaten gehouden via de `/health` route. Render zorgt voor de server uptime logs. Daarnaast controleert GitHub Dependabot wekelijks de repository op verouderde of onveilige NPM-pakketten om de veiligheid proactief te waarborgen.