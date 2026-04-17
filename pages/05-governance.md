---
layout: chapter
title: Governance, Compliance & externe Abhängigkeiten
shortTitle: Governance
sectionDuration: 3m30s
---

Wenn externe Plattformen die Regeln ändern

<!--
Jetzt wird es weniger technisch: Was passiert, wenn die Plattform die Regeln ändert –
  und man plötzlich nicht mehr nur Entwickler ist, sondern auch Vertragspartner?
-->

---
title: Die Strava API Review
---

Strava kündigt an: **Alle API-Apps müssen validiert werden.**

<v-click>

- Ziel: Sicherstellung, dass API-Guidelines eingehalten werden

- App muss eingereicht und geprüft werden

- Kein Bestehen = kein weiterer API-Zugriff

</v-click>
<v-click>

Für ein Hobby-Projekt mit wenigen Nutzern: **zunächst ignoriert.**

Bis die Frist näher rückte.

</v-click>

<!--
Irgendwann ist von Strava die Ankündigung gekommen: alle API-Apps müssen validiert werden.
  Das betrifft nicht nur große Firmen, sondern auch kleine Hobby-Projekte wie meines.

  [click] Der Prozess: App einreichen, Strava prüft, ob die Guidelines eingehalten werden.
  Wer nicht besteht, verliert den API-Zugriff.

  [click] Ich habe das zunächst nicht so ernst genommen – ein Hobby-Projekt mit einer Handvoll Nutzer.
  Aber als die Frist näher gerückt ist, habe ich mir das doch angeschaut.
-->

---
title: Branding-Vorgaben
split: 30
title-side: right
left:
  hideHeader: true
  hideFooter: true
  class: bg-white! p-0!
right:
  full-color: true
---

<img v-click="2" src="/media/updated-sidebar-header.png" class="w-full object-contain">

::right::

Die wichtigste Änderung: **offizielle Strava-Assets** verwenden.

<v-click>

- Eigene "Verbinden mit Strava"-Buttons: ✘
- Offizielle Bilder von Strava: ✓
- Farbgebung, Größe und Platzierung vorgegeben

</v-click>

<!--
Die größte technische Änderung war das Branding.
  Ich hatte eigene Buttons gebaut – das ist laut Guidelines nicht erlaubt.
  Strava stellt offizielle Assets bereit.

  [click] Kein selbstgebauter Button mehr, sondern das offizielle Strava-Bild einbinden.
  Farbe, Größe, Abstände – alles vorgegeben.

  [click] So hat das Login danach ausgesehen – mit dem offiziellen Strava-Button und „Powered by Strava“.
-->

---
title: Webhooks & Token-Revocation
---

Erstes Einreichen: **nicht bestanden.**

Grund: fehlende Webhook-Integration.

<v-click>

### Warum Webhooks?

Wenn ein Nutzer den App-Zugriff in der Strava-UI widerruft,
müssen alle gespeicherten Daten gelöscht werden.

</v-click>
<v-click>

### Fazit

Ich speichere **keine personenbezogenen Daten** am Backend –
nur ein Session-Token-Mapping. Die Aktivitäten liegen nur im Browser.

Trotzdem: Webhook eingebaut → ohnehin ungültigen Token löschen.
Im Frontend: Token-Gültigkeit prüfen, bei Bedarf Cache leeren.

</v-click>

<!--
Das Ganze hatte aber nicht gereicht.
  Die eigentliche Hürde war nicht das Branding – sondern Webhooks.
  Strava erwartet, dass Apps auf Deauthorisierungs-Events reagieren.

  [click] Wenn ein Nutzer den Zugriff entzieht, müssen alle Daten gelöscht werden.
  In meinem Fall etwas akademisch – ich speichere keine personenbezogenen Daten am Backend.

  [click] Trotzdem habe ich den Webhook eingebaut: bei Widerruf wird der ohnehin ungültige Token gelöscht.
  Im Frontend prüfe ich beim Laden die Token-Gültigkeit. Damit habe ich das Audit bestanden.
-->

---
title: Strava & Garmin — Rechtsstreit mit Folgen
title-side: right
split: 50
left:
  image: /media/patent-klage.png
  class: bg-contain bg-transparent!
right:
  full-color: true
---

::header::

<span/>

::right::

<p class="!mt-auto">
    <EndLink to="heise.de/news/Patentverletzungsklage-von-Strava-gegen-Garmin-wegen-Segments-und-Heatmaps-10712222.html" />
</p>

<!--
Aber es war nicht das letzte Mal, dass Strava die Spielregeln geändert hat.
  Ende 2025 gab es wieder Post – diesmal mit größerem Hintergrund.
  Strava und Garmin hatten sich gegenseitig verklagt.
-->

---
title: Strava & Garmin — Rechtsstreit mit Folgen
---

Ende 2025: **Strava und Garmin verklagen einander.**

<v-click>

Das Ergebnis: Garmin führt neue Developer Guidelines ein.
Diese gelten für alle Endabnehmer von Garmin-Daten — also auch für **Abnehmer der Strava-API**.

</v-click>
<v-click>

### Die Anforderung

Aktivitäten, die von einem Garmin-Gerät aufgezeichnet wurden,
müssen explizit als solche **gekennzeichnet** werden, am besten mit dem Gerätenamen.

</v-click>
<v-click>

### Umsetzung

Die Strava-API hatte den Gerätenamen nicht in den Übersichtsdaten geliefert — nur über einen separaten Endpunkt pro Aktivität.
Bei 3.000 Aktivitäten: **3.000 zusätzliche Requests.**

Strava hat das Feld erst **eine Woche vor der Frist** in den Übersichtsdaten ergänzt.

</v-click>

<!--
Kurz danach ist die zweite Meldung gekommen: Strava und Garmin haben sich geeinigt.
  Als Teil dieser Einigung hat Garmin neue Developer Guidelines eingeführt.

  [click] Diese gelten auch für alle, die die Strava-API nutzen.
  Denn Strava-Daten können von Garmin-Geräten stammen.

  [click] Die Anforderung: Garmin-Aktivitäten müssen in der App gekennzeichnet werden.

  [click] Das Problem: die Übersichts-API hat diese Info nicht geliefert – nur ein Detail-Endpunkt pro Aktivität.
  Bei 3.000 Aktivitäten keine Option.
  Strava hat das Feld dann doch ergänzt – aber erst eine Woche vor der Frist.
-->
