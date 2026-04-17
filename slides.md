---
theme: "@qaware-internal/slidev-theme-qaware"
layout: ec26
title: Heatmapper
shortTitle: Titel
duration: 20m
sectionDuration: 1m
end-time: '14:10'
author: Charlie Harding · QAware GmbH
authorUrl: qaware.de
articleClass: mr-55%
comark: true
htmlAttrs:
  lang: de-DE
---

Digitale Transformation mit fremden APIs —
von GPX-Dateien zu skalierbarer Kartenplattform

::footer::

Charlie Harding
<SmartLink to="charlie.harding@qaware.de"/>

<!--
  Herzlich willkommen zu meinem Talk über ein Hobby-Projekt von mir, namens Heatmapper.
  Ich habe diese App gebaut, um all meine Sportaktivitäten auf einer interaktiven Karte zu zeigen.
  Dabei bin ich auf einige spannende Probleme gestoßen — von API-Limits über Kartendarstellung bis hin zu Datenschutz.
-->

---
title: Wer bin ich?
inner-split: 50
right:
  class: flex
---

**Charlie Harding**  
Senior Software Engineer  
bei QAware GmbH, München

Frontend-begeistert {v-click}

Läufer/Trailrunner und Wanderer {v-click}

::right::

<v-switch at="1">
<template #0-2>
<img src="/media/cha.jpeg"  class="rounded-full w-70 mx-auto" />
</template>
<template #2>
<img src="/media/cha-run.jpeg"  class="rounded-full w-70 mx-auto" />
</template>
</v-switch>

<!--
  Kurz zu mir: Ich bin Charlie, Senior Software Engineer im Projekt CaVORS.

  [click] Ich baue gerne Frontends,

  [click] und bin in meiner Freizeit viel draußen unterwegs – beim Laufen, Trailrunning und Wandern.
  Und genau diese Kombination aus Tech und Outdoor hat zu dem Projekt geführt, über das ich heute spreche.
-->

---
title: Agenda
disabled: true
---

1. **Einstieg & Motivation** — von der Papierkarte zum digitalen Projekt

2. **Transformation & Architektur** — API statt Datei-Export

3. **Karten & Visualisierung** — Mapbox, Layer und Heatmap-Farben

4. **Datenmanagement** — Caching, Delta-Updates und Browser-Limits

5. **Governance & Compliance** — wenn die Plattform die Regeln ändert

6. **Takeaways** — Learnings für Hobby und Enterprise

<!--
  Kurzer Überblick: Wir starten mit der Motivation, schauen uns dann Architektur und Visualisierung an,
  gehen auf Datenmanagement ein und enden mit Governance und den wichtigsten Takeaways.
-->

---
src: ./pages/01-einstieg.md
---

---
src: ./pages/02-transformation.md
---

---
src: ./pages/03-visualisierung.md
---

---
src: ./pages/04-datenmanagement.md
---

---
src: ./pages/05-governance.md
---

---
src: ./pages/06-abschluss.md
---