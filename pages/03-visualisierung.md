---
layout: chapter
title: Karten & Visualisierung
shortTitle: Visualisierung
section:
  duration: 4m30s
---

Von klassischen Slippy Maps zu Vektor-basierten Karten

<!--
Die Architektur steht, die API liefert Daten – jetzt kommt der sichtbarste Teil: die Karte.
  Wie funktionieren moderne Web-Karten, und wie wird aus rohen Polylines eine Heatmap?
-->

---
title: Zwei Generationen von Web-Karten
inner-split: 50
right:
  class: flex flex-col gap-4
---

### Raster-Karten (Slippy Maps)

- Legacy-Ansatz: Server rendert **vorgefertigte Bild-Kacheln** (PNG)
- Zoom: neuer Request, neue Kacheln
- Bekannt aus: OpenStreetMap, Google Maps (alt), Leaflet

<v-click at="+2">

### Vektor-Karten

- Server liefert **Geodaten** (Geometrien, Labels)
- **Client rendert** die Karte per WebGL
- Beliebiger Zoom, Rotation und Neigung
- Dynamisches Styling zur Laufzeit

</v-click>

::right::

<div class="flex h-10em gap-4">
  <img src="/media/slippy-muc.png" class="h-full aspect-square" />
  <div v-click="1" class="rounded-full overflow-hidden border-4 border-black w-10em h-10em">
    <img src="/media/slippy-muc.png" class="aspect-square [image-rendering:pixelated] origin-[45%_75%] transition-transform" :class="{'scale-400': $clicks >= 1}" />
  </div>
</div>

<div v-click="2" class="flex h-10em gap-4">
  <img src="/media/vector-muc.png" class="h-full aspect-square" />
  <div class="rounded-full overflow-hidden border-4 border-black w-10em h-10em bg-white">
    <pre class="whitespace-pre-line break-all mr--1">01aaf7e0a0e77617465725f706f6c79676f6e7312251204000001011803221b098a16ee1d5202020810020600020102050000010307010500030f12251204000001021803221b09a016b01e52070d0403050101010109000106010608040a00120f12171204000001031803220d098617ec1f1a06120700030f0f12191204000001041803220f09ac17921d220e06040e130c0b1b0f12171204000001051803220d09d817a2191a061a0d0203150f121712040000</pre>
  </div>
</div>

<!--
Bevor wir eine Heatmap bauen können, müssen wir verstehen, wie Web-Karten überhaupt funktionieren.
  Es gibt zwei Generationen – und der Unterschied ist entscheidend:

  Raster-Karten, auch Slippy Maps genannt. Der Server rendert die Karte als Bild, der Browser zeigt sie nur an.

  [click] Reinzoomen zeigt das Problem – die Kachel wird pixelig, bis die Kacheln mit höherer Auflösung nachgeladen werden.

  [click] Vektor-Karten wie Mapbox GL funktionieren anders: Der Server schickt keine Bilder, sondern Geodaten.
  Der Browser rendert die Karte selbst per WebGL.
  Beliebiger Zoom, Rotation und dynamisches Styling – genau das brauchen wir.
-->

---
title: Von Encoded Polylines zur Karte
inner-split: 50
---

**Source**: die Datenschicht

- Enthält Rohdaten (GeoJSON, Raster-Kacheln, …)
- Unsere Activities: GeoJSON aus dekodierten Polylines

<v-click at="+2">

**Layer**: die Darstellungsschicht

- Definiert, wie eine Source dargestellt wird
- Mehrere Layer pro Source möglich

</v-click>
<v-click>

**Style**: der Kartenstil

- Definiert Hintergrund, Straßen, Labels
- Besteht aus Source- und Layer-Definitionen

</v-click>

::right::

````md magic-move {at: 1}
```js {*}
map.addSource('activities', {
  type: 'geojson',
  data: {
    type: 'FeatureCollection',
    features: activities.map((a) => ({
      type: 'Feature',
      geometry: {
        type: 'LineString',
        coordinates: decode(a.map.summary_polyline).map(([lat, lng]) => [
          lng,
          lat,
        ]),
      },
    })),
  },
});
```

```js
map.addSource('activities', {
  type: 'geojson',
  data: { ... }
})
```

```js {6-14}
map.addSource('activities', {
  type: 'geojson',
  data: { ... }
})

map.addLayer({
  id: 'heatmap',
  type: 'line',
  source: 'activities',
  paint: {
    'line-color': '#00F',
    'line-opacity': 0.75
  }
})
```

```js {1-3}
const map = new mapboxgl.Map({
  style: 'mapbox://styles/mapbox/streets-v12',
})

map.addSource('activities', {
  type: 'geojson',
  data: { ... }
})

map.addLayer({
  id: 'heatmap',
  type: 'line',
  source: 'activities',
  paint: {
    'line-color': '#00F',
    'line-opacity': 0.75
  }
})
```
````

<script setup>
import { useSlideContext, useNav, useIsSlideActive } from '@slidev/client'
import { watch } from 'vue'

const nav = useNav()
const active = useIsSlideActive()

// Auto-advance/rewind on click 1, with 1.2s delay to allow the code transition to complete.
// This fixes a mismatch in the code animation.

watch([active, nav.clicks], ([isActive, clicks], [wasActive, oldClicks]) => {
  if (isActive && clicks === 1 && wasActive) {
    const timer = setTimeout(() => {
      if (active.value && nav.clicks.value === 1) {
        clicks > oldClicks ? nav.next() : nav.prev()
      }
    }, 1200)
    const unwatch = watch([active, nav.clicks], () => {
      clearTimeout(timer)
    }, { once: true })
  }
})
</script>

<!--
Mapbox GL hat drei Kernkonzepte: Source, Layer und Style.
  Unsere Activities kommen als GeoJSON-Source rein.

  [click:2] Der Layer definiert, wie die Daten aussehen – beispielsweise Farbe, Transparenz.

  [click] Und alles baut auf einem Style auf – einer Basiskarte, die man fertig laden oder selbst definieren kann.
-->

---
title: Design
clicks: 6
transition: fade
---

<style>
.mockup {
  --uno: bg-weak flex flex-col flex-1 min-h-0 relative z-1;

  &, * {
    transition: all 0.3s ease-in-out;
  }

  fieldset {
    --uno: flex border-4 gap-2 p-2 relative;

    > legend {
      --uno: mx-auto px-1 font-600 text-lg;
    }
  }

  .foldable {
    overflow-y: hidden;
    interpolate-size: allow-keywords;

    &.folded {
      height: 0;
    }
  }

  &.overlap {
    --uno: mt--25 mb--8 pb-3.5;

    > fieldset {
      --uno: mx-7%;
    }

    fieldset { 
      --uno: gap-0 p-0;
      border-radius: 0 !important;
    }
  }

  fieldset {
    border-radius: 40px;
    
    fieldset {
      border-radius: 28px;

       fieldset {
        border-radius: 16px;
        
        fieldset {
          border-radius: 4px;
        }
      }
    }
  }
}
</style>

<div class="mockup" :class="{'overlap': $clicks > 5}">

<fieldset class="h-full border-gray-400 overflow-hidden"><legend>App</legend>
  <fieldset class="flex-col border-blue-500 p-3" :class="[$clicks > 5 ? 'basis-37' : 'basis-60', $clicks < 2 && 'opacity-0 flex-0 ml--62 min-w-0 overflow-hidden']"><legend>Sidebar</legend>
    <v-click at="3">
      <fieldset class="border-blue-500 text-sm font-bold"><legend>Logo</legend></fieldset>
      <div class="flex flex-col-reverse shrink-0 foldable" :class="{folded: $clicks > 4}">
        <fieldset class="border-green-500 text-sm text-center mx-auto"><legend>Button</legend></fieldset>
      </div>
    </v-click>
    <fieldset v-click="4" class="flex-col gap-1 mt-1 overflow-hidden border-red-500"><legend>Aktivitäten</legend>
      <fieldset v-for="i of 20" :key="i" class="border-orange-400 px-2 py-1 text-xs"><legend>Aktivität</legend></fieldset>
    </fieldset>
  </fieldset>
  <fieldset v-click="1" class="flex-1 border-green-500 items-center justify-center text-8xl"><legend>Karte</legend>
    🗺️
  </fieldset>
</fieldset>

</div>

<!--
Jetzt bauen wir die Anwendung zusammen.

  [click] Das Herzstück ist die Karte.

  [click] Dazu eine Sidebar für die Bedienelemente.

  [click] Ein Logo und ein Sync-Button.

  [click] Und darunter die Liste der Aktivitäten.

  [click] Wir können nach unten scrollen und mehr Aktivitäten sehen.

  [click] Es kann sogar viele Aktivitäten geben.

  Jetzt müssen wir nur noch die Karte mit Leben füllen.
-->

---
title: Screenshot der originalen Version
hideHeader: true
image: /media/heatmapper-old.png
pageClass: bg-transparent bg-contain
---

<!--
Tada! So hat die erste Version von Heatmapper ausgesehen.
-->

---
title: Farben
split: 40
title-side: right
left:
  image: /media/heatmapper-zoom.png
---

::right::

Je mehr Routen sich überlagern, desto wärmer die Farbe — aber wie?

<v-click>

Mehrere halbtransparente Layer übereinander:

- Layer `1`: `#00F` <div class="w-1em h-1em bg-[#00F] inline-block" /> × `75%`
  <Line :count="4" class="h-2em inline" color="#00F" :opacity="0.75" />
- Layer `2`: `#F00` <div class="w-1em h-1em bg-[#F00] inline-block" /> × `20%`
  <Line :count="4" class="h-2em inline" color="#F00" :opacity="0.2"/>
- Layer `3`: `#FF0` <div class="w-1em h-1em bg-[#FF0] inline-block" /> × `10%`
  <Line :count="4" class="h-2em inline" color="#FF0" :opacity="0.1"/>

</v-click>

<v-click>

Σ: <Line :count="10" class="h-2em inline" :layers="[
  { color: '#00F', opacity: 0.75 },
  { color: '#F00', opacity: 0.2 },
  { color: '#FF0', opacity: 0.1 }
]" />

</v-click>

<!--
Je mehr Routen sich überlagern, desto wärmer wird die Farbe. Daher kommt das Wort „Heatmap“. Aber wie funktioniert das technisch?

  [click] Die Lösung: Drei halbtransparente Layer übereinander.
  Blau mit 75 % Opacity, Rot mit 20 % und Gelb mit 10 %.

  [click] Überlagern sich viele Linien, addieren sich die Farben — von kaltem Violett bis hin zu warmem Orange.
-->

---
title: Globusansicht & 3D-Terrain
inner-split: 45
---

Mapbox hat danach neue Features eingeführt – kostenlos nutzbar mit wenigen Zeilen Code.

<v-click>

### 🌍 Globusansicht

```js
map.setProjection('globe');
```

</v-click>
<v-click>

### ⛰️ 3D-Terrain

```js
map.addSource('terrain', {
  type: 'raster-dem',
  url: 'mapbox://mapbox.terrain-rgb',
});
map.setTerrain({ source: 'terrain' });
```

</v-click>

::right::

<v-switch class="h-full *:h-full" at="1">
  <template #1>
    <img src="/media/globe.png" class="w-full h-full object-cover" />
  </template>
  <template #2>
    <img src="/media/nordkette.png" class="w-full h-full object-cover" />
  </template>
</v-switch>

<!--
Mapbox hat danach neue Features eingeführt – kostenlos nutzbar mit wenigen Zeilen Code.

  [click] Globusansicht: eine einzige Zeile, und die Karte wird zur Weltkugel.

  [click] 3D-Terrain: ein paar Zeilen mehr, und die Karte bekommt echte Höhenprofile – besonders beeindruckend mit Satellitenbildern.
  Das ist der Vorteil einer gepflegten externen Bibliothek: neue Features geschenkt.
-->
