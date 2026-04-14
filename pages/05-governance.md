---
layout: chapter
title: Governance, Compliance & externe Abhängigkeiten
---

Wenn externe Plattformen die Regeln ändern

<!--
  Bis hierher war das Projekt hauptsächlich ein technisches Puzzle.
  Aber Software existiert nicht im Vakuum – externe Plattformen ändern ihre Regeln,
  Rechtsstreitigkeiten haben Auswirkungen auf API-Konsumenten, und Compliance-Anforderungen
  treffen auch Hobby-Projekte.
  In diesem Kapitel zeige ich, was passiert, wenn man plötzlich nicht mehr nur Entwickler ist,
  sondern auch Vertragspartner einer Plattform.
-->

---
title: Die Strava API Review
---

Strava kündigt an: **alle API-Apps müssen validiert werden.**

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
  Irgendwann kam von Strava die Ankündigung: alle API-Apps müssen validiert werden.
  Das betrifft nicht nur große Firmen, sondern auch kleine Hobby-Projekte wie meines.

  [click] Der Prozess: App einreichen, Strava prüft, ob die Guidelines eingehalten werden.
  Wer nicht besteht, verliert den API-Zugriff.

  [click] Ich habe das zunächst nicht so ernst genommen – ein Hobby-Projekt mit einer Handvoll Nutzer.
  Aber als die Frist näher rückte, habe ich mir das doch angeschaut.
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

<img v-click="3" src="/media/updated-sidebar-header.png" class="w-full object-contain">


::right::

Die wichtigste Änderung: **offizielle Strava-Assets** verwenden.

<v-click>

- Eigene "Verbinden mit Strava"-Buttons: ✘
- Offizielle Bilder von Strava: ✓
- Farbgebung, Größe und Platzierung vorgegeben

</v-click>

<v-click>

Relativ einfach umzusetzen, aber kostet trotzdem Zeit, wenn man es nicht von Anfang an richtig macht.

</v-click>

<!--
  Die größte technische Änderung war das Branding.
  Ich hatte eigene Buttons gebaut – das ist laut Guidelines nicht erlaubt.
  Strava stellt offizielle Assets bereit: Bilder mit genau definierter Gestaltung.

  [click] Das bedeutet: kein selbstgebauter Button mehr, sondern das offizielle Strava-Bild einbinden.
  Farbe, Größe, Abstände – alles vorgegeben.

  [click] Technisch kein großes Problem – aber ein guter Reminder, dass man bei API-Integrationen
  von Anfang an die Branding-Guidelines lesen sollte.
  Ich hatte es nicht getan.
-->

---
title: Webhooks & Token-Revocation
inner-split: 55
---

Erstes Einreichen: **nicht bestanden.**

Grund: fehlende Webhook-Integration.

<v-click>

### Warum Webhooks?

Wenn ein Nutzer den App-Zugriff in der Strava-UI widerruft,
müssen alle gespeicherten Daten gelöscht werden.

</v-click>

<v-click>

### Mein Argument

Ich speichere **keine personenbezogenen Daten** am Backend –
nur ein Session-Token-Mapping.

Die Aktivitäten liegen im Browser des Nutzers.

</v-click>

::right::

<v-click>

### Die Lösung

```js
// Webhook-Handler
app.post('/webhook', (req, res) => {
  const { owner_id, aspect_type } = req.body
  if (aspect_type === 'deauthorize') {
    deleteSession(owner_id)
  }
  res.sendStatus(200)
})
```

Token löschen, wenn Zugriff widerrufen wird.

</v-click>

<v-click>

Zusätzlich: beim nächsten Laden prüfen, ob der Token noch gültig ist.
Falls nicht: lokalen Cache löschen.

</v-click>

<!--
  Die eigentliche Hürde war nicht das Branding – sondern Webhooks.
  Strava erwartet, dass Apps auf Deauthorisierungs-Events reagieren.

  [click] Der Gedanke dahinter: wenn ein Nutzer den Zugriff entzieht, müssen alle seine Daten gelöscht werden.
  Fair enough – aber in meinem Fall etwas akademisch.

  [click] Denn ich speichere keine personenbezogenen Daten am Backend – nur das Mapping von Session-Token auf Auth-Token.
  Die Aktivitäten selbst liegen im Browser des Nutzers.

  [click] Trotzdem habe ich den Webhook eingebaut: wenn Strava meldet, dass ein Nutzer den Zugriff widerrufen hat,
  lösche ich den ohnehin ungültigen Token vom Server.

  [click] Und im Frontend: beim Laden einmal prüfen, ob der Token noch gültig ist.
  Falls nicht, den lokalen Cache leeren – damit der Nutzer keine gecachten Daten mehr sieht.
  Damit habe ich die Prüfung bestanden.
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
  Ende 2025 gab es wieder Post von Strava – diesmal mit größerem Hintergrund.
  Strava und Garmin hatten sich verklagt.
-->

---
title: Strava & Garmin — Rechtsstreit mit Folgen
---

Ende 2025: **Strava und Garmin verklagen einander.**

<v-click>

Das Ergebnis: Garmin führt neue Developer Guidelines ein.
Diese gelten für alle Endabnehmer von Garmin-Daten — also für **Abnehmer der Strava-API**.

</v-click>

<v-click>

### Die Anforderung

Aktivitäten, die von einem Garmin-Gerät aufgezeichnet wurden,
müssen explizit als solche **gekennzeichnet** werden, am besten mit dem Gerätenamen.

</v-click>

<v-click>

### Das Problem

Die Strava-API lieferte diese Information **nicht** –
außer über einen separaten Endpunkt pro Aktivität.  
Bei 3.000 Aktivitäten: **3.000 zusätzliche Requests.**

</v-click>

<v-click>

### Die Auflösung

Strava hat das Feld in den Übersichtsdaten ergänzt, allerdings erst **eine Woche vor der Frist.**
Ich musste also kurzfristig umbauen, um die neue Information konform darzustellen.

</v-click>

<!--
  Kurz danach kam die Meldung: Strava und Garmin haben sich geeinigt.
  Als Teil dieser Einigung hat Garmin neue Developer Guidelines eingeführt.

  [click] Diese gelten nicht nur für Garmin-Entwickler, sondern auch für alle, die die Strava-API nutzen.
  Denn Strava-Daten können von Garmin-Geräten stammen.

  [click] Die konkrete Anforderung: wenn eine Aktivität von einem Garmin aufgezeichnet wurde,
  muss das in der App gekennzeichnet sein.

  [click] Das Problem: die Übersichts-API von Strava lieferte diese Information nicht.
  Man hätte für jede Aktivität einen eigenen Detail-Request machen müssen.
  Bei mir: rund 3.000 Requests. Das war keine Option.

  [click] Strava hat das Feld dann doch noch in den Übersichtsdaten ergänzt – aber erst eine Woche vor der Frist.
  Vermutlich nur weil ich, und viele andere Entwickler auch, den Support kontaktiert hatten.
-->
