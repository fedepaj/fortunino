<p align="center">
  <img src="media/logo.png" width="360" alt="La faccia di Fortunino sul suo schermo: occhi a cuore, guance rosse">
</p>

<h1 align="center">Fortunino</h1>

<p align="center">
  Un biscotto della fortuna stampato in 3D che scrive da solo le sue massime.<br>
  Avvicini il telefono, e la massima è tua.
</p>

---

## Cosa fa

Dentro il biscotto una Arduino Nano R4 scrive le massime con un piccolo transformer che gira tutto
sulla scheda (242K parametri, pesi int4, nessuna connessione). Un Modulino OLED mostra una faccia
animata che guarda intorno, arrossisce quando il telefono si avvicina, pensa mentre scrive e dorme
quando nessuno la cerca. Un Modulino NFC Tag passa la massima al telefono: il telefono apre una
pagina con la massima su un bigliettino e sei numeri fortunati.

## Come si usa

1. Alimenta Fortunino dalla USB-C.
2. Avvicina il telefono al biscotto: si apre la massima.
3. Allontana il telefono: la massima successiva è già pronta. Quando finiscono, Fortunino ne scrive
   altre sei (circa un minuto, la faccia pensa); quando dorme sogna, e ne scrive intanto.

Dalla seriale (115200) si può farlo pensare, simulare un tocco, elencare le massime, cambiare
faccia: i comandi sono in [firmware/](firmware/README.md).

```
Nano R4 ──Qwiic── Modulino NFC Tag (ST25DV16KC) ──Qwiic── Modulino OLED (SSD1306 128x64, 0x3D)
   └─ USB-C (alimentazione)
```

## Com'è fatto

| Cartella | Cosa contiene |
|---|---|
| [firmware/](firmware/) | libreria Arduino `Fortunino` (transformer, faccia, OLED, NFC), sketch IT / EN, benchmark, strumenti per il computer |
| [brain/](brain/) | il modello: addestramento, quantizzazione, export in header C, il corpus di Qwen3.6 e gli esperimenti |
| [enclosure/](enclosure/) | le versioni del case (OpenSCAD, Python) |
| [pages/](pages/) | la pagina che il telefono apre dal tag, pubblicata su https://fedepaj.github.io/fortunino/ |
| `media/` | il logo (`firmware/tools/face_sim/logo.py`) |

`firmware/`, `brain/` ed `enclosure/` sono submodule (repo private `fedepaj/fortunino-firmware`,
`-brain`, `-enclosure`); `pages/` è una cartella di questa repo e il workflow
`.github/workflows/pages.yml` la pubblica a ogni push che la cambia.

```
brain/corpus  ──►  brain/experiments  ──brain/export.py──►  firmware/.../model_<lang>.h  ──►  Nano R4
                                                                          tag NFC  ──►  pages/
```

## Il cervello

I modelli del firmware sono studenti int4 a 6 layer (242K parametri), addestrati su massime scritte
e giudicate da Qwen3.6 35B, distillati da un teacher da 4,8M parametri e poi migliorati con GRPO sul
modello esattamente come gira sulla scheda, a varietà fissata. Sulle massime che la scheda consegna
il giudice Qwen approva:

| | Precedente, t 0,8 | Attuale, t 0,8 | Attuale, alla temperatura della scheda |
|---|---|---|---|
| italiano | 42% | 69% | 83% (t 0,6) |
| inglese | 54% | 58% | 58% (t 0,8) |

In una prova alla cieca, per ciascuna lingua, il modello attuale è stato preferito agli altri
candidati. L'inglese scrive massime di lunghezza variabile. La ricetta passo per passo è in
[brain/](brain/README.md).

## Clonare e preparare l'ambiente

```sh
git clone --recurse-submodules https://github.com/fedepaj/fortunino.git
python3.12 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

Le submodule sono private: senza accesso si clona solo questa repo (pagina e documentazione).
Strumenti esterni: l'arduino-cli dell'Arduino IDE con il core `arduino:renesas_uno` 1.6.0 (firmware),
Ollama 0.34 con `qwen3.6:35b-a3b` (corpus e giudice), OpenSCAD 2026.09 (case). `assets/` (non
versionata) contiene i modelli 3D di terzi usati dal case.
