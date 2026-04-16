---
layout: chapter
title: Einstieg & Motivation
sectionDuration: 2m
---

<!--
  Bevor wir in die Technik einsteigen, möchte ich euch zeigen, wo das Ganze angefangen hat – und zwar ganz analog.
-->

---
title: Die analoge Ausgangslage
split: 40
title-side: right
left:
  image: /media/paper-map-closed.jpeg
  class: bg-right
---

::right::

- Kostenlose Landkarte vom Alpenverein (München, 2018)

<!--
  Als ich 2018 nach München gezogen bin, habe ich mich beim Alpenverein angemeldet und dort eine kostenlose Landkarte mitgenommen.
-->

---
title: Die analoge Ausgangslage
split: 40
title-side: right
left:
  image: /media/paper-map.jpeg
---

::right::

- Kostenlose Landkarte vom Alpenverein (München, 2018)
- Jede Wanderung, jeden Lauf, jede Radtour **von Hand eingezeichnet**
- Physische Visualisierung aller Outdoor-Aktivitäten
- Aber: Parallel digitale Aufzeichnung mit Strava

<!--
  Darauf habe ich dann angefangen, jede Wanderung, jeden Lauf, jede Radtour von Hand einzuzeichnen – als eine Art persönliche Heatmap auf Papier.
  Gleichzeitig habe ich aber all diese Aktivitäten auch digital mit Strava aufgezeichnet.
  Die Daten waren also doppelt vorhanden – einmal analog, einmal digital.
-->

---
title: Warum Digitalisierung?
inner-split: 50
---

## Das Problem der analogen Lösung

### ❌ Limitierungen
- Nur ein geografischer Ausschnitt
- Keine Filterung nach Zeit
- Die Karte ist nicht immer dabei
- Es kostet Zeit, jede Aktivität einzuzeichnen
- Die Daten sind eh schon digital vorhanden…

::right::

## &nbsp;

<v-click>

### ✅ Digitale Möglichkeiten
- Globale Darstellung
- Beliebige Zoom-Level
- Immer abrufbar
- Zeitliche Filter
- Automatische Updates

</v-click>

<!--
  Irgendwann wird klar: Die analoge Lösung hat echte Grenzen.
  - Die Karte zeigt nur einen geografischen Ausschnitt – alles außerhalb fehlt.
  - Man kann nicht nach Zeiträumen filtern.
  - Die Karte hat man nicht immer dabei.
  - Und selbstverständlich kostet es Zeit, jede Aktivität einzuzeichnen – obwohl die Daten längst digital vorhanden sind.

  [click] Wenn man das umdreht, sieht man sofort die Vorteile einer digitalen Lösung:
  - eine globale Darstellung
  - beliebige Zoom-Level
  - immer abrufbar
  - zeitliche Filter
  - automatische Updates
-->

---
title: 'Der Wendepunkt: Von analog zu digital'
split: 60
right:
  image: /media/derive.jpg
  pageClass: bg-[position:33%]
  articleClass: text-white flex flex-col justify-end text-right
  full-color: true
---

### Die Erkenntnis

Ich zeichne meine Aktivitäten bereits digital auf — warum nicht auch digital visualisieren?

<v-click>

### Der Auslöser

Entdeckung des Projekts <SmartLink to="github.com/erik/derive">**dérive**</SmartLink> (Open-Source, GPX-basiert)

</v-click>
<v-click>

::blockquote{.font-serif}
**GPX**  
*GPS Exchange Format*. XML-basiertes Dateiformat für GPS-Tracks
::

</v-click>
<v-click>

**Aber:** GPX-Export-Flow war umständlich...

Mit DSGVO sogar noch komplizierter:
Statt schneller Exporte nur Massenexport in einer riesigen Datei

</v-click>

::right::

<v-click :at="1">

(<SmartLink to="https://github.com/erik/derive"><mdi-github/> erik/derive</SmartLink>)

</v-click>

<!--
  Der entscheidende Moment kam, als ich gemerkt habe: Ich zeichne das alles sowieso schon digital auf – warum also nicht auch digital visualisieren?

  [click] Dann bin ich auf dérive gestoßen, ein Open-Source-Projekt, das genau das macht – GPX-Dateien auf einer Karte darstellen.

  [click] GPX steht (quasi) für GPS Exchange Format – ein XML-basiertes Dateiformat, das GPS-Tracks als Folge von Koordinaten mit Zeitstempeln speichert.
  Praktisch jede Fitness-App und jedes GPS-Gerät kann GPX exportieren.

  [click] Das war cool, aber der Workflow war umständlich: Man musste die GPX-Dateien entweder einzeln aus Strava exportieren, oder durch Massenexport als ZIP.

  Mit der DSGVO wurde es sogar noch schlimmer – statt GPX-Export enthält der Massenexport alle Daten von ganz Strava. Der Prozess hat viel länger gedauert, und die resultierenden Dateien waren riesig und unhandlich – besonders auf dem Handy.

  Das muss besser gehen.
-->