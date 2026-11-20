# Changelog

## 2.30
- AGGIORNAMENTO PROMPT: dopo un merge che cambia prompt o riferimento (e copia nel runtime), si riavvia un nuovo Timoniere via handoff (`j-cli-p tim --sostituisci`), senza chiudere i subagenti.

## 2.29
- MAPPE WAYFINDER: il Timoniere lancia le mappe con un subagente; ticket come sub-issue, file `docs/wayfinder/<slug>.md`, Project «<repo> · Wayfinder» (PR console-platform #140).

## 2.28
- COSTI TOKEN: riga nel nucleo per controllare i costi con `journal-cli usage` (sezione `guide costi`).

## 2.27
- PR e merge autonomi: i subagenti aprono PR non in bozza, il Timoniere controlla e mergia.
- Criteri di consenso del programmatore (deploy, sudo, segreti, migrazioni, scelte di sicurezza/prodotto, test rossi, scope, spesa token).
- Modello scelto dal Timoniere; revisione cross-modello non bloccante decisa da lui.
- Attesa in background dopo ogni consegna.
