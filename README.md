# Betriebsstatus

Quelltext von <https://status.werkgymnasium.eu> — der Statusseite für das
Kollegium. Sie beantwortet eine einzige Frage: **Liegt der Ausfall an mir oder
an allen?**

Erzeugt mit [Upptime](https://upptime.js.org) (Vorlage v1.44.0). Es gibt keinen
Server: Die Prüfungen laufen alle fünf Minuten in GitHub Actions, die Ergebnisse
landen als Commits in diesem Repo, die Seite liegt auf GitHub Pages.

**Das ist der Punkt.** Eine Statusseite, die auf der überwachten Anlage läuft,
ist genau dann weg, wenn man sie braucht. Diese nicht.

## Dieses Repo muss öffentlich sein

Upptime veröffentlicht Statusseiten nur aus öffentlichen Repos — die Seite holt
ihre Daten zur Laufzeit über die öffentliche GitHub-API. Außerdem sind Actions
nur für öffentliche Repos unbegrenzt kostenlos; der Fünf-Minuten-Takt verbraucht
rund 3.000 Minuten im Monat und würde das Freikontingent von 2.000 sprengen.

Daraus folgt eine Regel für die Konfiguration:

> **Die Liste der überwachten Adressen ist öffentlich — dauerhaft, auch in der
> git-Historie.** Hier gehören nur Dienste hinein, die das Kollegium ohnehin mit
> Namen aufruft. Nichts Internes: keine Verwaltungsoberflächen, keine
> Datenbanken, keine Backup-Ziele.

Das ist dieselbe Regel wie im Repo `anleitungen`, aber sie gilt hier nicht
automatisch mit — sie muss bei jedem neuen Eintrag neu bedacht werden. Ein
Fehlgriff lässt sich nicht durch einen Commit zurücknehmen.

Wer etwas Internes überwachen will, braucht dafür ein anderes Werkzeug auf
eigener Hardware (Gatus, Uptime Kuma). Upptime kann es ohnehin nicht: Seine
Prüfer laufen bei GitHub und kommen nur an das, was öffentlich erreichbar ist.

## In Betrieb nehmen

1. Öffentliches Repo `weg_status` auf GitHub anlegen, dieses hier pushen.
2. **Settings → Actions → General → Workflow permissions** auf _Read and write_
   stellen. Ohne das kann Upptime seine Messwerte nicht committen.
3. DNS: `status` als CNAME auf `<benutzer>.github.io` — **vor** dem nächsten
   Schritt. Sobald die Seite steht, leitet Pages die `github.io`-Adresse
   dauerhaft auf die eigene Domain um; zeigt die noch woandershin, ist die
   Seite unter beiden Adressen unerreichbar.
4. Bei Actions **Uptime CI** von Hand auslösen (*Run workflow*), danach
   **Static Site CI**. Die Reihenfolge zählt: Die Seite braucht die Messwerte
   aus dem ersten Lauf. Erst dabei entsteht der Branch `gh-pages`.
5. **Settings → Pages** auf Branch `gh-pages`, Verzeichnis `/ (root)`. Vorher
   steht der Branch nicht zur Auswahl — das ist kein Fehler.

*Setup CI* ist dafür **nicht** nötig. Der Workflow stößt die anderen nur an und
beginnt mit dem Vorlagen-Abgleich, der hier abgeschaltet ist (siehe unten). Die
beiden Läufe oben von Hand auszulösen führt zum selben Ergebnis.

## Was die Seite nicht kann

**Der Fünf-Minuten-Takt ist eine Absichtserklärung.** GitHubs Cron kennt fünf
Minuten als kürzestes Intervall, hält es bei Last aber regelmäßig nicht ein.
Verzögerungen von zehn Minuten und mehr sind normal. Für „läuft der Chat?"
reicht das; als Alarmierung taugt es nicht.

**Geplante Workflows schaltet GitHub nach 60 Tagen ohne Commit ab.** Upptime
committet im Normalbetrieb ständig selbst und hält sich damit am Leben. Stockt
es aber einmal, stirbt der Zeitplan lautlos — und eine unbemerkt eingefrorene
Statusseite ist schlimmer als gar keine. Einmal im Quartal nachsehen, ob die
Zeitstempel frisch sind.

**Die Prüfung kommt aus dem Internet.** Hängt der Anschluss der Schule, sitzt
das Kollegium im Haus vor toten Diensten, während hier alles grün steht — die
Dienste _sind_ ja erreichbar, nur nicht von dort. Deshalb steht dieser Fall im
Vorspann der Seite ausdrücklich drin.

## Benachrichtigungen

Sind nicht eingerichtet. Upptime kann das über Repository-Secrets (Mail, Matrix
per Webhook, ntfy und andere); ohne sie legt es bei einem Ausfall lediglich ein
GitHub-Issue an. Für den Anfang reicht das.

## Änderungen

Im Normalfall nur [`.upptimerc.yml`](.upptimerc.yml) anfassen. Die
Verzeichnisse `api/`, `graphs/` und `history/` legt Upptime selbst an.

## Der Vorlagen-Abgleich ist abgeschaltet

In [`.templaterc.json`](.templaterc.json) steht eine leere Dateiliste. Damit
holt Upptime **keine Änderungen mehr aus der Vorlage**, und die Dateien unter
`.github/workflows/` bleiben so, wie sie hier liegen.

Das ist Absicht, und der Grund ist eine harte Grenze von GitHub: Der
eingebaute `GITHUB_TOKEN` **darf Dateien unter `.github/workflows/`
grundsätzlich nicht schreiben** — die Berechtigung `workflows` gibt es für ihn
nicht, damit ein kompromittierter Workflow sich nicht selbst umschreiben kann.
Der Abgleich scheiterte deshalb zuverlässig mit:

```
refusing to allow a GitHub App to create or update workflow
`.github/workflows/response-time.yml` without `workflows` permission
```

Und weil er in *Setup CI* der erste Schritt ist, riss er alles Folgende mit —
auch den Bau der Seite.

Der dokumentierte Ausweg wäre ein Personal Access Token mit
`workflows: write` als Secret `GH_PAT`. Dagegen sprach die Ablaufzeit:
Fine-grained Tokens gelten höchstens ein Jahr, und wenn einer ausläuft, hört
die Statusseite **still** auf sich zu aktualisieren. Das ist genau der
Fehlerfall, der weiter oben als „schlimmer als gar keine Seite" steht — für
den Gegenwert einer automatischen Vorlagenpflege zu teuer.

Laut Upptimes Maintainer ist das auch der einzige Teil, der den Token braucht:

> All functionality works perfectly fine when using the GitHub token. The only
> real reason the PAT is necessary is because of the template update action.

**Was das im Alltag bedeutet:** Bei einer neuen Upptime-Version die Workflows
von Hand nachziehen. Der aktuelle Stand ist **v1.44.0**.

```bash
git remote add vorlage https://github.com/upptime/upptime.git   # einmalig
git fetch vorlage
git checkout vorlage/master -- .github/
git diff --cached                                              # ansehen!
```

Der Blick auf den Diff ist nicht optional: Die Workflows erzeugt Upptime aus
`.upptimerc.yml`, und was von dort kommt, läuft danach mit den Rechten dieses
Repos. Ein- oder zweimal im Jahr reicht dafür völlig.
