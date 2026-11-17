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

# projectM — édition Lactea

Voici **projectM 4.1.8 avec des corrections de compatibilité MilkDrop**. C'est la version utilisée
par [Lactea](https://github.com/Karlos-fr/lactea), un visualiseur musical pour Android TV, et toute
application construite sur projectM peut l'utiliser.

Avec le projectM 4.1.8 d'origine, **67 presets** de la collection
[Cream of the Crop](https://github.com/projectM-visualizer/presets-cream-of-the-crop) (9 857 presets)
ne se chargent pas du tout. Avec cette version, **ils se chargent tous**. Rien d'autre ne change :
aucune nouvelle fonction, même API, même compilation.

Le README d'origine de projectM (en anglais) se trouve [à la suite du README anglais](../README.md#original-projectm-readme).

## Quelle branche utiliser

| Branche | Contenu |
|---|---|
| `lactea/4.1.8` (par défaut) | projectM 4.1.8 + les corrections ci-dessous. Utilisée par Lactea. |
| `fix/milkdrop-equation-compat` | Les mêmes corrections sur la dernière version de projectM, proposées à l'équipe de projectM. |

Elle se compile exactement comme projectM (voir son README d'origine). Dans un projet CMake, elle peut
remplacer les sources officielles en sous-module Git :

```bash
git submodule add -b lactea/4.1.8 https://github.com/Karlos-fr/projectm.git third_party/projectm
git submodule update --init --recursive
```

## Bugs corrigés

### 1. Les équations des presets sont lues comme MilkDrop les lit

MilkDrop recolle les lignes des équations d'un preset (`per_frame_1`, `per_frame_2`…) **sans aucun
séparateur**, après avoir retiré les commentaires `//` et `\\`. Beaucoup de presets en dépendent, avec
un nom ou un nombre coupé sur deux lignes :

```
per_frame_21=k1 =  is_
per_frame_22=beat*equal(index%2,0);
```

MilkDrop lit `is_beat`. projectM 4.1.8 mettait un retour à la ligne entre les deux, lisait `is_ beat`
et refusait le preset (« Could not compile per-frame code »). projectM lit maintenant les équations
comme MilkDrop, ce que dit d'ailleurs sa propre documentation de `PresetFileParser::GetCode()`. Le
code des shaders n'est pas concerné.

### 2. Un bloc d'équations fautif ne fait plus refuser tout le preset

Quand un bloc d'équations contient une erreur, MilkDrop affiche un avertissement et **joue le preset
sans ce bloc**. projectM refusait tout le preset. Il fait maintenant comme MilkDrop (équations par
image, par pixel, d'initialisation, des formes et des ondes personnalisées).

### 3. Les messages d'erreur ne sont plus perdus

Quand un preset échouait, l'application recevait seulement `std::exception` au lieu de la vraie
erreur. Les exceptions de projectM donnent maintenant leur message. *Cette correction ne concerne que
la 4.1.8 : la dernière version de projectM l'a déjà.*

## Résultats

| Sur Cream of the Crop (9 857 presets) | projectM 4.1.8 | Cette version |
|---|---|---|
| Presets qui ne se chargent pas | 67 | **0** |

- Les 144 906 blocs d'équations de la collection ont été compilés à l'ancienne et à la manière de
  MilkDrop : **tous les blocs qui fonctionnaient fonctionnent toujours**.
- Les 67 presets ont été chargés et mesurés sur une TV (Sony BRAVIA, puce graphique Mali-G57).
- Ces corrections suivent le code source de MilkDrop 2 (`ReadCode()` et
  `StripLinefeedCharsAndComments()` dans `state.cpp`, gestion des erreurs dans
  `RecompileExpressions()`).

## Transmis à l'équipe de projectM

- Signalement du bug : [projectm#1083](https://github.com/projectM-visualizer/projectm/issues/1083)
- Correction proposée : [projectm#1084](https://github.com/projectM-visualizer/projectm/pull/1084)
- Différence restante, dans projectm-eval : un `.` seul vaut 0 dans MilkDrop (`.-.4` vaut `0 - 0.4`)
  mais projectm-eval le refuse. Quelques presets perdent le bloc qui le contient.
  [projectm-eval#23](https://github.com/projectM-visualizer/projectm-eval/issues/23)

## Comment ça a été fait

Les bugs ont été trouvés en testant toute la collection sur une TV pour Lactea, et corrigés avec un
agent de programmation IA, **Anthropic Claude** (Claude Code), sous la direction et le contrôle d'un
humain.

## Licence

projectM est sous licence LGPL 2.1, et ces modifications aussi.
