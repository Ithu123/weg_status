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

## Noch zu tun

In [`.upptimerc.yml`](.upptimerc.yml) stehen noch fünf Platzhalter — Dateien, Mail,
Vibe, BenotPDF und Moodle:

```bash
grep -n BITTE-EINTRAGEN .upptimerc.yml
```

`.invalid` ist eine reservierte Endung und löst garantiert nicht auf. Ein
vergessener Platzhalter fällt deshalb sofort rot auf, statt stillschweigend
grün zu bleiben.

## In Betrieb nehmen

1. Öffentliches Repo `weg_status` auf GitHub anlegen, dieses hier pushen.
2. **Settings → Actions → General → Workflow permissions** auf _Read and write_
   stellen. Ohne das kann Upptime seine Messwerte nicht committen.
3. **Settings → Pages** auf den Branch `gh-pages` stellen (entsteht beim ersten
   Lauf von _Static Site CI_).
4. DNS: `status` als CNAME auf `<benutzer>.github.io`.
5. Bei Actions _Uptime CI_ und _Static Site CI_ einmal von Hand auslösen
   (**Run workflow**), sonst wartet man bis zum nächsten Zeitplan.

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

Nur [`.upptimerc.yml`](.upptimerc.yml) anfassen. Die Dateien unter
`.github/workflows/` werden aus der Vorlage erzeugt und wöchentlich
überschrieben — Änderungen daran sind beim nächsten Abgleich weg.

Die Verzeichnisse `api/`, `graphs/` und `history/` legt Upptime selbst an.
