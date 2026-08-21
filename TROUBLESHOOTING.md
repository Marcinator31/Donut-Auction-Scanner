# Build schlägt fehl — so findest du die Ursache

## Zuerst: den echten Fehler holen

Die Übersichtsseite zeigt nur rot oder grün. Der Fehler steht eine Ebene tiefer:

1. Reiter **Actions**
2. Auf den obersten fehlgeschlagenen Lauf klicken
3. Links auf den Job **Build (Java 25)**
4. Der rote Schritt ist schon aufgeklappt — den Text kopieren

Mit dem neuen Workflow sagt dir bereits der **Name des roten Schritts**, worum
es geht:

| Roter Schritt | Bedeutung |
|---|---|
| **Preflight - Dateien prüfen** | Eine Datei fehlt im Repo. Welche, steht dabei. |
| **Gradle-Wrapper prüfen** | `gradle-wrapper.jar` fehlt oder ist beschädigt |
| **JDK 25 einrichten** | Java 25 nicht verfügbar auf dem Runner |
| **A - Gradle startet** | Wrapper kaputt oder Gradle-Download scheitert |
| **B - Projekt lässt sich konfigurieren** | Loom-Plugin, Minecraft-Download, Abhängigkeiten |
| **C - Kompilieren und bauen** | Der Java-Code selbst |

## Die wahrscheinlichsten Ursachen

### Laufzeit unter einer Minute

Ein Loom-Build lädt Minecraft herunter und dekompiliert es. Von Null dauert
das **mehrere Minuten**. Bricht es nach 30–60 Sekunden ab, ist es gar nicht
bis zum Kompilieren gekommen — dann liegt es an Schritt Preflight, A oder B,
nicht am Java-Code.

### Dateien über die Weboberfläche hochgeladen

„Add files via upload" im Browser lässt regelmäßig Dateien liegen:

- `gradle/wrapper/gradle-wrapper.jar` — Binärdatei, verschwindet gern
- alles unter `.github/` — Ordner mit Punkt werden oft nicht mitgenommen
- das Ausführbar-Flag von `gradlew` — der Workflow setzt es selbst

Der Preflight-Schritt listet jetzt den kompletten Repo-Inhalt und nennt jede
fehlende Datei beim Namen.

**Besser gleich per Git hochladen:**

```bash
tar -xzf donutmarket-repo.tar.gz
cd donutmarket
git init -b main
git add -A
git commit -m "DonutMarket"
git remote add origin https://github.com/DEINNAME/donutmarket.git
git push -u origin main --force
```

### Zwei build.yml im Repo

Wenn du über die Weboberfläche eine eigene `build.yml` angelegt hast und dann
meine dazu hochgeladen wurde, laufen womöglich zwei Workflows. Unter
`.github/workflows/` sollten genau zwei Dateien liegen: `build.yml` und
`release.yml`. Alles andere löschen.

### Loom-Snapshot

`gradle.properties` steht auf `loom_version=1.17-SNAPSHOT`. Snapshots können
sich ändern oder verschwinden. Scheitert **Schritt B** mit einer Meldung über
`net.fabricmc.fabric-loom`, dann auf der Seite
<https://fabricmc.net/develop/> die aktuell empfohlene Loom-Version nachsehen
und in `gradle.properties` eintragen.

## Dependabot-PRs

Solange `main` rot ist, scheitern die Dependabot-PRs am **selben** Grundproblem
— sie sind kein zusätzlicher Fehler. Erst `main` grün bekommen, dann die PRs
anschauen. `.github/dependabot.yml` unterdrückt Major-Sprünge inzwischen, damit
nicht bei jedem Push ein Schwung roter PRs entsteht.

## Was du mir schicken solltest

Den Text des roten Schritts. Bei Schritt C reichen die Zeilen, die mit
`error:` beginnen — dort steht Datei und Zeilennummer.
