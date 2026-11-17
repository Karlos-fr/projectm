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

# projectM — edición Lactea

Esto es **projectM 4.1.8 con correcciones de compatibilidad con MilkDrop**. Es la versión que usa
[Lactea](https://github.com/Karlos-fr/lactea), un visualizador musical para Android TV, y cualquier
aplicación basada en projectM puede usarla.

Con el projectM 4.1.8 original, **67 presets** de la colección
[Cream of the Crop](https://github.com/projectM-visualizer/presets-cream-of-the-crop) (9857 presets)
no se cargan en absoluto. Con esta versión, **se cargan todos**. Nada más cambia: ninguna función
nueva, la misma API, la misma compilación.

El README original de projectM (en inglés) está [a continuación del README en inglés](../README.md#original-projectm-readme).

## Qué rama usar

| Rama | Contenido |
|---|---|
| `lactea/4.1.8` (por defecto) | projectM 4.1.8 + las correcciones de abajo. La usa Lactea. |
| `fix/milkdrop-equation-compat` | Las mismas correcciones sobre el código más reciente de projectM, propuestas al equipo de projectM. |

Se compila exactamente igual que projectM (consulta su README original). En un proyecto CMake, puede
sustituir a las fuentes oficiales como submódulo de Git:

```bash
git submodule add -b lactea/4.1.8 https://github.com/Karlos-fr/projectm.git third_party/projectm
git submodule update --init --recursive
```

## Errores corregidos

### 1. Las ecuaciones de los presets se leen como las lee MilkDrop

MilkDrop une las líneas de las ecuaciones de un preset (`per_frame_1`, `per_frame_2`…) **sin ningún
separador**, después de quitar los comentarios `//` y `\\`. Muchos presets dependen de ello, con un
nombre o un número partido en dos líneas:

```
per_frame_21=k1 =  is_
per_frame_22=beat*equal(index%2,0);
```

MilkDrop lee `is_beat`. projectM 4.1.8 ponía un salto de línea entre las dos líneas, leía `is_ beat`
y rechazaba el preset («Could not compile per-frame code»). Ahora projectM lee las ecuaciones como
MilkDrop, que es también lo que dice su propia documentación de `PresetFileParser::GetCode()`. El
código de los shaders no se ve afectado.

### 2. Un bloque de ecuaciones erróneo ya no hace rechazar todo el preset

Cuando un bloque de ecuaciones contiene un error, MilkDrop muestra un aviso y **reproduce el preset
sin ese bloque**. projectM rechazaba todo el preset. Ahora hace lo mismo que MilkDrop (ecuaciones
por fotograma, por píxel, de inicialización, de formas y de ondas personalizadas).

### 3. Los mensajes de error ya no se pierden

Cuando un preset fallaba, la aplicación solo recibía `std::exception` en lugar del error real. Ahora
las excepciones de projectM dan su mensaje. *Esta corrección solo hace falta en la 4.1.8: el código
más reciente de projectM ya la tiene.*

## Resultados

| En Cream of the Crop (9857 presets) | projectM 4.1.8 | Esta versión |
|---|---|---|
| Presets que no se cargan | 67 | **0** |

- Los 144 906 bloques de ecuaciones de la colección se compilaron a la manera antigua y a la manera
  de MilkDrop: **todos los bloques que funcionaban siguen funcionando**.
- Los 67 presets se cargaron y se midieron en una TV (Sony BRAVIA, chip gráfico Mali-G57).
- Estas correcciones siguen el código fuente de MilkDrop 2 (`ReadCode()` y
  `StripLinefeedCharsAndComments()` en `state.cpp`, gestión de errores en `RecompileExpressions()`).

## Enviado al equipo de projectM

- Informe del error: [projectm#1083](https://github.com/projectM-visualizer/projectm/issues/1083)
- Corrección propuesta: [projectm#1084](https://github.com/projectM-visualizer/projectm/pull/1084)
- Diferencia pendiente, en projectm-eval: un `.` solo vale 0 en MilkDrop (`.-.4` vale `0 - 0.4`),
  pero projectm-eval lo rechaza. Algunos presets pierden el bloque que lo contiene.
  [projectm-eval#23](https://github.com/projectM-visualizer/projectm-eval/issues/23)

## Cómo se hizo

Los errores se encontraron al probar toda la colección en una TV para Lactea, y se corrigieron con
un agente de programación con IA, **Anthropic Claude** (Claude Code), bajo la dirección y la
supervisión de una persona.

## Licencia

projectM tiene licencia LGPL 2.1, y estos cambios también.
