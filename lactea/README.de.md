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

# projectM — Lactea-Edition

Dies ist **projectM 4.1.8 mit Korrekturen für die MilkDrop-Kompatibilität**. Es ist die Version, die
[Lactea](https://github.com/Karlos-fr/lactea) verwendet, eine Musikvisualisierung für Android TV.
Jede Anwendung, die auf projectM aufbaut, kann sie nutzen.

Mit dem originalen projectM 4.1.8 laden **67 Presets** der Sammlung
[Cream of the Crop](https://github.com/projectM-visualizer/presets-cream-of-the-crop) (9.857 Presets)
überhaupt nicht. Mit dieser Version **laden alle**. Sonst ändert sich nichts: keine neue Funktion,
gleiche API, gleicher Build.

Das originale README von projectM (auf Englisch) folgt [im Anschluss an das englische README](../README.md#original-projectm-readme).

## Welcher Branch

| Branch | Inhalt |
|---|---|
| `lactea/4.1.8` (Standard) | projectM 4.1.8 + die Korrekturen unten. Wird von Lactea verwendet. |
| `fix/milkdrop-equation-compat` | Dieselben Korrekturen auf dem neuesten Code von projectM, dem projectM-Team vorgeschlagen. |

Sie wird genau wie projectM gebaut (siehe sein originales README). In einem CMake-Projekt kann sie
die offiziellen Quellen als Git-Submodul ersetzen:

```bash
git submodule add -b lactea/4.1.8 https://github.com/Karlos-fr/projectm.git third_party/projectm
git submodule update --init --recursive
```

## Behobene Fehler

### 1. Preset-Gleichungen werden so gelesen, wie MilkDrop sie liest

MilkDrop fügt die Zeilen der Gleichungen eines Presets (`per_frame_1`, `per_frame_2`…) **ohne jedes
Trennzeichen** zusammen, nachdem es die Kommentare `//` und `\\` entfernt hat. Viele Presets verlassen
sich darauf, mit einem Namen oder einer Zahl, die über zwei Zeilen geteilt ist:

```
per_frame_21=k1 =  is_
per_frame_22=beat*equal(index%2,0);
```

MilkDrop liest `is_beat`. projectM 4.1.8 setzte einen Zeilenumbruch zwischen die Zeilen, las
`is_ beat` und lehnte das Preset ab („Could not compile per-frame code“). projectM liest Gleichungen
jetzt wie MilkDrop. Das sagt auch seine eigene Dokumentation von `PresetFileParser::GetCode()`.
Shader-Code ist nicht betroffen.

### 2. Ein fehlerhafter Gleichungsblock lässt nicht mehr das ganze Preset scheitern

Wenn ein Gleichungsblock einen Fehler enthält, zeigt MilkDrop eine Warnung und **spielt das Preset
ohne diesen Block**. projectM lehnte das ganze Preset ab. Jetzt verhält es sich wie MilkDrop
(Gleichungen pro Bild, pro Pixel, zur Initialisierung, für eigene Formen und eigene Wellen).

### 3. Fehlermeldungen gehen nicht mehr verloren

Wenn ein Preset scheiterte, bekam die Anwendung nur `std::exception` statt des echten Fehlers. Die
Exceptions von projectM liefern jetzt ihre Meldung. *Diese Korrektur ist nur für 4.1.8 nötig: Der
neueste Code von projectM enthält sie bereits.*

## Ergebnisse

| Mit Cream of the Crop (9.857 Presets) | projectM 4.1.8 | Diese Version |
|---|---|---|
| Presets, die nicht laden | 67 | **0** |

- Die 144.906 Gleichungsblöcke der Sammlung wurden auf die alte Art und auf die MilkDrop-Art
  kompiliert: **Jeder Block, der funktionierte, funktioniert weiterhin**.
- Die 67 Presets wurden auf einem Fernseher geladen und gemessen (Sony BRAVIA, Grafikchip Mali-G57).
- Diese Korrekturen folgen dem Quellcode von MilkDrop 2 (`ReadCode()` und
  `StripLinefeedCharsAndComments()` in `state.cpp`, Fehlerbehandlung in `RecompileExpressions()`).

## An das projectM-Team weitergegeben

- Fehlerbericht: [projectm#1083](https://github.com/projectM-visualizer/projectm/issues/1083)
- Vorgeschlagene Korrektur: [projectm#1084](https://github.com/projectM-visualizer/projectm/pull/1084)
- Verbleibender Unterschied, in projectm-eval: Ein einzelner `.` ist in MilkDrop die Zahl 0 (`.-.4`
  ist `0 - 0.4`), wird aber von projectm-eval abgelehnt. Einige Presets verlieren den Block, der ihn
  enthält.
  [projectm-eval#23](https://github.com/projectM-visualizer/projectm-eval/issues/23)

## Wie es gemacht wurde

Die Fehler wurden gefunden, als für Lactea die ganze Sammlung auf einem Fernseher getestet wurde. Sie
wurden mit einem KI-Programmieragenten behoben, **Anthropic Claude** (Claude Code), unter der Leitung
und Kontrolle eines Menschen.

## Lizenz

projectM steht unter der LGPL 2.1, und diese Änderungen ebenfalls.
