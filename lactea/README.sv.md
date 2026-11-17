<p align="center">
  <img src="header.png" alt="Lactea" width="760" />
</p>

<p align="center">
  <a href="../README.md"><img src="flags/flag-gb.svg" alt="" width="18" height="12" /> English</a>
  /
  <a href="README.de.md"><img src="flags/flag-de.svg" alt="" width="18" height="12" /> Deutsch</a>
  /
  <a href="README.it.md"><img src="flags/flag-it.svg" alt="" width="18" height="12" /> Italiano</a>
  /
  <a href="README.es.md"><img src="flags/flag-es.svg" alt="" width="18" height="12" /> Español</a>
  /
  <a href="README.sv.md"><img src="flags/flag-se.svg" alt="" width="18" height="12" /> Svenska</a>
  /
  <a href="README.fr.md"><img src="flags/flag-fr.svg" alt="" width="18" height="12" /> Français</a>
</p>

# projectM — Lactea-utgåvan

Det här är **projectM 4.1.8 med rättningar för kompatibilitet med MilkDrop**. Det är versionen som
används av [Lactea](https://github.com/Karlos-fr/lactea), en musikvisualiserare för Android TV. Alla
appar som bygger på projectM kan använda den.

Med det ursprungliga projectM 4.1.8 laddas **67 presets** i samlingen
[Cream of the Crop](https://github.com/projectM-visualizer/presets-cream-of-the-crop) (9 857 presets)
inte alls. Med den här versionen **laddas alla**. Inget annat ändras: ingen ny funktion, samma API,
samma bygge.

projectM:s ursprungliga README (på engelska) finns [efter den engelska README-filen](../README.md#original-projectm-readme).

## Vilken gren ska man använda

| Gren | Innehåll |
|---|---|
| `lactea/4.1.8` (standard) | projectM 4.1.8 + rättningarna nedan. Används av Lactea. |
| `fix/milkdrop-equation-compat` | Samma rättningar ovanpå projectM:s senaste kod, föreslagna till projectM-teamet. |

Bygg den exakt som projectM (se den ursprungliga README-filen). I ett CMake-projekt kan den ersätta
de officiella källorna som en Git-submodul:

```bash
git submodule add -b lactea/4.1.8 https://github.com/Karlos-fr/projectm.git third_party/projectm
git submodule update --init --recursive
```

## Rättade buggar

### 1. Presetens ekvationer läses som MilkDrop läser dem

MilkDrop klistrar ihop raderna i ett presets ekvationer (`per_frame_1`, `per_frame_2`…) **utan
någon avgränsare**, efter att ha tagit bort kommentarer med `//` och `\\`. Många presets är beroende
av det, med ett namn eller ett tal som delas över två rader:

```
per_frame_21=k1 =  is_
per_frame_22=beat*equal(index%2,0);
```

MilkDrop läser `is_beat`. projectM 4.1.8 satte en radbrytning mellan raderna, läste `is_ beat` och
avvisade presetet ("Could not compile per-frame code"). projectM läser nu ekvationerna som MilkDrop.
Det är också vad projectM:s egen dokumentation av `PresetFileParser::GetCode()` säger. Shader-koden
påverkas inte.

### 2. Ett felaktigt ekvationsblock avvisar inte längre hela presetet

När ett ekvationsblock innehåller ett fel visar MilkDrop en varning och **spelar presetet utan det
blocket**. projectM avvisade hela presetet. Nu gör det som MilkDrop (per bildruta, per pixel,
initiering, egna former och egna vågor).

### 3. Felmeddelanden försvinner inte längre

När ett preset misslyckades fick appen bara `std::exception` i stället för det riktiga felet. Nu ger
projectM:s undantag sitt meddelande. *Den här rättningen behövs bara i 4.1.8: projectM:s senaste
kod har den redan.*

## Resultat

| På Cream of the Crop (9 857 presets) | projectM 4.1.8 | Den här versionen |
|---|---|---|
| Presets som inte laddas | 67 | **0** |

- Samlingens 144 906 ekvationsblock kompilerades på det gamla sättet och på MilkDrops sätt:
  **alla block som fungerade fungerar fortfarande**.
- De 67 presetsen laddades och mättes på en tv (Sony BRAVIA, grafikkrets Mali-G57).
- Rättningarna följer källkoden för MilkDrop 2 (`ReadCode()` och `StripLinefeedCharsAndComments()` i
  `state.cpp`, felhantering i `RecompileExpressions()`).

## Skickat till projectM-teamet

- Felrapport: [projectm#1083](https://github.com/projectM-visualizer/projectm/issues/1083)
- Föreslagen rättning: [projectm#1084](https://github.com/projectM-visualizer/projectm/pull/1084)
- Kvarstående skillnad, i projectm-eval: en ensam `.` är talet 0 i MilkDrop (`.-.4` är `0 - 0.4`),
  men projectm-eval avvisar den. Några presets tappar blocket som innehåller den.
  [projectm-eval#23](https://github.com/projectM-visualizer/projectm-eval/issues/23)

## Hur det gjordes

Buggarna hittades när hela samlingen testades på en tv för Lactea. De rättades med en AI-agent för
programmering, **Anthropic Claude** (Claude Code), under ledning och kontroll av en människa.

## Licens

projectM är licensierat under LGPL 2.1, och det är även de här ändringarna.
