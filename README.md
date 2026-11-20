# Timoniere

Prompt versionato del Timoniere (orchestratore di subagenti per ai4educ-console).

- `timoniere-skill.md`: nucleo sempre attivo (versione nell'intestazione, oggi 2.30). `{{repo}}` e `{{bugReportRule}}` sono placeholder sostituiti dalla Console.
- `docs/operations/timoniere-riferimento.md`: procedure lunghe, lette on-demand con `journal-cli guide <sezione>`.
- `CHANGELOG.md`: cronologia delle versioni.

Ogni modifica: branch, PR, incremento di "Prompt version". Nucleo compatto (max ~1590 parole, 11 KB): i dettagli vanno nel riferimento. Copia in console-platform: aggiornarla a mano da qui. Le modifiche vanno fatte in entrambi i repo nella stessa sessione (console-platform è la copia che la Console usa).
