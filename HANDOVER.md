# Bodega-kompasset — handover

## Hva dette er

En enkeltside som fungerer som et kompass med bare én funksjon: pila peker alltid mot
Bodega (Kong Oscars gate 23, Bergen). Skal nås via QR-kode på trykt materiell —
klistremerker, plakater, kanskje undersiden av glassbrikker.

Målgruppen er folk som står ute i Bergen sentrum på kvelden med telefon i hånda.
Siden skal gjøre én ting, umiddelbart, uten forklaring.

## Status

`index.html` er ferdig som første versjon. Én fil, ingen avhengigheter, ingen
byggesteg. All CSS og JS ligger inline. Den er **ikke testet på ekte enheter ennå** —
det er neste steg.

## Teknisk oppsummering

| Del | Løsning |
|---|---|
| Posisjon | `navigator.geolocation.watchPosition`, `enableHighAccuracy: true` |
| Retning | `webkitCompassHeading` (iOS) / `deviceorientationabsolute` + `alpha` (Android) |
| Peiling | Haversine for avstand, standard great-circle bearing for retning |
| Animasjon | `requestAnimationFrame` med korteste-vei-interpolasjon, faktor 0.16 |
| Fallback | Uten magnetometer brukes `coords.heading` fra GPS (krever bevegelse) |

Konstanter øverst i scriptet:

```js
const TARGET      = { lat: 60.3937156, lon: 5.3298621 };
const ARRIVED_M   = 30;   // under dette regnes du som framme
const ALIGNED_DEG = 10;   // under dette sier den "rett fram"
```

## Oppgaver

### 1. Deploy til Netlify for testing
Drag-and-drop av mappa på Netlify Drop holder. Trenger HTTPS — uten det gir hverken
geolocation eller DeviceOrientation noe fra seg.

### 2. Test på ekte enheter
Dette er hovedjobben. Sjekkliste:

- [ ] **iOS Safari** — tillatelsesdialogen for bevegelse og retning må komme opp ved
      trykk på startknappen. Kommer den ikke, er `requestPermission()` kalt utenfor
      brukergesten.
- [ ] **Android Chrome** — `deviceorientationabsolute` fyrer ikke på alle enheter.
      Verifiser at `e.absolute === true`; hvis ikke, faller den til GPS-kurs.
- [ ] Snu deg 180° og se at pila følger etter, ikke bare vibrerer.
- [ ] Roter telefonen til landskap — pila skal ikke hoppe 90°. Håndteres av
      `screen.orientation.angle`, men fortegnet er verdt å verifisere på begge
      plattformer.
- [ ] Gå mot Kong Oscars gate og bekreft at "framme"-tilstanden (hele skjermen gul)
      slår inn på riktig sted, ikke et kvartal for tidlig.
- [ ] Test med posisjon avslått — feilmeldingen skal være lesbar, ikke en hvit skjerm.

### 3. Sett opp permanent URL
`part.no/kompass` eller `bodega.part.no`. Viktig: QR-koden skal peke på den
permanente adressen fra dag én, med redirect til testadressen i mellomtiden.
En trykt QR-kode mot `*.netlify.app` er død den dagen vi flytter siden.

### 4. Generer QR-kode
Mot den permanente URL-en. Høy feilkorreksjon (level H) siden koden kan havne på
klistremerker som blir slitt.

## Designregler

Braun / Dieter Rams-logikk: svart skive, gul viser, alt annet skal holde kjeft.
Fargene ligger som CSS-variabler øverst.

- Typografi: **Neue Haas Grotesk**. Fontstacken ber om den først og faller til
  Helvetica. Hvis part.no allerede laster webfonten, plukkes den opp automatisk —
  ellers må `@font-face` legges inn.
- **Ikke** Barlow Condensed.
- Ingen nye farger uten grunn. Gult er viserens farge alene.
- Motion respekterer `prefers-reduced-motion`.

## Kjente svakheter

- Magnetometeret i telefoner er upresist, særlig nær stål og betong. Litt skjelving
  i pila er normalt og skal ikke "fikses" med hardere demping — da blir den treg.
- Interpolasjonen i `animate()` bruker en fast faktor uavhengig av framerate. Fungerer
  fint på 60 Hz, blir raskere på 120 Hz-skjermer. Kan gjøres delta-tid-basert hvis det
  merkes.
- Ingen offline-håndtering. Kan vurderes som PWA senere, men er ikke nødvendig for
  første versjon.

## Ideer vi ikke har tatt stilling til

- Teller for meter gått siden skanning
- Noe som skjer når du står stille lenge
- Åpningstider — bevisst utelatt, blir fort utdatert i en statisk fil
