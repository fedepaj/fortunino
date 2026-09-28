# pages

La pagina che il telefono apre toccando Fortunino, pubblicata con GitHub Pages su
https://fedepaj.github.io/fortunino/ dal workflow `.github/workflows/pages.yml` della repo ombrello,
a ogni push su `main` che cambia questa cartella.

Il link del tag è `https://fedepaj.github.io/fortunino/#<lang>/<massima>`: la massima con `+` al
posto degli spazi e `%XX` per gli altri caratteri. La pagina mostra la massima su un bigliettino, i
sei numeri fortunati (calcolati dal testo come nel firmware), un tasto per condividerla; senza
massima mostra un invito ad avvicinare il telefono. Testi in italiano o inglese secondo `<lang>`.

`index.html` è tutta la pagina; `.nojekyll` fa servire i file così come sono.
