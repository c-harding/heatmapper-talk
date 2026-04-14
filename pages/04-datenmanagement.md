---
layout: chapter
title: Datenmanagement unter realen Constraints
---

Von LocalStorage zu IndexedDB

<!--
  Die Karte sieht gut aus – aber wie halten wir die Daten dahinter aktuell?
  In diesem Kapitel geht es um die weniger sichtbare Seite des Projekts: wie Daten effizient geladen,
  zwischengespeichert und im Browser verwaltet werden.
  Und wo die Grenzen des Browsers dabei liegen.
-->

---
title: Das Problem mit Delta-Updates
inner-split: 50
---

### ❌ Naiver Ansatz: alles neu laden

```js
async function loadActivities() {
  return await fetchStrava(
    '/api/strava/activities'
  )
}
```

<v-click>

- Strava-API: max. **200 Aktivitäten pro Request**
- Bei 1.000 Aktivitäten → **5 Requests, ~30 Sekunden**
- Läuft schneller gegen **Rate Limits** (600 Req./15 Min, 30.000/Tag)
- Und das, obwohl wir schon die Aktivitäten vom letzten Mal im LocalStorage haben!

</v-click>

::right::

<v-click>

### ✅ Ziel: nur laden, was neu ist

```js
async function loadActivities(lastFetch) {
  return await fetchStrava(
    '/api/strava/activities',
    { after: lastFetch }
  )
}
```

</v-click>

<v-click>

**Was müssen wir dafür speichern?**

- Alle bereits geladenen Aktivitäten im Browser
- Das **Datum der letzten Abfrage** (`lastFetch`)

→ Beim nächsten Aufruf: nur `after: lastFetch` anfragen

</v-click>

<!--
  Wir haben unsere Aktivitäten im LocalStorage – aber wie hält man den Cache aktuell?
  Der naive Ansatz: bei jedem Aufruf alles neu laden.

  [click] Das Problem: Die Strava-API liefert maximal 200 Aktivitäten pro Request.
  Bei jemandem mit vielen Aktivitäten sind das mehrere Requests, mehrere Sekunden – und das frisst die Rate Limits auf.
  600 Requests in 15 Minuten klingen viel, aber wenn man debuggt oder viel testet, ist das schnell aufgebraucht.

  [click] Die bessere Lösung: ein Delta-Update.
  Strava's Aktivitäten-Endpoint unterstützt einen `after`-Parameter – einen Unix-Timestamp.
  Man fragt also nur ab, was seit der letzten Abfrage neu dazugekommen ist.

  Dafür brauchen wir zwei Dinge im LocalStorage: die Aktivitäten selbst, und den Zeitstempel der letzten Abfrage.
  Beim nächsten Aufruf reicht dann ein einziger Request für die neuen Aktivitäten.
-->


---
title: Das Problem mit dem Zeitstempel
---

`after: lastFetch` klingt einfach —
ist es aber nicht.

<v-click>

### IDs ≠ Datum

Die API sortiert nach **ID**, nicht nach Datum.

- IDs werden per Increment vergeben
- Aber: Aktivitäten können **nachträglich hochgeladen** werden
- Eine alte Radtour, heute importiert → hohe ID, altes Datum

</v-click>

<v-click>

### Das Risiko

Wenn `after: lastFetch`:
- Neue Imports von heute: ✅
- Gestrigen Lauf, wenn Garmin-Sync gestern noch nicht abgeschlossen war: ❌ verpasst
- Radtour von vor einer Woche, heute rückwirkend importiert: ❌ verpasst

</v-click>

<!--
Der Delta-Ansatz hat einen Haken: die API sortiert nach ID, nicht nach Datum.

  [click] IDs sind zwar inkrement, aber sie spiegeln nicht das tatsächliche Aktivitätsdatum wider.
  Wer eine alte Radtour heute von einem Garmin importiert, bekommt eine neue, hohe ID – aber das Datum liegt Jahre zurück.

  [click] Wenn wir also nur `after: lastFetch` verwenden, gibt es zwei Fallen.
  Erstens: eine Tour, die vor lastFetch stattgefunden hat, aber erst danach zu Strava hochgeladen wurde – zum Beispiel weil die Sync von Garmin noch nicht abgeschlossen war, als wir die Seite geladen haben.
  Zweitens: eine Aktivität von vor einer Woche, die heute rückwirkend importiert wird.
-->

---
title: Die Heuristik
---

```js
async function loadActivities(newestActivity) {
  return await fetchStrava(
    '/api/strava/activities',
    { after: newestActivity.date - ONE_DAY }
  )
}
```

Statt `lastFetch` verwenden wir das Datum der **neuesten bekannten Aktivität** minus einen Tag –
ein kleines Überlappungsfenster, das nachträgliche Uploads abfängt.

<v-click>

**Duplikate** werden beim Mergen anhand der ID dedupliziert.

</v-click>

<v-click>

**Wenn die Heuristik versagt:** Der Lade-Button wird durch einen Neu-Laden-Button ersetzt – der Nutzer kann den Cache manuell leeren und alles neu abfragen.

</v-click>

<!--
  Die Lösung: statt des Zeitstempels der letzten Abfrage verwenden wir das Datum der neuesten bekannten Aktivität – und ziehen einen Tag ab.
  Das erzeugt ein kleines Überlappungsfenster, das nachträgliche Uploads mit hoher Wahrscheinlichkeit abfängt.

  [click] Duplikate, die durch das Überlappen entstehen, werden beim Mergen einfach anhand der ID dedupliziert.

  [click] Und wenn die Heuristik doch mal nicht reicht – also eine Aktivität trotzdem fehlt –
  wird der Lade-Button durch einen Neu-Laden-Button ersetzt.
  Der Nutzer kann dann den Cache leeren und alles sauber neu abfragen.
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

- Die Heuristik funktioniert hier nicht
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
inner-split: 50
---

Nach mehreren Jahren und vielen Aktivitäten: **localStorage voll.**

`localStorage` hat ein Limit von **~5 MB** pro Domain.

<v-click>

### Lösung: IndexedDB

- Echte Datenbank im Browser
- Kein festes Größenlimit (nutzt Festplattenplatz)
- Unterstützt strukturierte Objekte & Indizes
- Abstraktion über `localforage`

</v-click>

::right::

<v-click>

### Die Migration

```js
// Vorher
localStorage.setItem('activities',
  JSON.stringify(activities))

// Nachher
await localforage.setItem(
  'activities', activities)
```

API fast identisch – aber asynchron.

</v-click>

<v-click>

### Nachteil: Debugging

- `localStorage` ist direkt in der Browser-Konsole lesbar.
- `localforage` ist nur im Code mit der Library nutzbar.
- Chrome DevTools zeigt die IndexedDB-Inhalte an, aber: keine Queries, keine Variablenzuweisung.

</v-click>

<!--
  Nach mehreren Jahren und vielen hundert Aktivitäten hat mich die nächste Grenze eingeholt.
  Zum Glück bin ich selbst ein Power-User meines eigenen Tools – das Problem trat erst bei rund 3.000 Aktivitäten über mehr als 10 Jahre auf.
  Die meisten Nutzer hätten das nie erreicht.

  Der Fehler war ein QuotaExceededError beim Synchronisieren – aber weil ich dort keine ordentliche Fehlerbehandlung eingebaut hatte, ist er still untergegangen.
  Aufgefallen ist er erst beim nächsten Seitenaufruf: die neusten Aktivitäten waren einfach nicht da.
  `localStorage` hat ein Limit von ungefähr 5 MB – das variiert allerdings je nach Browser.

  [click] Die Lösung: IndexedDB – eine echte Datenbank im Browser, ohne hartes Größenlimit.
  Ich nutze `localforage` als Abstraktionsschicht, die eine ähnliche API wie localStorage bietet.

  [click] Die Migration war überraschend einfach: die API ist fast identisch, nur asynchron.
  Ein paar `await` mehr, und die Daten landen in IndexedDB statt im localStorage.

  Ich hatte eh schon ein Konzept für Datenmigrationen eingebaut, um bei zukünftigen Änderungen die Datenstruktur anpassen zu können – das hat mir hier sehr geholfen.

  [click] Aber es gibt einen echten Nachteil beim Debugging.
  Mit localStorage konnte ich im Browser direkt `localStorage.getItem('activities')` aufrufen, Daten in Variablen speichern, manuell filtern.
  Mit localforage geht das nicht – die Library ist in der DevTools-Konsole nicht verfügbar.
  Chrome zeigt zwar die IndexedDB-Inhalte an, aber man kann keine Queries ausführen und nichts in Variablen speichern.
  Das macht das manuelle Debuggen deutlich mühsamer.
-->
