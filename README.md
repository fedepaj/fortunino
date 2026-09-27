# Fortunino

Un biscotto della fortuna stampato in 3D. Una Arduino Nano R4 scrive le massime con un transformer
di ≈153K parametri (int8), un Modulino OLED mostra una faccia animata, un Modulino NFC Tag passa la
massima al telefono che lo tocca.

```
Nano R4 ──Qwiic── Modulino NFC Tag (ST25DV16KC) ──Qwiic── Modulino OLED (SSD1306 128x64, 0x3D)
   └─ USB-C (alimentazione)
```

Questa cartella è la repo ombrello: ogni sottocartella qui sotto è una repo git a sé, agganciata
come submodule (`git submodule status` mostra la versione di ognuna).

| Repo | GitHub | Contenuto |
|---|---|---|
| [firmware/](firmware/) | `fedepaj/fortunino-firmware` | libreria Arduino `Fortunino`, sketch EN / IT / benchmark, simulatore delle facce |
| [brain/](brain/) | `fedepaj/fortunino-brain` | il modello (PyTorch), addestramento, export in header C; `corpus/` (massime e negativi di Qwen3.6); `experiments/` (un esperimento per cartella) |
| [enclosure/](enclosure/) | `fedepaj/fortunino-enclosure` | le versioni del case, `v1/` … `v7_figurine/`, e `ragnar/` (prompt text-to-CAD) |
| [pages/](pages/) | `fedepaj/fortunino` (pubblica) | la pagina che il telefono apre dal tag (GitHub Pages) |

Clonare tutto: `git clone --recurse-submodules <repo ombrello>`, oppure `git submodule update --init`
dentro una copia già clonata.

Fuori dalle repo:

- `assets/`: modelli 3D di terzi (`fortune_cookie_figurine.stl`, usato da v6 e v7;
  `Fortune+Cookie_Serev3d.stl`), non versionati;
- `.venv/`: l'ambiente Python comune (Python 3.12, `requirements.txt`).

## Come si collegano

```
brain/corpus/raw/<lang>  ──►  brain/corpus/data/<lang>/vN  (versioni congelate)  ──►  brain/experiments/<nome>
brain/experiments/distill-gemma/runs/<lang>  ──brain/export.py──►  firmware/libraries/Fortunino/src/model_<lang>.h
                                                                     └─► firmware/sketches/fortunino_<lang>  ──►  Nano R4
brain/experiments/maestro-qwen/champions/<lang>  ──brain/export.py──►  (stesso header, quando lo si sceglie)
firmware (tag NFC)  ──►  https://fedepaj.github.io/fortunino/#<lang>/<massima>  (pages/)
```

Gli header `model_en.h` e `model_it.h` nel firmware sono l'export di
`brain/experiments/distill-gemma/runs/en` e `runs/it`.

## Ambiente

```sh
python3.12 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

Strumenti esterni: OpenSCAD 2026.09 (case), Ollama 0.34 (corpus e supervisore), l'arduino-cli
dell'Arduino IDE con il core `arduino:renesas_uno` 1.6.0 (firmware).
