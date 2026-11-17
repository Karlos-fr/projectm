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

# projectM — edizione Lactea

Questo è **projectM 4.1.8 con correzioni di compatibilità con MilkDrop**. È la versione usata da
[Lactea](https://github.com/Karlos-fr/lactea), un visualizzatore musicale per Android TV, e può
essere usata da qualsiasi applicazione basata su projectM.

Con il projectM 4.1.8 originale, **67 preset** della raccolta
[Cream of the Crop](https://github.com/projectM-visualizer/presets-cream-of-the-crop) (9.857 preset)
non si caricano affatto. Con questa versione **si caricano tutti**. Non cambia nient'altro:
nessuna nuova funzione, stessa API, stessa compilazione.

Il README originale di projectM (in inglese) si trova [dopo il README inglese](../README.md#original-projectm-readme).

## Quale branch usare

| Branch | Contenuto |
|---|---|
| `lactea/4.1.8` (predefinito) | projectM 4.1.8 + le correzioni qui sotto. Usato da Lactea. |
| `fix/milkdrop-equation-compat` | Le stesse correzioni sul codice più recente di projectM, proposte al team di projectM. |

Si compila esattamente come projectM (vedi il suo README originale). In un progetto CMake può
sostituire i sorgenti ufficiali come submodule Git:

```bash
git submodule add -b lactea/4.1.8 https://github.com/Karlos-fr/projectm.git third_party/projectm
git submodule update --init --recursive
```

## Bug corretti

### 1. Le equazioni dei preset vengono lette come le legge MilkDrop

MilkDrop unisce le righe delle equazioni di un preset (`per_frame_1`, `per_frame_2`…) **senza alcun
separatore**, dopo aver tolto i commenti `//` e `\\`. Molti preset contano su questo, con un nome o
un numero spezzato su due righe:

```
per_frame_21=k1 =  is_
per_frame_22=beat*equal(index%2,0);
```

MilkDrop legge `is_beat`. projectM 4.1.8 metteva un a capo tra le righe, leggeva `is_ beat` e
rifiutava il preset ("Could not compile per-frame code"). Ora projectM legge le equazioni come
MilkDrop, ed è anche ciò che dice la sua stessa documentazione di `PresetFileParser::GetCode()`. Il
codice degli shader non è interessato.

### 2. Un blocco di equazioni con errori non fa più rifiutare tutto il preset

Quando un blocco di equazioni contiene un errore, MilkDrop mostra un avviso e **riproduce il preset
senza quel blocco**. projectM rifiutava tutto il preset. Ora fa come MilkDrop (equazioni per
fotogramma, per pixel, di inizializzazione, delle forme e delle onde personalizzate).

### 3. I messaggi di errore non vanno più persi

Quando un preset falliva, l'applicazione riceveva solo `std::exception` invece dell'errore vero.
Ora le eccezioni di projectM riportano il loro messaggio. *Questa correzione serve solo sulla 4.1.8:
il codice più recente di projectM ce l'ha già.*

## Risultati

| Su Cream of the Crop (9.857 preset) | projectM 4.1.8 | Questa versione |
|---|---|---|
| Preset che non si caricano | 67 | **0** |

- I 144.906 blocchi di equazioni della raccolta sono stati compilati nel modo vecchio e nel modo di
  MilkDrop: **tutti i blocchi che funzionavano funzionano ancora**.
- I 67 preset sono stati caricati e misurati su una TV (Sony BRAVIA, chip grafico Mali-G57).
- Queste correzioni seguono il codice sorgente di MilkDrop 2 (`ReadCode()` e
  `StripLinefeedCharsAndComments()` in `state.cpp`, gestione degli errori in
  `RecompileExpressions()`).

## Inviato al team di projectM

- Segnalazione del bug: [projectm#1083](https://github.com/projectM-visualizer/projectm/issues/1083)
- Correzione proposta: [projectm#1084](https://github.com/projectM-visualizer/projectm/pull/1084)
- Differenza rimasta, in projectm-eval: un `.` da solo vale 0 in MilkDrop (`.-.4` vale `0 - 0.4`),
  ma projectm-eval lo rifiuta. Alcuni preset perdono il blocco che lo contiene.
  [projectm-eval#23](https://github.com/projectM-visualizer/projectm-eval/issues/23)

## Come è stato fatto

I bug sono stati trovati testando tutta la raccolta su una TV per Lactea, e corretti con un agente di
programmazione IA, **Anthropic Claude** (Claude Code), guidato e controllato da una persona.

## Licenza

projectM è distribuito con licenza LGPL 2.1, e lo sono anche queste modifiche.
