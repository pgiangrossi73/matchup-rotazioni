# Scambio tra Matchup Rotazioni e Setter Radar

Le due app sono sullo stesso sito (`pgiangrossi73.github.io`) e quindi, **sullo stesso dispositivo e nello stesso browser**, condividono la memoria (localStorage). Lo scambio usa due chiavi piccole: **ogni app scrive solo la sua e legge quella dell'altra**. Nessuna app legge i dati interni dell'altra (`rotmatch.v1`, `setter-radar-v2`), che cambiano da una versione all'altra.

Questo è il primo pezzo del modello dati comune previsto per il progetto di rilevazione completa.

## `suite.sr.v1` — scritta da Setter Radar (dalla v55)

Aggiornata a ogni salvataggio, solo se cambia qualcosa. La distribuzione (`dist`) viene ricalcolata 1,2 s dopo l'ultima modifica.

| Campo | Significato |
|---|---|
| `ts` | ora dell'ultima scrittura (ms) |
| `v`, `app` | `1`, `"setter-radar"`; `none: true` se non c'è nessuna partita aperta |
| `id` | id della partita live |
| `oppKey`, `oppName` | squadra avversaria (chiave = nome normalizzato, stessa regola di Matchup) |
| `myName` | "La tua squadra" |
| `setter` | `{ key, name, num }` palleggiatrice seguita |
| `set`, `started` | set in corso e se è iniziato |
| `oppStart`, `weServe` | partenza avversaria (P1–P6) e chi batte per primo nel set in corso |
| `serving`, `oppRot`, `score`, `won` | stato attuale: chi batte (`us`/`them`), loro rotazione, punteggio [noi, loro], set vinti |
| `plog` | registro dei punti del set in corso (formato sotto) |
| `sets` | set chiusi: `{ set, score, oppStart, weServe, plog }` |
| `dist` | `{ setter, liveId, rule: { covMin, lead }, rots: { 1..6: { d, nh, nl, lanes: { a, b, c: { sig, warn, cov, nh, nl } } } } }` |

`d` = distribuzione per destinazione (`L` lato 4, `L5` zona 5, `C` centrale, `R` lato 2, `R1` zona 1, `P` pipe, `D` 2° tocco). Corsie di ricezione: `a` = Z1-9-2, `b` = Z6-8-3, `c` = Z5-7-4. `sig` = `fist`, `4`, `2`, `read`. Stessa regola del segnale di Setter Radar (ricezioni contate, solo alzate della palleggiatrice, storico pesato con la partita di oggi).

### Registro dei punti (`plog`)

Stesso formato del conta punti di Matchup, così Matchup lo usa senza conversioni:

- `"U"` punto nostro, `"T"` punto loro;
- `{ fix: "op", to: N }` rotazione avversaria corretta (pulsanti P, cambio della palleggiatrice);
- `{ fix: "sv" }` battuta corretta (⇄);
- `{ fix: "sc", to: [noi, loro] }` punteggio corretto a mano.

Le correzioni vengono registrate confrontando lo stato prima di ogni punto e a ogni salvataggio; "Annulla" toglie anche le voci del registro.

## `suite.mr.v1` — scritta da Matchup Rotazioni (dalla rot-v9)

| Campo | Significato |
|---|---|
| `ts`, `v`, `app` | ora, `1`, `"matchup"` |
| `myKey`, `oppKey`, `oppName` | squadre scelte in Matchup |
| `set`, `phase` | set e fase della Panchina (`sheet`, `live`, `summary`) |
| `myStart` | nostra partenza confermata (null prima della conferma) |
| `oppStart` | rotazione reale avversaria indicata in Matchup |
| `weServe` | chi batte per primo (null al 5° set prima del sorteggio) |

## Regole di collegamento

- Collegamento automatico quando la squadra avversaria è la stessa (`oppKey`) e i dati hanno meno di 6 ore.
- **Setter Radar → Matchup**: con il set iniziato in Setter Radar, Matchup prende partenza avversaria, battuta e punti del set (anche dei set chiusi). Il conta punti di Matchup mostra "Punti da Setter Radar"; si può scollegare per set. Il riquadro "Distribuzione in gara" della Panchina mostra i dati di Setter Radar per la rotazione avversaria del momento. La distribuzione pre-gara resta in Studio e nel Report.
- **Matchup → Setter Radar**: nella preparazione del set, se Matchup ha già la rotazione reale avversaria per lo stesso set, compare "Da Matchup Rotazioni · P… · battono …" con **Applica** (formazione automatica + battuta).
- Le app si aggiornano quando l'altra scrive (evento `storage`) e quando si torna all'app.

## Memoria

Il limite misurato in Chromium è circa **5,24 milioni di caratteri per sito** (chiavi + valori), diviso tra le due app. Se un salvataggio non riesce, compare un avviso rosso in alto; sopra il 90% compare un avviso arancione all'avvio. Lo spazio usato, diviso per app, è nella scheda Dati di entrambe.
