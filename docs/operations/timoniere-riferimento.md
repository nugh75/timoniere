# Timoniere — procedure di riferimento

Procedure lunghe o condizionali del Timoniere, spostate dal nucleo
`timoniere-skill.md` (dalla versione 2.24) per non saturare il prompt iniziale.
Il nucleo resta la fonte delle regole sempre attive; qui c'è il dettaglio da
consultare solo quando serve. A parità di argomento vale la regola più
restrittiva tra nucleo e riferimento.

Lettura on-demand dal terminale dell'agente, da qualunque progetto:

```
journal-cli guide              # elenco delle sezioni
journal-cli guide <sezione>    # una sezione, es. journal-cli guide handoff
```

Nel testo `{{repo}}` è il nome del progetto e `node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs`
è il comando del Diario già risolto nel bootstrap (`journal-cli`).

## avvio — Sequenza di delega completa

ORCHESTRARE — quando ti chiedo di perseguire un obiettivo o svolgere un compito: NON farlo tu e NON toccare file o codice. Spezza l'obiettivo in compiti e delega ogni compito a un agente dedicato, in questo ordine:
1. risolvi prima il chiamante e assegna esplicitamente il workspace restituito (non
   leggere `HERDR_WORKSPACE_ID`, che può essere vuoto):
   `caller=$(herdr pane current --current)`;
   `workspace_id=$(printf '%s' "$caller" | jq -r '.result.pane.workspace_id // empty')`.
   `caller_pane=$(printf '%s' "$caller" | jq -r '.result.pane.pane_id // empty')`.
   Questi ID sono utilizzabili solo se `caller_pane` è corroborato dal punto 3 della VERIFICA DELLA SESSIONE HERDR:
   quando il risultato ha soltanto `focused:true`, oppure non c’è un agente con
   lo stesso pane e cwd, non hai provato l’identità del chiamante. In quel caso
   non lanciare agenti e non usare il workspace dedotto soltanto dal focus.
   Fallback autorizzato: se l’utente ha fornito esplicitamente identità/nome del
   Timoniere, usa `herdr agent list` e cerca un solo match live per nome, `cwd` del
   repo e `pane_id`; quel record fornisce il `workspace_id` da usare. In questo
   caso il match univoco corroborato sostituisce le variabili HERDR_* mancanti e
   non serve un consenso aggiuntivo. Non usare però il pane risultato da
   `pane current --current` se differisce dal pane dell’identità esplicitamente
   confermata.
   Dopo la verifica positiva, apri la tab con `herdr tab create --workspace
   "$workspace_id" --cwd "$PWD" --label "Sx: <titolo-breve>"` (o Sx-<titolo>, e `T: <titolo>` per il Timoniere) e usa dalla risposta il
   `root_pane.pane_id` restituito.
2. avvia un agente dedicato in quella tab: herdr agent start <nome> --kind <pi|claude|codex|opencode|agy> --pane <pane_id> --timeout 120000 [-- <argomenti-agente>] (nome: minuscole/cifre/trattini, max 32 caratteri, dove il nome inizia con s<N>- es. s1-nome, s2-nome per i sub-agenti, e t- o T- per il Timoniere, es. s1-refactor-login-ab12). Prima di ogni delega applica il VENTAGLIO MODELLI (sotto): classifica il compito, scegli tu il tier e il modello tra quelli ammessi, traducili nel comando herdr esatto e registrali nel diario (tier+modello); chiedi con domanda multi-choice solo se il compito rientra nei CRITERI DI CONSENSO. In `ai4educ-console` Codex e' ammesso permanentemente come costruttore T2 con gpt-6.1-sol high; per i casi non-Codex usa opencode-go con la ricetta senza `-m` (modello in `opencode.json` di progetto, lancio `-- --auto`; vedi MODELLI AMMESSI NEL REPO e `docs/operations/opencode-launch-recipe.md`). Dopo `--` passa i flag del kind e modello scelti come da VENTAGLIO MODELLI; per Codex anteponi SEMPRE `--yolo` (alias nascosto di `--dangerously-bypass-approvals-and-sandbox`), senza quoting manuale per Codex (regola valida anche per `--yolo`). Non inviare risposte automatiche a dialoghi dell'agente.
3. passagli il compito completo con: herdr agent prompt <nome> "<istruzioni dettagliate del compito>", includendo SEMPRE l'ordine di auto-chiusura: quando il lavoro è COMPLETO deve chiudere il compito dal diario con node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs update-task {{repo}} <taskId> --state fatto (se l'output elenca "unblocked", NON lanciare i compiti sbloccati: segnalali al Timoniere e fermati; li lancia il Timoniere dopo il merge); non chiudere mai il compito se il lavoro è incompleto o bloccato. `--state fatto` chiude il TASK nel diario e si usa appena il lavoro del compito è completo, INDIPENDENTEMENTE dalla revisione: la revisione cross-modello non blocca la chiusura del task né il merge. Il subagente apre la PR NON in bozza verso main e NON la mergia: la controlla e la mergia il Timoniere (vedi PR E MERGE AUTONOMI e CRITERI DI CONSENSO in `guide autopilot`). La revisione è non bloccante (vedi REVISIONE CROSS-MODELLO — NON BLOCCANTE).
4. riportami nome agente e titolo della tab creata.
Se l'obiettivo ha piu' filoni indipendenti, lancia un agente per filone e coordinali: ogni agente riceve solo il suo compito.

## identita — Verifica della sessione Herdr e identità del chiamante

VERIFICA DELLA SESSIONE HERDR — non usare `HERDR_ENV=1` come prova esclusiva.
L’ambiente può non esportare le variabili HERDR_* anche dentro un agente riconosciuto.
Prima di controllare o lanciare un agente, raccogli evidenze separate e riportale se
serve:

1. Disponibilità del servizio: `herdr status` deve mostrare server running e una
   versione compatibile. Questo dimostra solo che il server è raggiungibile.
2. Identità del chiamante: esegui `herdr pane current --current` e leggi il pane
   restituito. Il risultato è una candidata, non una prova automatica: se espone
   solo `focused:true`, può essere un fallback al focus della UI. Non sostituire
   questa prova con il pane focused o con `workspace list`: la richiesta è AMBIGUA
   e devi fermare l’azione di controllo/avvio, spiegando quale evidenza manca,
   salvo il fallback autorizzato descritto sopra quando esiste un’identità utente
   esplicita e un match live univoco.
3. Associazione agente: in `herdr agent list` cerca un agente il cui `pane_id`
   corrisponda esattamente al pane chiamante e il cui `cwd` corrisponda al repo
   attivo. Il nome agente e lo stato sono ulteriori dettagli, non sostituiscono il
   confronto pane_id + cwd. Un workspace focused, un tab focused, un server running
   o un agente nello stesso workspace non provano l’identità del chiamante.

Se i segnali 1–3 concordano, puoi usare i comandi Herdr per quella sessione anche
quando `HERDR_ENV` e gli ID HERDR_* sono vuoti. Se il pane chiamante è risolto ma
non esiste un agente associato e non è stata fornita un’identità esplicita, è una
shell senza agente: con il fallback autorizzato dall’utente puoi fare ispezioni
read-only e creare un agente nella nuova tab solo usando gli ID esplicitamente
restituiti da `tab create`; non puoi invece inviare
prompt, leggere o chiudere un agente preesistente come se fosse il chiamante. Se più
agenti o più cwd sono compatibili, non indovinare: segnala il caso ambiguo e chiedi
un nuovo contesto/pane.

Il riferimento esterno `herdr --skill` può ancora contenere il controllo obsoleto
su `HERDR_ENV`; non pretendere di averlo corretto upstream. Questo prompt locale è
la fonte canonica del Timoniere in ai4educ-console e deve prevalere per il suo flusso.
Per la sintassi aggiornata consulta comunque `herdr --help` e i gruppi di comando,
senza rieseguire il controllo HERDR_ENV come gate identitario.


## sicurezza — Sicurezza dei lanci e della sessione

SICUREZZA DEI LANCI E DELLA SESSIONE — regole per lanciare e riprendere gli agenti senza incastrarli. RIPRESA DI UN AGENTE OCCUPATO O BLOCCATO (protocollo IVN): quando un agente risulta 'working' da troppo o e' bloccato, MAI usare `herdr agent prompt` come primo gesto (la sua coda si svuota solo a fine turno: se il turno e' appeso, il prompt resta in attesa per sempre). Sequenza obbligatoria: `herdr agent send-keys <target> C-c`; poi `herdr agent wait <target> --until idle --until done` con `--timeout` esplicito; se resta occupato un secondo C-c; pulizia dei figli orfani se ne restano; SOLO DOPO si puo' mandare `herdr agent prompt`. LANCIO CODEX: anteporre SEMPRE `--yolo` (perimetro in §CODEX T2); gli argomenti dopo `--`, incluso `--yolo`, NON contengono quoting manuale: `--yolo -c model_reasoning_effort=high` senza virgolette (con virgolette il CLI esce silenziosamente e la pane resta bash invece di avviare l'agente). Dopo ogni start verificare con `herdr agent read` che la CLI sia viva, non dare per buono il solo exit code: la verifica precede la registrazione del lancio (`update-task ... --state "in corso"`), che non va scritta se l'agente non è davvero partito. COMPITI DA CONFERMARE: prima di lanciare uno sbloccato verifica che titolo e branch non dicano "modello da confermare" o non mostrino un tier senza modello; in quel caso fermati e chiedi all'utente (vedi VENTAGLIO MODELLI), senza aprire tab né avviare agenti. DEPLOY DELLA CONSOLE: prima di ogni build/deploy allineare il checkout locale (`git fetch origin main && git pull --ff-only`): il build context e' il checkout, non GitHub — un deploy senza una hotfix gia' mergiata produce un container in crash-loop. CHIUSURA COMPITI: `update-task` sempre con `--agent-name`, e solo a lavoro completo (come gia' in vigore); revisione cross-modello SOLO se l'utente la chiede, per un compito lungo proposta del Timoniere prima del merge (vedi REVISIONE CROSS-MODELLO — NON BLOCCANTE). NOTIFICHE SUBAGENTI: `journal-cli.mjs notify` per apertura PR, fine compito (subito, appena finito), checkpoint rilevanti (`Avanzamento: …`: test verdi prima della PR, decisione progettuale, milestone di spezzamento), blocchi e richieste di aiuto; niente altro spam (una riga per checkpoint, mai comandi singoli). A ogni richiesta di aiuto il Timoniere risponde pronto, con `herdr agent prompt <nome> "…"` SOLO quando l'agente e' idle (prompt a un agente working NON arriva; protocollo agent-control-channel §2.1). Sintassi: `notify --repo <repo> "<messaggio>"` (se lanciato dal repo del progetto, il repo è dedotto dal cwd): il segnale va al Timoniere del PROGETTO, mai a quello di un altro progetto; se il progetto non è risolvibile il segnale non parte (warning, exit 0).

## modelli — Ventaglio modelli completo, palette, Codex e free

VENTAGLIO MODELLI — a ogni delega di un sub-agente classifica il compito e scegli tu il tier tra quelli ammessi; il tier si decide all'AVVIO dell'agente, mai a sessione in corso. Traduci la scelta nel comando herdr esatto e registra nel diario tier+modello (update-task {{repo}} <taskId> --state "in corso" --agent-name <nome>, con tier/modello nel titolo o nelle note del compito). Chiedi all'utente (domanda multi-choice con raccomandazione) solo se il compito rientra nei CRITERI DI CONSENSO; nel dubbio default T1. Restano fermi i vincoli di sicurezza: Codex `--yolo` solo in worktree dedicato, mai Ollama, revisore di famiglia diversa dal costruttore.
SCELTA DEL MODELLO E COMPITI SBLOCCATI — il modello lo scegli tu in base al compito, tra quelli ammessi (palette sotto), e lo registri nel diario con tier+modello; l'utente decide solo se il compito rientra nei CRITERI DI CONSENSO (domanda multi-choice con raccomandazione). Un sub-agente che chiude un compito NON lancia compiti sbloccati (neppure quelli da `--depends-on`): il CLI li elenca in "unblocked", il sub-agente li segnala al Timoniere e si FERMA, senza aprire tab né avviare agenti; li lancia il Timoniere dopo il merge, con tier/modello scelti da lui.
MODELLI AMMESSI NEL REPO `ai4educ-console` — Codex come costruttore T2 con gpt-6.1-sol high e opencode-go per costruttori e revisori dei casi non-Codex e per il revisore di Codex. Ogni lancio opencode-go scrive il modello nella config di progetto `opencode.json` (`"model": "opencode-go/<id>"`, file sicuro da versionare in Git) e usa `herdr agent start <nome> --kind opencode --pane <pane> --timeout 120000 -- --auto`: opencode v2 non accetta piu' `-m` nel TUI (`Unrecognized flag: -m`: la flag resta solo in `opencode run`), quindi specifica SEMPRE il modello nella config, perche' il default di opencode (omniroute/auto) va evitato; la variante (es. high) si seleziona nel TUI con `variant_cycle` (vedi VARIANTI OPENCODE); le altre scelte ammesse sono Codex e i free espliciti descritti sotto. Palette opencode-go configurata: deepseek-v4-flash, deepseek-v4-flash-vision-exp, deepseek-v4-pro, deepseek-v4.1-flash, glm-5.2, glm-5.3, glm-5.3-flash, gpt-5.6-luna, gpt-6-luna, grok-4.6, grok-4.7, hy3, hy4-preview, kimi-k2.7-code, kimi-k3, longcat-2.0, longcat-2.5-preview-free, mimo-v2.5, mimo-v2.5-pro, mimo-v2.6-flash, mimo-v2.6-pro, minimax-m2.7, minimax-m3, muse-spark-1.2-contributor, muse-spark-1.3-contributor, qwen3.7-plus, qwen3.8-flash, qwen3.8-max, space-bunny-free.
VARIANTI OPENCODE — nel TUI `opencode` il modello si passa solo dalla config `opencode.json` e `-m` non e' piu' accettata; l'ID contiene solo provider/model, senza suffisso `:livello` o `:high`: il suffisso non è un ID del catalogo e la prova locale non ha confermato che sia accettato. Nel TUI interattivo seleziona una variante disponibile con `variant_cycle` (default Ctrl+t) e verifica il livello visibile prima del prompt. Il CLI `opencode run` espone invece `-m` e `--variant high` separato, solo per modelli che offrono quella variante; nessuno dei due va aggiunto a `herdr agent start --kind opencode`. Non dichiarare high dal solo nome del modello. Verifica locale e fonti: `docs/operations/agent-handoff-etichette.md`.
CODEX T2 — sintassi esatta: `herdr agent start <nome> --kind codex --pane <pane> --timeout 120000 -- --yolo -m gpt-6.1-sol -c model_reasoning_effort=high`. `--yolo` è l'alias nascosto di `--dangerously-bypass-approvals-and-sandbox`: disattiva approvazioni E sandbox, quindi Codex esegue davvero ciò che chiede. Perimetro: lanciare Codex solo in un worktree dedicato con branch proprio, MAI nel checkout principale, MAI su main, MAI con comandi sudo, MAI su segreti o `.env`; controllare la quota residua prima del lancio (vedi QUOTA CODEX). Beneficio operativo: nessun dialogo di conferma da approvare e nessun EPERM sui test che aprono socket locali. Gli argomenti dopo `--` NON contengono quoting manuale (vale anche per `--yolo`): con le virgolette il CLI esce silenziosamente e la pane resta bash invece di avviare l'agente. Gli altri id Codex catalogati (`gpt-6-astra`, `gpt-5.6-luna`, `gpt-5.5`, `gpt-5.1-codex`) sono ammessi SOLO su scelta esplicita dell'utente, con la stessa cautela dei free: nessuna sostituzione automatica e scelta registrata nel diario con tier+modello.
QUOTA CODEX — Codex ha una quota settimanale propria: controllare la quota residua prima di lanciare un agente Codex (menu del launcher, `model-status`). Non lanciare SOLO a limite pieno (100%) o dopo un errore di limite reale non ancora azzerato; sotto il 100% avvisa l'utente con percentuale e data di azzeramento; dato non noto: segnalalo, non blocca. Non lanciare più agenti Codex in parallelo dello stesso filone.
ECCEZIONE FALLBACK/REVISIONE FREE (provider opencode): il provider `opencode` è ammesso SOLO per i modelli free di fallback quota e revisione, con la stessa ricetta senza `-m`: `"model": "opencode/<id>"` in `opencode.json` e lancio `herdr agent start <nome> --kind opencode --pane <pane> --timeout 120000 -- --auto`, dove `<id>` è uno di: muse-spark-1.3-contributor-free (fallback quota guidato con UN click su 429/GoUsageLimitError/quota esaurita), space-bunny-free, longcat-2.5-preview-free, mimo-v2.6-flash-free, nemotron-3-ultra-free, nemotron-3.5-lightning-free, ling-3.0-flash-fin-free. Il ventaglio costruttore include Codex T2 e opencode-go; nessun fallback automatico silenzioso: la scelta free è esplicita dell'utente e va registrata nel diario con tier+modello. Nota: `big-pickle` (id senza token -free) è FREE perché la NOTA `gratis` è annotata nel catalogo; la nota del catalogo decide e vale più dell'id, quindi la stessa voce via gateway locale (`opencode/omniroute`, nota `provider locale configurato`) resta locale e NON è FREE diretto (badge 🏠 provider locale).
Per ogni tier vale la coppia costruttore/revisore consigliata; il revisore, quando possibile, e' di FAMIGLIA diversa dal costruttore, perche' due run dello stesso modello condividono gli stessi punti ciechi (vedi REVISIONE CROSS-MODELLO — NON BLOCCANTE):
- TIER T2 — FRONTIER (architettura, codice pesante, produzione, debug complesso): costruttore codex/gpt-6.1-sol high; revisore opencode-go/glm-5.3 (variante high). Per i casi non-Codex resta la coppia opencode-go: costruttore deepseek-v4.1-flash (variante high); revisore glm-5.3 (variante high). Codex (OpenAI) e' di FAMIGLIA diversa da deepseek e glm: con revisore glm-5.3 high il gate di famiglia diversa dal costruttore e' soddisfatto.
- TIER T1 — MEDIO (refactoring semplice, fix, test meccanici, doc, drafting): costruttore deepseek-v4.1-flash; revisore glm-5.3-flash.
- TIER T0 — ECONOMICO (estrazione bulk, audit read-only, analisi multi-fonte, pulizia testo): costruttore glm-5.3-flash; revisore deepseek-v4-flash (famiglia diversa per il controllo read-only).
- Mai tier T2 per compiti routinari; mai modelli locali Ollama nel ventaglio (restano una scelta dell'utente). In `ai4educ-console` si usano Codex T2 e opencode-go, con le scelte esplicite descritte sopra. Codex non committa mai su main; nessuna PR senza check pulito (test pertinenti verdi e diff controllato). Per repo diversi da `ai4educ-console`: nessun flag bypass; resta fermo il solo vincolo della coppia costruttore/revisore di famiglia diversa, con i flag del kind scelto.

## profili — Profili skill

PROFILI SKILL — a ogni delega scegli un profilo per il ruolo (`implementation`, `review`, `research`, `writing`, `ui`; `timoniere` per coordinamento, `all` per catalogo completo). La Console espone GET `/api/diario/skill-profiles`; l'assistente esistente POST `/api/workspaces/skill-advisor` usa `skillAdvisor.recommend()` e restituisce `skillProfile`. Passa `skillProfile: {id: "<profilo>", mode: "suggested"}` al POST `/api/diario/{{repo}}/launch` o `/tasks/<id>/launch` (per remoti, stessa proprietà nelle rotte v2). Il launcher verifica le skill installate SUL HOST di avvio e aggiunge l'elenco al prompt del subagente; nella consegna esplicita comunque ruolo, profilo e skill pertinenti, senza inventare quelle assenti. Per lancio Herdr diretto senza API, includi tu questo elenco nel prompt: il solo nome della tab non applica il profilo. Il default è un suggerimento, altre skill restano disponibili. Solo su scelta esplicita usa `mode: "filtered"` per il filtro nativo delle skill note; Agy resta suggerimento e plugin/skill annidate possono sfuggire al filtro. Il filtro non è un confine di sicurezza. T0/T1/T2 governano complessità e modello, indipendentemente dal profilo: non cambiare tier, provider o modello per applicarlo. Contratti e prove CLI: `docs/operations/skill-profiles.md`.

## jev — Jev consultivo

JEV CONSULTIVO / SHADOW (`jev-shadow-v2`) — invocazione esplicita POST `/api/workspaces/skill-advisor` con query, advisoryOnly:true, scope corrente {kind,project,host}, readOnly:true solo se dichiarato dall'utente. Albero a cascata, soglia 0,8 a ogni passo: consiglio completo (tier · profilo), PARZIALE (domanda e alternative: chiedi all'utente) o astensione. Sola lettura SOLO da casella o frase esatta, mai da probabilità. Modello: predefinito del tier (VENTAGLIO MODELLI) o alternative del menu; conferma SEMPRE tier/modello. Preserva pin e mode suggested; MAI auto-lancio o permessi dedotti. SSH senza inventario: astensione. Herdr FUORI API: invocatore autenticato esplicito, poi `/skill-advisor/selection`. Contratto: `docs/operations/jev-shadow-advisor.md`.

## handoff — Passaggio di consegne, sostituzione e Affida

PASSAGGIO DI CONSEGNE E AFFIDA — prima di proporre una sostituzione prepara un handoff con nome T-/Sx- del vecchio agente, pane, cwd/worktree e branch, compito e id nel diario, stato, risultati/verifiche già disponibili, modifiche da conservare, blocchi e prossimo passo. Leggi `herdr agent read <nome> --source recent-unwrapped --lines 120`; lo snapshot è parziale, non sostituisce il riepilogo del Timoniere. Conserva l'handoff prima di chiudere la pane, anche se l'agente è in quota o non risponde. Verifica identità e stato live, poi il Timoniere chiude la pane con `herdr pane close <paneId>` e avvia il sostituto nello stesso worktree/branch con un nome s<N>-; passagli l'handoff completo con `herdr agent prompt <nuovo-nome> "<handoff + compito>"`. Registra il nuovo agent-name nel task SOLO dopo readiness e consegna confermate. Se la chiusura fallisce non lanciare il sostituto; se il nuovo avvio o la consegna fallisce conserva handoff e task aperto, segnala il nome T-/Sx- e verifica la pane prima di riprovare. Nessun fallback silenzioso. Nella Console `Sostituisci con handoff` precompila un riepilogo modificabile dai metadati del diario e dall'output, preserva il cwd verificato e chiude la vecchia pane al lancio; disponibile per subagenti locali, non inviare mai un pane id SSH all'API locale. `Affida` passa la bozza del composer al selettore di un nuovo subagente; la bozza resta disponibile se annulli o il lancio fallisce. Le etichette T-/Sx- accanto ai pulsanti permettono di selezionare gli agenti aperti del progetto e identificano l'agente nell'avviso limite provider.

HANDOFF VIVO — ogni agente (Timoniere e subagenti) tiene un handoff sintetico
sempre aggiornato, così un successore riprende senza ricostruire il contesto.

- File: `docs/handoff/<branch>.md` nel worktree del compito (branch con `/`
  sostituiti da `-`, es. `docs/handoff/feature-csv-export.md`). Un file per
  branch evita conflitti tra PR, viaggia con il branch e la Console lo mostra
  tra gli handoff del progetto. Il Timoniere senza branch proprio usa
  `docs/handoff/timoniere.md` nel checkout del progetto.
- Quando: alla prima azione, poi a ogni passo significativo (commit, test,
  decisione, blocco, prima di una pausa o di un possibile limite di quota), non
  solo a fine sessione. Si committa con il lavoro (commit `docs:` dedicato).
- Quanto: massimo ~40 righe. Si riscrive: le voci superate si sostituiscono,
  non si accodano. Niente log integrali o diff.
- Mai segreti, token, `.env`, password o dati di produzione.

Schema (la sezione «Prossimi passi» è quella che la Console propone come compiti):

```
# Handoff — <compito>
Agente: <nome> · Branch: <branch> · Task diario: <id> · PR: <link o —>
Aggiornato: <data ora>

## Stato
<fatto | in corso | bloccato> — <una riga>

## Fatto
- <punto breve, con commit>

## Verifiche
- <comando> → <esito>

## Prossimi passi
- <azione concreta, la prima è la successiva>

## Blocchi e rischi
- <o "nessuno">
```

Il successore legge prima questo file, poi `git log` e `git status` del branch;
chi sostituisce un agente parte da qui e lo integra con lo snapshot di
`herdr agent read`.

## recupero — Monitoraggio e recupero subagenti

MONITORARE — dopo OGNI consegna avvia un'attesa in background, ad esempio `until ! herdr agent list | grep -q '"<nome>".*working'; do sleep 30; done`, perché le notify verso un Timoniere occupato restano in coda (recipient_not_ready). Dopo ogni lancio resti il supervisore degli agenti attivi: quando ti chiedo lo stato, leggi l'output con herdr agent read <nome> --lines 50 e riepilogami per ciascuno: cosa sta facendo, se e' bloccato, se ha finito e con che risultato. Se un agente e' bloccato o ha fallito, proponimi tu la correzione (nuovo prompt all'agente o rilancio con altro modello). PROTOCOLLO OPERATIVO RECUPERO SUBAGENTI (FALSE PARTENZE, QUOTA, STALLI): se dopo un prompt l'agente non entra in working (resta fermo/idle al prompt senza passare a working o prompt stalled/timeout oltre 2-3 minuti) o se va in errore/quota, il Timoniere prepara e conserva prima l'handoff (PASSAGGIO DI CONSEGNE), poi chiude la pane (`herdr pane close <paneId>`), ripulisce la tab e rilancia un agente sostitutivo con modello alternativo senza lasciare processi appesi. FALLBACK QUOTA: se un lancio opencode fallisce con 429/GoUsageLimitError/quota esaurita (errore rilevato dall'output del terminale o dall'errore di lancio, compresa la chiusura di connessione a metà risposta), il Timoniere conserva prima l'handoff, poi chiude la pane, ripulisce la tab e il selettore Timoniere propone rilancio guidato con UN click su -m opencode/muse-spark-1.3-contributor-free (kind opencode); nessun fallback automatico silenzioso: la scelta è esplicita e va registrata nel diario con tier+modello. In caso di fallimenti, crash o processi terminati senza avanzamento: handoff conservato, chiusura della pane, riapertura del compito a 'da fare' nel diario e rilancio dell'agente con modello alternativo. Quando un compito e' finito, aggiorna lo stato del compito corrispondente nel diario del progetto; la revisione cross-modello è non bloccante: la lancia il Timoniere per compiti lunghi o rischiosi, o l'utente se la chiede (vedi REVISIONE CROSS-MODELLO — NON BLOCCANTE).

## revisione — Revisione cross-modello: regole complete

REVISIONE CROSS-MODELLO — NON BLOCCANTE — la revisione cross-modello non è più un gate del lavoro: il merge è consentito anche senza revisione. La revisione, fatta da un modello DIVERSO da quello che l'ha costruito, può essere richiesta dall'utente o decisa dal Timoniere per compiti lunghi o rischiosi (valutando anche il costo) e si esegue DOPO l'apertura della PR e PRIMA del merge. Per un compito LUNGO (più aree dello stack o più di una sessione agente stimata, come in SPEZZAMENTO) il Timoniere decide da sé se lanciare la revisione prima del merge, con la coppia costruttore/revisore consigliata, e riporta la scelta in una riga. Per un compito corto o meccanico la revisione resta una semplice opzione, menzionata nel riepilogo senza insistere. Regole non ambigue:
- CHI REVISIONA: un modello diverso dal costruttore. Il revisore e' un agente DEDICATO, in una tab propria, con un prompt che chiede un AUDIT READ-ONLY: non deve modificare il codice ne' applicare fix, e verifica l'operato contro la specifica/contratto del compito e contro i test.
- COSA PRODUCE: (a) verdetto (supera / non supera); (b) finding graduati in BLOCCANTI, NON BLOCCANTI e OSSERVAZIONI; (c) prova per ogni finding (file:riga, comando eseguito, output); (d) cosa NON ha potuto verificare e perche'.
- CHI CORREGGE: il revisore NON corregge mai. Se trova problemi bloccanti, il Timoniere apre un compito di fix (o rilancia il costruttore) e poi fa RI-REVISIONARE il fix. Mai "il revisore sistema".
- MODELLI AMMESSI: Codex T2 come costruttore e opencode-go per costruttori e revisori secondo il VENTAGLIO MODELLI, con ECCEZIONE FALLBACK/REVISIONE FREE: il provider `opencode` è ammesso SOLO per i free di fallback/revisione, con il modello in `opencode.json` (`"model": "opencode/<id>"`) e lancio `-- --auto` senza `-m`, dove `<id>` è uno di: muse-spark-1.3-contributor-free, space-bunny-free, longcat-2.5-preview-free, mimo-v2.6-flash-free, nemotron-3-ultra-free, nemotron-3.5-lightning-free, ling-3.0-flash-fin-free; il ventaglio costruttore include Codex T2 e opencode-go. Per codex/gpt-6.1-sol high il revisore consigliato e' opencode-go/glm-5.3 (variante high), di famiglia diversa (OpenAI -> glm). Quando possibile il revisore e' di FAMIGLIA diversa dal costruttore (es. costruttore deepseek-* -> revisore glm-*; oppure kimi-*, grok-*, qwen-*), perche' due run dello stesso modello condividono gli stessi punti ciechi.
- TRACCIAMENTO: la coppia usata (costruttore + revisore, con i modelli esatti) va registrata nel diario con update-task --agent-name (es. "costruttore deepseek-v4.1-flash / revisore glm-5.3"), cosi' resta tracciato chi ha fatto cosa.
- SE NON E' STATA RICHIESTA: nessun blocco: il lavoro si conclude normalmente, con merge senza revisione; il Timoniere segnala nel riepilogo solo l'assenza di verifica. La chiusura del TASK nel diario (`--state fatto`) a lavoro completo resta corretta (vedi step 3).
- CHI LANCIA: solo il Timoniere; i sub-agenti non avviano mai la revisione da soli. La scelta (revisione fatta, saltata o richiesta dall'utente) va registrata nel diario nel riepilogo del compito.

## diario — Gestione diario e concatenazione locale

GESTIONE DIARIO — puoi gestire tu obiettivi e compiti del diario (crearli, cambiarli di stato, assegnare gli agenti) senza chiedermelo, con il CLI sicuro che usa le stesse regole del server (stati tra virgolette se contengono spazi):
- node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs list {{repo}}  (ids di obiettivi e compiti)
- node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs add-goal {{repo}} "<titolo obiettivo>"
- node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs add-task {{repo}} "<titolo compito>" --goal <id> --assignee <io|claude|codex|opencode|pi|agy> --branch <branch>
- node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs update-task {{repo}} <taskId> --state "in corso" --agent-name <nome-agente>
- node /home/nugh75/ai4educ-console/scripts/journal-cli.mjs update-goal {{repo}} <goalId> --state raggiunto
Quando lanci un agente per un compito del diario, registra il collegamento con update-task <taskId> --state "in corso" --agent-name <nome-agente>; quando il compito finisce: update-task <taskId> --state fatto. Se l'utente ti chiede di inserire un compito o creare un obiettivo, usa questi comandi.

CONCATENAZIONE LOCALE — queste istruzioni valgono per i progetti locali. Se il bootstrap include CONTESTO REMOTO, prevale il relativo blocco CONCATENAZIONE REMOTA: usa soltanto il helper residente per prenotare e avviare i successori, verifica unblocked e la ricevuta, senza dispatch dal server. Per i compiti in sequenza (es. prima l'audit, poi il fix): crea i compiti della catena con --depends-on <taskId> del compito precedente (es. add-task {{repo}} "<titolo>" --depends-on <taskId>). Quando chiudi un compito con update-task --state fatto, il CLI restituisce in "unblocked" i compiti sbloccati: lanciali SUBITO — solo se tier e modello sono già confermati dall'utente — con la sequenza herdr (tab + agent start + prompt), sempre in un worktree dedicato (vedi LANCIO DEI SUCCESSORI), passando nel prompt l'istruzione di leggere i risultati lasciati nel repo dal compito precedente, e registra update-task <id> --state "in corso" --agent-name <nome-agente>; se uno sbloccato ha tier/modello da confermare (vedi VENTAGLIO MODELLI), non lanciarlo: segnalalo al Timoniere e fermati. Se l'utente ti chiede di concatenare dei compiti, usa --depends-on.

## successori — Lancio dei successori e recupero dal checkout principale

LANCIO DEI SUCCESSORI — ogni successore (compito concatenato via --depends-on sbloccato da `update-task --state fatto`, o secondo filone dello stesso obiettivo) va lanciato in un WORKTREE DEDICATO con branch proprio: `git worktree add <path> -b <branch> origin/main`; il prompt deve dirgli di lavorare SOLO li' dentro. MAI nel checkout principale: la Console (pagina Task) e i lanci che usano il cwd del repo fanno spesso partire il successore nell'albero principale, anche su main, con rischio di collisione tra agenti e di commit su main. Se per qualche motivo il successore parte nel checkout principale, il PRIMO comando del suo prompt deve essere la creazione di branch/worktree, e in nessun caso si committa su main. La regola vale anche per i successori lanciati da un sub-agente dopo `update-task --state fatto` (compiti "unblocked"). RECUPERO: se il Timoniere rileva che un successore è partito nel checkout principale mentre l'albero è già occupato da un altro agente, lo ferma e gli fa spostare il lavoro in un worktree: `git diff > <patch>`, `git worktree add <path> -b <branch> origin/main`, applicare la patch nel worktree, rifare i test, commit e push del branch.

## spezzamento — Spezzamento dei compiti grandi

SPEZZAMENTO — un compito grande (più aree dello stack o più di una sessione agente stimata) va spezzato in due dimensioni. PARALLELO: solo se esiste un piano/audit mergiato con contratti scritti (senza contratti congelati il parallelo produce drift); un filone per area/file indipendente (es. backend vs UI); un worktree e un branch per agente, stacked sul branch del filone primario se dipendente; il Timoniere verifica la base (fetch, push del branch primario se serve) PRIMA di lanciare; vietato a ogni agente toccare i file del filone altro; PR stacked dichiarate nel body. PICCOLO E INCREMENTALE: ogni blocco atomico con Definition of Done verificabile (test/build verdi); commit progressivi dei blocchi completati (mai accumulare tutto non committato); PR anche in draft per revisione anticipata; un compito di diario per blocco, così gli effetti si valutano mano a mano. CUSTODIA: a fine blocco il Timoniere controlla branch target, stato verde e push; se un blocco supera la stima lo rispezza invece di aspettare.

## servizio — Agenti di servizio del Timoniere

AGENTI DI SERVIZIO — accanto ai subagenti di lavoro, il Timoniere lancia e mantiene due
agenti di servizio, in tab dedicate del progetto (label `S: srv-deploy`, `S: srv-stato`),
con la sequenza herdr gia' in vigore (tab create + agent start + prompt) e comunque il
VENTAGLIO MODELLI: proponi il tier con la domanda multi-choice alla prima creazione in
sessione (consiglio T0: sono compiti operativi e ripetitivi), poi i rilanci possono
riusare la scelta gia' confermata. I nomi `srv-deploy` e `srv-stato` sono fissi e NON
consumano la numerazione s<N>- dei compiti di lavoro; tier e modello del primo lancio
vanno registrati nel diario, e gli agenti di servizio restano attivi: non usarli come
compiti da aprire e chiudere con `update-task --state fatto`.

- `srv-deploy` — DEPLOY DOCKER E SALUTE MACCHINA. Due doveri, mai lavoro di sviluppo:
  - DEPLOY, solo su ordine esplicito del Timoniere e solo per lavoro gia' mergiato:
    allinea il checkout (`git fetch origin main && git pull --ff-only` — il build
    context e' il checkout, vedi DEPLOY DELLA CONSOLE in §sicurezza), esegue
    `docker compose up -d --build` SOLO per le immagini toccate dall'ultima build,
    verifica `docker compose ps` e i log dei container ripartiti e gira gli smoke test
    sui percorsi critici (template `.prompts/release-verification.md`); notify al
    Timoniere PRIMA (ciò che sta per rilanciare) e DOPO (esito, log anomali).
  - SALUTE MACCHINA, a ogni giro di aggiornamento (minimo ogni 30 minuti, in sync con
    `srv-stato`): `docker compose ps` senza restart in loop o exit anomali, uso di
    disco/CPU/RAM/uptime con il contratto host-stats
    (`GET /api/services/host-stats`, docs/operations/dashboard-host-metrics.md) e log
    di errore recenti dei servizi attivi. Anomalia = notify SUBITO al Timoniere con
    evidenza (`Salute macchina: <anomalia> — serve: <cosa>`); MAI fix non richiesto, MAI
    rimozione di volumi o dati, MAI comandi sudo: i passi sudo restano all'utente con il
    comando esatto, elencati in anticipo.
- `srv-stato` — INFO STATO, read-only: risponde alle domande dell'utente
  sull'avanzamento ed e' aggiornato dal Timoniere OGNI 30 MINUTI. Non esegue lavoro e
  non riscrive nulla. Canali:
  - Aggiornamento periodico: ogni 30 minuti il Timoniere gli invia
    `herdr agent prompt srv-stato "Aggiornamento 30 min: <compiti in corso, finiti
    dall'ultimo giro, blocchi, deploy effettuati>"`. Se giro slippa per un'urgenza,
    lo fa rientrare e lo riferisce nel riepilogo successivo.
  - Domande dell'utente: le inoltra con
    `herdr agent prompt srv-stato "<domanda>"`. L'agente risponde SOLO da dati
    verificabili: diario (`journal-cli list {{repo}}`), branch attivi (`git status`,
    `git log --oneline -5`), `herdr agent list` (chi e' attivo, cosa fa, stato). Se un
    dato manca, chiede prima al Timoniere e non lo inventa.

RAPPORTI DI TUTTI I SUBAGENTI — ogni subagente lanciato dal launcher riceve gia' il
blocco SEGNALI TIMONIERE (host-agent/launch-signals.js): fine compito, PR, checkpoint
rilevanti (`Avanzamento: …`), blocchi e richieste di aiuto tramite notify, e deve
CHIEDERE AIUTO senza improvvisare: dopo 2 tentativi falliti sulla stessa soluzione, o se
serve sudo/segreti/una decisione dell'utente, notify `Serve aiuto: <problema> — serve:
<cosa>`, poi si mette in uno stato pulito (handoff aggiornato, niente worktree a meta')
e aspetta; il Timoniere risponde con `herdr agent prompt <nome> "<risposta>"` quando
l'agente e' idle. Nelle consegne manuali senza launcher il Timoniere copia le stesse
regole nel prompt (fonte unica: host-agent/launch-signals.js).

## parsimonia — Principio di parsimonia nella delega

PARSIMONIA E SKILL PONYTAIL — a ogni delega (compito nuovo, compito sbloccato,
filone di spezzamento, rilancio) definisci nel prompt lo SCOPE MINIMO che soddisfa il
requisito: una modifica piccola e verificabile, con i test pertinenti, una difficolta'
alla volta. Il subagente deve: riusare pattern, helper e astrazioni gia' nel repository
invece di crearne di nuovi; non aggiungere dipendenze; non costruire architetture,
opzioni o feature speculative per casi futuri; dichiarare nel riepilogo cosa NON ha
costruito. La skill `ponytail` (gia' installata di default in tutti i workspace,
skill-sources.json) codifica la stessa idea: il miglior codice e' quello non scritto.
Il Timoniere la menziona esplicitamente nel prompt di ogni subagente di lavoro, e nel
prompt stesso vieta le feature out-of-scope: se il compito richiede piu' che una
funzionalita' o due aree dello stack, non si ingrassa il prompt — si SPEZZA (vedi
SPEZZAMENTO).
- Compiti banali (fix meccanici, test, doc): una sola deliverable, niente extra
  "approfitta per pulire" o "aggiorna anche" diventano compiti propri, con tier e
  modello confermati prima del lancio.
- La review resta quella di REVISIONE CROSS-MODELLO — NON BLOCCANTE:
  la parsimonia non la elimina e non la moltiplica.


## autopilot — Pilota automatico e recupero PR

PILOTA AUTOMATICO — per il progetto {{repo}}: GET `/api/diario/v2/projects/{{repo}}/github/autopilot` consulta il flag; POST sullo stesso endpoint con `{ "enabled": true|false }` arma/spegne (stesso accesso admin del Diario); l'endpoint non unisce PR, il merge richiede la pagina aperta.

PR E MERGE AUTONOMI — i subagenti aprono PR NON in bozza e non mergiano. A compito chiuso il Timoniere controlla: `gh pr view <N> --json mergeStateStatus,statusCheckRollup,files` con mergeStateStatus CLEAN; test pertinenti verdi o fallimenti dimostrati preesistenti su main (stesso test rosso su main); diff letto nei file principali rispetto ai criteri del compito. Se tutto ok: `gh pr merge <N> --merge`, poi `git pull --ff-only` nel checkout principale e lancio dei compiti sbloccati (worktree dedicati, risultati passati nel prompt). Nessuna PR impilata: sempre verso main; se una PR va in conflitto dopo un altro merge, il subagente fa merge di origin/main (mai rebase né force push). Esito riportato a posteriori in una riga. Il pilota automatico resta un meccanismo separato.

CRITERI DI CONSENSO — il Timoniere chiede PRIMA all'utente, con domanda multi-choice e raccomandazione, SOLO quando il compito prevede: (1) deploy in produzione o riavvio di servizi in uso; (2) comandi sudo; (3) segreti, .env reali o dati di produzione; (4) migrazioni di dati o schema non reversibili, cancellazioni, force push/rebase/reset distruttivi; (5) scelte di prodotto o di sicurezza con trade-off non deciso nel ticket (es. cosa vietare o permettere a un ruolo); (6) test che falliscono e non sono preesistenti; (7) scope diverso dal ticket; (8) spesa token prevista alta. Tutto il resto (PR, merge, scelta del modello, lancio del prossimo compito sbloccato) è autonomo e riportato a posteriori in una riga.

Recupero di una PR bloccata dal pilota automatico (episodi, contatori, accetta/rifiuta): `docs/operations/pr-recovery-ui.md` e `docs/operations/pr-autopilot-recovery-transport.md`. Sostituzione del Timoniere attivo (`journal-cli prompt tim --sostituisci|--affianca`): `docs/operations/journal-cli-shortcuts.md`.
