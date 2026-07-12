# AGENTS.md — Piano Nutrizione & Allenamento (fathergym)

> File di contesto per agenti AI (Cursor / Claude Code).
> Leggi **tutto** prima di modificare `index.html`.
> Lingua del progetto e della UI: **italiano**.

---

## 1. Cos'è il progetto

App personale (single-user) per **tracciare dieta e composizione corporea** di un
uomo adulto sano che si allena in casa. Non è un prodotto multi-utente né un'app
clinica: è uno strumento personale, minimale, "low mental effort".

- **Un solo file**: `index.html` — self-contained (CSS in `<style>`, JS inline).
- **Nessun build step**, nessun bundler, nessun framework.
- **Dipendenze esterne**: solo Firebase via CDN (SDK 8.10.1). Niente altro.
- **Backend**: Firebase Realtime Database, usato solo per il log misurazioni.

L'utente è uno sviluppatore (Node.js, WSL2, Cursor): il codice può essere tecnico,
ma la UI deve restare semplicissima da usare da telefono.

---

## 2. Architettura di `index.html`

### Struttura
- **Barra superiore sticky** (`.sticky-top`): header + toggle piani + toggle ON/OFF.
- **Box RM**: spiega come sono calcolati i target (vedi §3).
- **3 pannelli piano**: `#panel-mantenimento`, `#panel-massa`, `#panel-cut`.
- **Sezioni pasto**: accordion apribili (`.section` + `toggleSection()`).
- **Sezione log**: tabella misurazioni + form, sincronizzata con Firebase.

### JavaScript chiave
- `setPlan(plan)` — mostra un pannello, evidenzia il bottone, mostra/nasconde il
  toggle ON/OFF. Config nell'oggetto `PLANS`. **Solo `massa` e `cut` hanno ON/OFF.**
- `setMassaDay(day)` / `setCutDay(day)` — alternano i blocchi ON/OFF.
- `updateStickyOffset()` — misura l'altezza della barra superiore e imposta la CSS
  var `--sticky-h`, usata come `top` del banner macro sticky. Va richiamata quando
  la barra cambia altezza (cambio piano, resize, orientationchange).
- `toggleSection(header)` — apre/chiude un accordion.
- Firebase: `saveLogEntry()`, `deleteLogEntry(id)`, `clearLog()`, listener `on('value')`
  su `users/${currentUser}/measurements`.

### Piani attuali (valori di riferimento)
| Piano | Kcal | Proteine | ON/OFF | Note |
|---|---|---|---|---|
| Mantenimento | ~2400 | ~145 g | No (giorno unico) | **Default / fase attuale** |
| Massa (lean bulk) | ~2400–2755 | 150 g fisse | Sì | Solo a palestra riaperta |
| Cut | ~1700–2000 | 165 g fisse | Sì | Ultime settimane più strette |

---

## 3. Principi NUTRIZIONALI (canone corretto — NON reintrodurre gli errori)

Questi principi sono già applicati nel file. Sono stati corretti da una versione
precedente che conteneva errori: **non ripristinarli**.

### Come si calcolano i target
1. **Base = RM** (metabolismo a riposo, dalla bilancia bioimpedenza). Valore attuale: **1699 kcal**.
2. **Mantenimento (TDEE) = RM × ~1.4** (lavoro da casa sedentario + allenamento breve
   + famiglia attiva) ≈ **~2400 kcal**.
3. **Massa** = mantenimento + ~200/250 kcal · **Cut** = mantenimento − ~450 kcal medio.

### Regole ferme
- **Proteine alte e FISSE**, anche nei giorni OFF e di sgarro:
  ~1.8 g/kg in mantenimento (~145 g), ~2.2 g/kg in cut (~165 g), fino ~2.3 g/kg (~175 g)
  nella fase finale di cut. **Mai** far crollare le proteine nei giorni OFF.
- **Grassi: pavimento ~0.7–0.8 g/kg (~60 g).** NON esiste una regola "mai sotto 90 g
  per il testosterone" → è un **mito, rimosso**. Non reinserirla.
- **Carboidrati**: riempiono le calorie restanti; più alti nei giorni di allenamento
  (ON), più bassi nei giorni OFF. In deficit si riducono ma **non si azzerano**.
- **Dimagrire = deficit calorico**, non "togliere i carboidrati". Il calo rapido da
  low-carb è **acqua/glicogeno**, non grasso.

### Invariante da rispettare SEMPRE
> **Il banner macro (Proteine / Carbo / Kcal) DEVE corrispondere alla somma dei pasti**
> di quel piano/giorno, e al box `TOTALE`.

Se modifichi un pasto o un macro, **ricalcola e aggiorna sia il banner sia il TOTALE**.
Questo era il bug principale della vecchia versione (banner che non tornava coi pasti).

### Da evitare (pseudoscienza — già ripulita)
- ❌ "Le proteine saziano 2× i carboidrati" (numero inventato).
- ❌ Enfasi su "patate raffreddate / amido resistente" (effetto pratico trascurabile).
- ❌ Claim su ormoni/testosterone legati a soglie di grassi arbitrarie.

### Note di misura
- L'**RM** e il **BF%** da bilancia bioimpedenza sono **stime grezze**: utili per il
  *trend* nel tempo, non come valori assoluti.
- Tarare i target sul **peso reale**: se in 2–3 settimane non si muove come previsto,
  aggiustare di **±150 kcal**.

---

## 4. Principi ALLENAMENTO (contesto per coerenza)

Non c'è codice di allenamento nell'app, ma questi vincoli spiegano le scelte e vanno
rispettati se si aggiungono contenuti/sezioni allenamento.

- **Casa, manubri + elastici, tetto 20 minuti**, filosofia "low mental effort".
- **Solo 2 manubri disponibili**: non si possono mettere in superset due esercizi che
  richiedono pesi diversi (uno pesante + uno leggero) → vanno resi **sequenziali**
  (un solo cambio dischi per sessione).
- **Spalle = punto carente / priorità** → più frequenza e volume mirato (shoulder
  press seduto ~80–85°, alzate laterali ad alta frequenza).
- **Mantenimento muscolare**: basta poco volume **se** l'intensità è alta (vicino al
  cedimento, 1–2 RIR). Volume alto non serve in questa fase.
- **Gambe**: solo con elastici (squat/RDL con banda), opzionali, bassa priorità.
- **Fase attuale = mantenimento**: palestra riapre tra ~2 mesi, quarto figlio in
  arrivo. **Non** è il momento per bulk o cut aggressivi.

---

## 5. Vincoli di EDITING (regole per l'agente)

1. **Mantieni un unico file self-contained.** Niente build tool, niente nuove
   dipendenze oltre a Firebase CDN. CSS e JS restano inline.
2. **NON usare `localStorage`/`sessionStorage`** per i dati: la persistenza è Firebase.
3. **Rispetta l'invariante banner = somma pasti = TOTALE** (vedi §3). È la regola più
   importante: ogni modifica ai macro va propagata.
4. **Proteine fisse tra ON e OFF**: non ridurle nei giorni di riposo.
5. **UI in italiano**, tono semplice, mobile-first (schermo telefono).
6. **Non reintrodurre** la pseudoscienza o la regola dei 90 g di grassi (§3).
7. **Coerenza dei piani**: se aggiungi/rimuovi un piano, aggiorna l'oggetto `PLANS`,
   i bottoni `.plan-btn`, i pannelli e la griglia `.plan-toggle`
   (`grid-template-columns` = numero di piani).
8. **Sticky banner**: se cambi la struttura della barra superiore, verifica che
   `updateStickyOffset()` venga ancora richiamata nei punti giusti.
9. **Non peggiorare l'accessibilità/leggibilità** su schermo piccolo.

---

## 6. SICUREZZA

- La `apiKey` Firebase nel client **è normale** per Firebase (non è un segreto).
- La protezione dei dati dipende dalle **Security Rules del Realtime Database**:
  **NON devono essere pubbliche** (`read/write: true`). I dati sono sensibili (peso,
  BF%). Se le regole sono in "test mode", segnalarlo e suggerire di limitarle.
- `currentUser` è hardcoded (single-user): accettabile per uso personale, ma va
  ricordato che senza auth chiunque abbia l'URL + regole aperte accede ai dati.
- Non committare chiavi/segreti diversi da questa config Firebase client.

---

## 7. Tono degli agenti

- Consigli nutrizionali/allenamento **realistici e non estremi** (adulto sano, fitness
  normale). Niente diete drastiche, niente deficit aggressivi.
- Preferire **modifiche mirate** e spiegate, coerenti coi principi sopra.
- In dubbio su un numero: privilegiare **coerenza interna** (banner = pasti) e
  **prudenza** rispetto a valori "aggressivi".
