---
layout: chapter
title: Takeaways
---

Was bleibt – für Hobby-Projekte und Enterprise-Software

<!--
  Ein kleines Projekt, das über Jahre gewachsen ist.
  Aber die Probleme, die ich dabei gelöst habe, sind dieselben, die wir auch in Enterprise-Projekten kennen.
-->

---
title: Was ich mitgenommen habe
---

<v-clicks>

- **Externe APIs sind Abhängigkeiten** – sie ändern sich, haben Limits, und man hat keinen Einfluss auf den Zeitplan
- **Client-seitiger Zustand skaliert** – nicht jedes Problem braucht ein Backend mit Datenbank
- **Heuristiken sind legitim** – solange man die Ausnahmefälle kennt und behandelt
- **Compliance trifft alle** – auch Hobby-Projekte müssen Branding-Guidelines und Webhooks implementieren
- **Externe Änderungen erzwingen Reaktionen** – wer früh kommuniziert, hat mehr Zeit zum Reagieren
- **Eigene Software benutzen** – man findet Probleme nur, wenn man das Tool selbst nutzt

</v-clicks>

<!--
  Was nehme ich mit – und was lässt sich auf Enterprise-Projekte übertragen?

  [click] Externe APIs sind Abhängigkeiten mit eigenem Lebenszyklus.
  Rate Limits, Pricing-Änderungen, neue Guidelines – das muss von Anfang an eingeplant werden.

  [click] Client-seitiger Zustand ist keine Notlösung.
  Wenn Daten nur einem Nutzer gehören, gehören sie in dessen Browser – nicht auf einen Server.

  [click] Heuristiken sind kein Zeichen schwacher Architektur.
  Manchmal ist eine pragmatische Näherungslösung mit bekannten Ausnahmen besser als eine komplexe perfekte Lösung.

  [click] Compliance betrifft auch kleine Projekte.
  Wer fremde APIs nutzt, ist Vertragspartner – mit allen Pflichten, die dazugehören.

  [click] Frühzeitige Kommunikation hilft.
  Wer ein Problem erkennt und meldet, hat mehr Zeit und mehr Einfluss auf die Lösung.

  [click] Und das Wichtigste: eigene Software benutzen.
  Ich habe Bugs und Grenzen nur gefunden, weil ich Heatmapper selbst täglich nutze.
  Dogfooding ist kein Luxus – es ist die ehrlichste Form von Qualitätssicherung.
-->

---
layout: end
hideHeader: true
---

::middle::

<QrCode
  value="https://heatmapper.charding.dev"
  color="white"
  background="transparent"
  caption="heatmapper.charding.dev"
/>

::default::

**Charlie Harding**

<EndLink to="charlie.harding@qaware.de" />

::right::

<SmartLink to="github.com/c-harding/heatmapper"><mdi-github /> c-harding/heatmapper</SmartLink>
<SmartLink to="github.com/c-harding/heatmapper-talk"><mdi-github /> c-harding/heatmapper-talk</SmartLink>
