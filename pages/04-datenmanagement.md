---
layout: chapter
title: Datenmanagement unter realen Constraints
shortTitle: Datenmanagement
sectionDuration: 6m
---

Von LocalStorage zu IndexedDB

<!--
So viel zur Darstellung. Aber eine schöne Karte bringt nichts ohne aktuelle Daten.
  Und genau da wird es tricky: Caching, Delta-Updates und die Grenzen des Browsers.
-->

---
title: Delta-Updates
inner-split: 50
---

### ❌ Naiver Ansatz: alles neu laden

````md magic-move {at:2}
```js
async function loadActivities() {
  return await fetchStrava('/api/strava/activities');
}
```

```js
async function loadActivities(lastSync) {
  return await fetchStrava('/api/strava/activities', { after: lastSync });
}
```
````

::right::

<v-click>

- Strava-API: max. **200 Aktivitäten pro Request**
- Bei 1.000 Aktivitäten → **5 Requests, ~30 Sekunden**
- Stößt schneller an **Rate Limits** (600 Req./15 Min, 30.000/Tag)
- Und das, obwohl wir schon die Aktivitäten vom letzten Mal im LocalStorage haben!

</v-click>
<v-click>

### ✅ Ziel: nur laden, was neu ist

**Was müssen wir dafür speichern?**

- Alle bereits geladenen Aktivitäten im Browser
- Das **Datum der letzten Synchronisation** (`lastSync`)

→ Beim nächsten Aufruf: nur `after: lastSync` anfragen

</v-click>

<!--
Wir haben unsere Aktivitäten im LocalStorage – aber wie hält man den Cache aktuell?
  Der naive Ansatz: bei jeder Sync alles neu laden.

  [click] Das Problem: Die Strava-API liefert maximal 200 Aktivitäten pro Request.
  Bei vielen Aktivitäten sind das mehrere langsame Requests – und das frisst die Rate Limits auf.

  [click] Die bessere Lösung: ein Delta-Update.
  Die API unterstützt einen `after`-Parameter – man fragt also nur ab, was seit der letzten Synchronisation neu dazugekommen ist.
-->

---
title: Das Problem mit dem Zeitstempel
split: 57
clicks: 6
right:
  hideFooter: true
  class: bg-white
---

`after: lastSync` klingt einfach —
ist es aber nicht.

<v-click at="+3">

### Problem: Upload ≠ Aktivität

Die API filtert nach **Aktivitätsdatum**, nicht nach Upload-Datum.

→ Verspätete Uploads fallen durch den Filter.

</v-click>
<v-click>

### Besser: `after: newestActivity`

Das Startdatum der neuesten bekannten Aktivität statt `lastSync`.

</v-click>
<v-click>

### Kein perfekter Filter möglich

Ältere Uploads bleiben unsichtbar.

</v-click>
<v-click>

**Fallback:** Neuladen-Button löscht den Cache.

</v-click>

::right::

<TimestampDiagram
  :step="$clicks"
  :activities="[
    { label: 'Lauf 1', start: 1.2, end: 1.4, uploaded: 1.5 },
    { label: 'Lauf 2', start: 2.3, end: 2.6, uploaded: 2.7 },
    { label: 'Radtour', start: 3.2, end: 3.6, uploaded: 3.8 },
    { label: 'Alte Tour', start: 0.2, end: 0.9, uploaded: 3.8, showAt: 5 },
  ]"
  :syncs="$clicks < 6 ? [[0, 1.8], [1, 2.8], [2, 3.7], [3, 3.9]] : [[6, 3.9]]"
  :maxTime="5"
  :filterMode="$clicks >= 4 ? 'newestActivity' : 'lastSync'"
/>

<!--
`after: lastSync` klingt nach einer einfachen Lösung – aber in der Praxis steckt der Teufel im Detail.
  Schauen wir uns ein konkretes Beispiel an.

  Am Dienstag bin ich laufen gegangen, habe die Aktivität anschließend zu Strava importiert, und dann meine Daten in den Heatmapper synchronisiert.

  [click] Nach meinem nächsten Lauf am Mittwoch synchronisiere ich nochmal. Lauf 1 wird nicht erneut geladen – der ist ja schon im Cache.

  [click] Am nächsten Tag nach einer Radtour wird es problematisch. Wir synchronisieren, aber die Daten wurden noch nicht in Strava importiert.
  
  [click] Und wenn wir es kurz danach nochmal versuchen, klappt es trotzdem weiterhin nicht: der Filter basiert auf dem Aktivitätszeitstempel, nicht auf dem Upload-Zeitstempel. Die Radtour liegt vor der letzten Synchronisation – also wird sie nie abgefragt.

  [click] Das lässt sich verbessern: statt `lastSync` verwenden wir das Startdatum der neuesten bereits importierten Aktivität. So laden wir zumindest ab dem Zeitpunkt, an dem wir zuletzt etwas gesehen haben. Aber auch das ist nicht perfekt.

  [click] Als ich meinen Fahrradcomputer mit dem Handy verbunden habe, um die Radtour zu importieren, ist mir aufgefallen, dass ich eine ältere Tour vom Montag noch nicht importiert hatte. Auch diese wird nie geladen, weil sie älter ist als die neueste Aktivität.

  [click] Es gibt keinen perfekten Filter – ältere nachträgliche Uploads lassen sich nicht zuverlässig erkennen.
  Die pragmatische Lösung: ein Neuladen-Button, der den Cache löscht.
  In der Praxis nutzt man das nur selten – nur wenn man neulich alte Aktivitäten nachträglich hochgeladen hat.
-->

---
title: Sonderfall – Routen
inner-split: 50
---

Strava kann nicht nur Aktivitäten aufzeichnen – man kann auch Routen planen.

Ich wollte diese ebenfalls in Heatmapper anzeigen.

<v-click>

### Das Problem

Die Routen-API hat **keinen `after`-Parameter** – weder nach Erstelldatum noch nach Änderungsdatum filterbar.

</v-click>

<v-click>

### Die Konsequenz

- Delta-Filterung ist hier nicht möglich
- Bei jeder Synchronisation: **alle Seiten neu abfragen**
- Routen ändern sich selten → akzeptabler Kompromiss

</v-click>

::right::

<v-click>

### Was wir tun können

Routen und Aktivitäten **getrennt cachen** und getrennt synchronisieren.

```js
localStorage.setItem('activities', ...)
localStorage.setItem('routes', ...)

// Aktivitäten: Delta-Update möglich
await syncActivities()

// Routen: immer komplett neu laden
await syncRoutes()
```

</v-click>

<!--
Neben Aktivitäten bietet Strava auch Routen-Planung – und ich wollte die ebenfalls auf der Karte sehen.
  Klingt einfach, aber die API macht hier einen Strich durch die Rechnung.

  [click] Der Routen-Endpoint unterstützt keinen Zeitfilter: weder nach Erstelldatum noch nach letzter Änderung.
  Es gibt nur Pagination.
  Das bedeutet: unsere schöne Delta-Update-Logik greift hier überhaupt nicht.

  [click] Die Konsequenz ist, dass wir bei jeder Synchronisation alle Routen-Seiten neu abfragen müssen.
  Das ist suboptimal – aber da Routen sich selten ändern, ist es in der Praxis ein akzeptabler Kompromiss.

  [click] Die sauberste Lösung: Routen und Aktivitäten getrennt cachen und getrennt synchronisieren.
  Aktivitäten mit Delta-Update, Routen immer komplett.
  Das hält den Code klar und die Verantwortlichkeiten getrennt.
-->

---
title: 5 MB reichen nicht
split: 50
---

Nach mehreren Jahren und vielen Aktivitäten: **localStorage voll.**

`localStorage` hat ein Limit von **~5 MB** pro Domain.

<v-click>

### Lösung: IndexedDB

- Echte Datenbank im Browser
- Kein festes Größenlimit
- Komplexe API,
  - aber `localforage` bietet eine einfache Abstraktion

</v-click>

::right::

<v-click>

### Die Migration

```diff
-  localStorage.setItem('activities',
-    JSON.stringify(activities))

+  await localforage.setItem('activities',
+    activities)
```

</v-click>
<v-click>

<div class="h-4 mx--10 mt-6 mb-2 shrink-0 bg-white" />

### Nachteil: Debugging

- `localStorage` ist direkt in der Browser-Konsole lesbar.
- `localforage` ist nur im Code mit der Library nutzbar.
- Chrome DevTools zeigt die IndexedDB-Inhalte zwar an, aber ohne JavaScript-Zugriff.

</v-click>

<!--
Nach mehreren Jahren hat mich die nächste Grenze eingeholt.
  Das Problem ist erst bei rund 3.000 Aktivitäten über mehr als 10 Jahre aufgetreten — ich bin selbst Power-User von Strava.

  Der Fehler war ein QuotaExceededError beim Synchronisieren, der still untergegangen ist.
  Aufgefallen ist er erst beim nächsten Seitenaufruf: die am neuesten geladenen Aktivitäten haben einfach gefehlt.

  [click] Die Lösung: IndexedDB – eine echte Datenbank im Browser, ohne hartes Größenlimit.
  Ich nutze `localforage` als Abstraktionsschicht.

  [click] Die Migration war überraschend einfach: die API ist fast identisch, nur asynchron.

  [click] Der Nachteil: Mit localStorage konnte ich in der Konsole direkt auf die Daten zugreifen und sie filtern.
  Mit IndexedDB geht das nicht – die Daten sind nur über die Library erreichbar.
-->
