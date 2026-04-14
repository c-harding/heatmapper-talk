---
layout: chapter
title: Karten & Visualisierung
---

Von klassischen Slippy Maps zu Vektor-basierten Karten

<!--
  Die Architektur steht, die API liefert Daten – jetzt kommt der sichtbarste Teil des Projekts: die Karte.
  In diesem Kapitel schauen wir uns an, wie moderne Web-Karten funktionieren, warum ich Mapbox gewählt habe,
  und wie aus rohen Polylinien eine echte Heatmap wird.
-->

---
title: Zwei Generationen von Web-Karten
inner-split: 50
right:
  class: flex flex-col gap-4
---

### Raster-Karten (Slippy Maps)
- Karte wird als **Bild-Kacheln** geliefert (PNG)
- Server rendert die Karte – Client zeigt sie nur an
- Zoom: neuer Request, neue Kacheln
- Bekannt aus: OpenStreetMap, Google Maps (alt), Leaflet

<v-click at="+3">

### Vektor-Karten
- Server liefert **Rohdaten** (Geometrien, Labels)
- **Client rendert** die Karte in WebGL
- Beliebiger Zoom, Rotation und Neigung
- Dynamisches Styling zur Laufzeit

</v-click>

::right::

<div class="flex h-10em gap-4">
  <img src="/media/slippy-muc.png" class="h-full aspect-square" />
  <div v-if="$clicks > 0" class="rounded-full overflow-hidden border-4 border-black w-10em h-10em">
    <img src="/media/slippy-muc.png" class="aspect-square [image-rendering:pixelated] origin-[45%_75%] transition-transform" :class="{'scale-400': $clicks > 1}" />
  </div>
</div>

<div v-click="3" class="flex h-10em gap-4">
  <img src="/media/vector-muc.png" class="h-full aspect-square" />
  <div v-click="4" class="rounded-full overflow-hidden border-4 border-black w-10em h-10em bg-white">
    <pre class="whitespace-pre-line break-all mr--1">01aaf7e0a0e77617465725f706f6c79676f6e7312251204000001011803221b098a16ee1d5202020810020600020102050000010307010500030f12251204000001021803221b09a016b01e52070d0403050101010109000106010608040a00120f12171204000001031803220d098617ec1f1a06120700030f0f12191204000001041803220f09ac17921d220e06040e130c0b1b0f12171204000001051803220d09d817a2191a061a0d0203150f121712040000</pre>
  </div>
</div>

<!--
  Kurz zur Technologie.
  Die erste Generation von Web-Karten, bekannt als Slippy Maps, funktioniert mit Raster-Kacheln.
  Der Server rendert die Karte als Bild und schickt sie Kachel für Kachel – der Browser zeigt sie nur an.
  Das ist einfach und gut etabliert, aber inflexibel.

  [click] [click] So sieht eine Kachel aus, wenn man reinzoomt – sie wird pixelig, weil es ein Bild ist. Erst danach werden neue Kacheln geladen.

  [click] Moderne Vektor-Karten wie Mapbox GL funktionieren anders: Der Server schickt Rohdaten, und der Browser rendert die Karte selbst in WebGL.
  Das bedeutet: beliebiger Zoom ohne Qualitätsverlust, Rotation – und dynamisches Styling, das sich zur Laufzeit ändern lässt.
  Genau das brauchen wir für eine Heatmap.
-->

---
title: Von Encoded Polylines zur Karte
inner-split: 50
---

**Source** – die Datenschicht

- Enthält Rohdaten (GeoJSON, Raster-Kacheln, …)
- Unsere Activities: GeoJSON aus dekodierten Polylinien

<v-click at="+2">

**Layer** – die Darstellungsschicht

- Referenziert eine Source
- Definiert Typ (`line`, `fill`, `circle`, …) und Stil
- Mehrere Layer pro Source möglich, ggf. mit Filtern

</v-click>

<v-click>

**Style** – der Kartenstil

- Definiert Hintergrund, Straßen, Labels
- Bestehen aus Source- und Layer-Definitionen

</v-click>

::right::

````md magic-move {at: 0}
```js {*}
map.addSource('activities', {
  type: 'geojson',
  data: {
    type: 'FeatureCollection',
    features: activities.map(a => ({
      type: 'Feature',
      geometry: {
        type: 'LineString',
        coordinates:
          decode(a.map.summary_polyline)
            .map(([lat, lng]) => [lng, lat])
      }
    }))
  }
})
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

<!--
  Mapbox GL arbeitet mit drei Kernkonzepten: Source, Layer und Style.
  Eine Source ist der Datenbehälter – unsere Activities kommen als GeoJSON mit dekodierten Polylinien rein.

  [click] Der Layer beschreibt, wie die Source aussehen soll: Typ, Farbe, Transparenz.
  Mehrere Layer können dieselbe Source nutzen, ggf. mit Filtern.

  [click] Und alles baut auf einem Style auf: der Basiskarte, die Mapbox liefert.
  Straßen, Beschriftungen, Hintergrund – alles fertig konfiguriert, abrufbar per URL. Mapbox kann diese Styles hosten, oder man kann eine eigene JSON-Datei mit individuellen Anpassungen bereitstellen.
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
  Jetzt haben wir alle Zutaten parat und können die Anwendung bauen.

  [click] Das Herzstück ist die Karte – die zeigen wir groß und in der Mitte.

  [click] Dazu kommt eine Sidebar mit den Bedienelementen.

  [click] In der Sidebar gibt es ein Logo und einen Button zum Laden der Aktivitäten.

  [click] Darunter erscheint dann die Liste der geladenen Aktivitäten.

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

Wie kriegt man diese Farben hin?

<v-click>

Lösung: Mehrere Layer

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
  Wie entstehen eigentlich diese Farben auf der Heatmap?
  
  [click] Die Antwort: Mehrere halbtransparente Layer übereinander.
  Ein blauer Layer mit 75 % Opacity, ein roter mit 20 % und ein gelber mit 10 %.
  
  [click] Überlagert man die, ergibt sich der typische Heatmap-Farbverlauf – von violett bis orange, je nachdem wie viele Linien sich überlagern.
-->

---
title: Globe-Modus & 3D-Terrain
inner-split: 45
---

Mapbox hat danach neue Features eingeführt – kostenlos nutzbar mit wenigen Zeilen Code.

<v-click>

### 🌍 Globe-Modus

```js
map.setProjection('globe')
```

</v-click>
<v-click>

### ⛰️ 3D-Terrain

```js
map.addSource('terrain', {
  type: 'raster-dem',
  url: 'mapbox://mapbox.terrain-rgb'
})
map.setTerrain({ source: 'terrain' })
```

</v-click>

::right::

<v-switch at="0" class="h-full *:h-full">
  <template #1>
    <img src="/media/globe.png" class="w-full h-full object-cover" />
  </template>
  <template #2>
    <img src="/media/nordkette.png" class="w-full h-full object-cover" />
  </template>
</v-switch>

<!--
  Mapbox hat danach kontinuierlich neue Features eingeführt – kostenlos nutzbar mit wenigen Zeilen Code.
  Das Schöne an einer gut abstrahierten Bibliothek: man profitiert davon fast automatisch.

  [click] Globe-Modus: eine einzige Zeile Code, und die Karte wird zur Weltkugel – sobald man weit genug herauszoomt.
  Das ist besser als die Mercator-Projektion, die viele Online-Karten nutzen: Mercator verzerrt die Größen stark, vor allem in den polaren Regionen.

  [click] 3D-Terrain: ein paar Zeilen mehr, und die Karte bekommt echte Höhenprofile.
  Wanderungen in den Alpen sehen plötzlich ganz anders aus – man sieht sofort, wie bergig eine Route war.
  Besonders beeindruckend in Kombination mit Satellitenbildern als Hintergrundkarte.

  Das ist der Vorteil, wenn man auf eine gepflegte externe Bibliothek setzt: man bekommt neue Features geschenkt.
-->
