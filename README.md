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

| Repo | Contenuto |
|---|---|
| [firmware/](firmware/) | libreria Arduino `Fortunino`, sketch EN / IT / benchmark, simulatore delle facce |
| [brain/](brain/) | il modello (PyTorch), l'addestramento, l'export in header C per il firmware |
| [experiments/distill-gemma/](experiments/distill-gemma/) | corpus distillato da gemma3 (Ollama) e i modelli EN / IT addestrati su di esso |
| [experiments/maestro-qwen/](experiments/maestro-qwen/) | addestramento con Qwen3.6 come supervisore, i suoi round e i giudizi |
| [enclosure/v1](enclosure/v1/) … [v7_figurine](enclosure/v7_figurine/) | le versioni del case (OpenSCAD, Python) |
| [enclosure/ragnar/](enclosure/ragnar/) | prompt per il tool text-to-CAD Ragnar |
| [pages/](pages/) | la pagina che il telefono apre dal tag (GitHub Pages, `fedepaj/fortunino`) |

Fuori dalle repo:

- `assets/`: modelli 3D di terzi (`fortune_cookie_figurine.stl`, usato da v6 e v7;
  `Fortune+Cookie_Serev3d.stl`), non versionati;
- `.venv/`: l'ambiente Python comune (Python 3.12, `requirements.txt`).

## Come si collegano

```
experiments/distill-gemma/runs/<lang>  ──brain/export.py──►  firmware/libraries/Fortunino/src/model_<lang>.h
                                                               └─► firmware/sketches/fortunino_<lang>  ──►  Nano R4
experiments/maestro-qwen/inputs/<lang>  (copia congelata di distill-gemma/runs/<lang> e del suo corpus)
experiments/maestro-qwen/champions/<lang>  ──brain/export.py──►  (stesso header, quando lo si sceglie)
firmware (tag NFC)  ──►  https://fedepaj.github.io/fortunino/#<lang>/<massima>  (pages/)
```

Gli header `model_en.h` e `model_it.h` nel firmware sono l'export di
`experiments/distill-gemma/runs/en` e `runs/it`.

## Ambiente

```sh
python3.12 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

Strumenti esterni: OpenSCAD 2026.09 (case), Ollama 0.34 (corpus e supervisore), l'arduino-cli
dell'Arduino IDE con il core `arduino:renesas_uno` 1.6.0 (firmware).
