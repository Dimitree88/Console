# ADAT — Backup Architettura

> Backup dello stato di progettazione dell'interfaccia ADAT I/O standalone (8 in / 8 out, half-rack 1U).
> Progetto separato dal CONSOLE — nessuna interazione tra i due.

---

## 1. Panoramica

Interfaccia ADAT I/O standalone con:

- **8 canali ADC** (ingresso analogico → ottico ADAT)
- **8 canali DAC** (ottico ADAT → uscita analogica)
- **I/O analogico**: TRS bilanciati
- **I/O digitale**: connettori ottici ADAT (TOSLINK)
- **Form factor**: enclosure half-rack 1U
- **Configurazione**: completamente cablata (no MCU, no firmware)
- **Clocking**: solo slave (nessuna generazione clock interna, nessun wordclock I/O)

---

## 2. Chip principali

| Funzione | Chip | Note |
|---|---|---|
| ADAT encoder/transmitter | **AL1401A OptoGen** | Alesis Semiconductor — *NON* la variante V1401 |
| ADAT decoder/receiver | **AL1402G OptoRec** | Wavefront Semiconductor — *NON* la variante V1402 |
| ADC stereo (×4) | **PCM4222** | Texas Instruments / Burr-Brown |
| DAC stereo (×4) | **PCM1794A** | Texas Instruments / Burr-Brown |
| Op-amp dual (in-amp + I/V) | **OPA1679** | Audio dual op-amp |
| Op-amp fully-differential | **OPA1632** | FDA per drive ADC |
| Op-amp voltage follower | **OPA227** | Buffer per CM bias |

---

## 3. Stadio ADC (×8 canali, 4× PCM4222)

### Topologia per canale

```
TRS bilanciato
    │
    ▼
┌─────────────────────────┐
│  In-amp 3-opamp         │
│  (OPA1679 dual = 2      │
│   buffer di ingresso)   │
│  Rg variabile/preset    │
│  tra ingressi invert.   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  OPA1632 FDA            │
│  Guadagno fisso         │
│  Feedback RC = LPF      │
│  anti-alias             │
└────────────┬────────────┘
             │
             ▼ (5.6 Vpp diff, CM 1.95 V)
   Ingressi differenziali
        PCM4222
```

### Note di progettazione

- **In-amp di ingresso**: OPA1679 dual configurato come i due buffer di un classico in-amp a 3 op-amp. Alta impedenza di ingresso e alto CMRR.
- **Guadagno variabile**: settato da `Rg` (resistore tra i due ingressi invertenti dei buffer). Implementato con pot o resistor selector switch per canale.
- **OPA1632**: stadio finale fully-differential a guadagno fisso. La rete RC di feedback implementa anche il filtro anti-alias seguendo la topologia dell'EVM PCM4222.
- **Bias di modo comune**: il PCM4222 fornisce `VCOML`/`VCOMR` (1.95 V tipico). Questo viene bufferato tramite **OPA227** voltage follower e portato ai pin `VOCM` dell'OPA1632 per stabilire il riferimento CM dello stadio differenziale.
- **PCM4222 mode pins**: `FS0 = FS1 = 0` → Normal mode (range 8–54 kHz). Auto-adattamento alla sample rate senza switch esterni.

---

## 4. Stadio DAC (×8 canali, 4× PCM1794A)

### Topologia per canale

```
PCM1794A (uscita corrente differenziale)
    │
    ▼
┌─────────────────────────┐
│  I/V converter          │
│  OPA1679 (dual)         │
│  Topologia da datasheet │
│  PCM1794A di riferim.   │
└────────────┬────────────┘
             │
             ▼
   TRS bilanciato (uscita)
```

### Note di progettazione

- **I/V con OPA1679**: segue la topologia di riferimento del datasheet PCM1794A per il convertitore corrente-tensione e lo stadio differenziale di uscita.
- **Pin di configurazione PCM1794A**: `DEM`, `MONO`, `CHSL` tutti cablati a livelli logici fissi (no MCU).
- **MUTE**: pilotato dal segnale di errore di sync (vedi sezione 6).

---

## 5. Clocking

### Strategia: solo slave

- **Nessun clock interno generato**.
- **Nessun wordclock I/O esterno** (né input né output).
- Tutto il sistema si sincronizza al solo stream ADAT in ingresso.

### Recupero clock

```
OPDIGIN (stream ADAT in)
    │
    ▼
┌─────────────────────────┐
│  AL1402G in Master mode │
│  (PLL interna)          │
│                         │
│  Outputs:               │
│   • SVCO  (256 fs)      │
│   • WDCLK               │
│   • BCLK                │
└────────────┬────────────┘
             │
             ├──► PCM4222 (×4) — clock ADC
             └──► PCM1794A (×4) — clock DAC
```

### Auto-adattamento sample rate

- L'AL1402G in Master mode aggancia automaticamente la frequenza dello stream ADAT in ingresso.
- **Nessuno switch 44.1/48 kHz** richiesto: l'AL1402G locka alla rate effettiva in arrivo.
- I PCM4222 in Normal mode (FS0=FS1=0) coprono 8–54 kHz senza riconfigurazione.

### Conseguenza progettuale

- **Senza sync ADAT in ingresso → l'intero sistema si ferma.** Comportamento intenzionale: l'unità è puramente slave e dipende interamente dallo stream entrante per ogni clock.

---

## 6. Gestione sync/errore

Il pin **ERROR** dell'AL1402G (pin 21) è il **segnale centrale** di tutta la gestione sync.

- `ERROR = HIGH` → no sync / sync perso
- `ERROR = LOW` → sync valido

### Distribuzione del segnale ERROR

```
                    AL1402G ERROR (pin 21)
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
   │ MUTE input   │  │  AND gate    │  │  NOT(ERROR)  │
   │ PCM1794A ×4  │  │  blocca      │  │      │       │
   │              │  │  SVCO/BCLK   │  │      ▼       │
   │ Soft-mute    │  │  verso ADC   │  │  LED sync    │
   │ pop-free     │  │  e DAC       │  │  frontale    │
   │ sui DAC      │  │              │  │              │
   └──────────────┘  └──────────────┘  └──────────────┘
```

### Logica di muting / blocco clock

1. **DAC soft-mute**: il pin `MUTE` di tutti e 4 i PCM1794A è pilotato direttamente da ERROR. Quando ERROR=high, i DAC entrano in soft-mute (pop-free), evitando rumori in uscita durante la perdita di sync.
2. **Blocco clock**: una **AND gate generica** blocca SVCO e BCLK diretti verso PCM4222 e PCM1794A quando ERROR=high. Questo previene clock incontrollati ai convertitori, come avvisato esplicitamente dal datasheet AL1402.
3. **LED sync globale**: un singolo LED sul frontale, pilotato da `NOT(ERROR)`, indica visivamente lo stato di sync (acceso = sync OK).

### Configurazione AL1402G

| Pin | Stato | Motivo |
|---|---|---|
| MUTE (pin 22) | Tied LOW | Mute interno disattivato — il muting è gestito esternamente via ERROR |
| HOLDERR (pin 20) | Tied LOW | Auto-recovery quando il sync ritorna (no latch errore) |
| MODE | Master mode | PLL interna genera tutti i clock dallo stream OPDIGIN |

---

## 7. Configurazione fissa (no MCU)

L'unità è **completamente cablata**, nessun microcontrollore, nessun firmware, nessuna configurazione runtime. Tutti i pin di configurazione sono tied a livelli logici fissi:

| Chip | Pin | Stato |
|---|---|---|
| PCM4222 | FS0, FS1 | LOW, LOW (Normal mode, 8–54 kHz) |
| AL1401A | FMT0–3, WDCLKNEG | Cablati per formato ADAT |
| AL1402G | FMT0, FMT1 | Cablati per formato ADAT (Left Just 24 raccomandato) |
| AL1402G | MODE0, MODE1 | Master mode |
| AL1402G | MUTE, HOLDERR, LINEMODE | Tied a livelli fissi |
| PCM1794A | DEM, MONO, CHSL | Cablati a livelli logici fissi |

---

## 8. Frontale e fisico

- **Enclosure**: half-rack, 1U
- **Frontale**:
  - 1× LED globale di sync (NOT(ERROR))
  - **Nessun metering per canale**
- **Retro**:
  - 8× TRS bilanciato — ingressi analogici (verso ADC)
  - 8× TRS bilanciato — uscite analogiche (dai DAC)
  - 1× connettore ottico ADAT — ingresso (verso AL1402G)
  - 1× connettore ottico ADAT — uscita (da AL1401A)
  - Connettore alimentazione

---

## 9. Punti aperti / da definire

Aspetti **non ancora trattati** o da rivedere quando si riprende il progetto:

- [ ] Schema di alimentazione (rail analogici ±, digitali, riferimenti)
- [ ] Decoupling per ogni chip (specie PCM4222/PCM1794A vicino ai pin)
- [ ] Layout PCB e separazione piani analogico/digitale (vedi Maxim/TI app notes nel project knowledge)
- [ ] Scelta definitiva del componente per il guadagno variabile dell'in-amp (pot vs. selector switch)
- [ ] Gestione del transitorio di accensione / sequenza power-up
- [ ] Reset hardware all'AL1402G (FMT[1:0]=10) al power-up — opzionale ma raccomandato dal datasheet
- [ ] BOM finale e selezione passivi (specie feedback OPA1632 e I/V PCM1794A)
- [ ] Connettore alimentazione e tensioni di ingresso

---

## 10. Riferimenti dal project knowledge

Documenti consultati / da consultare:

- `AL1401_datasheet.pdf` — encoder ADAT (versione Alesis originale)
- `AL1402_datasheet.pdf` — decoder ADAT (versione Wavefront)
- `PCM4222_datasheet.pdf` + `PCM4222_eval_board.pdf` — ADC e topologia di riferimento ingresso
- `PCM1794A_datasheet.pdf` + `PCM1794A_ev_board.pdf` — DAC e topologia I/V di riferimento
- `Texas__Currenttovoltage_converter_circuit_for_audio_DACs.pdf` — approfondimento I/V
- `Texas_Instruments__Grounding_in_mixedsignal_systems_Part_I.pdf` + `Part_II.pdf` — grounding mixed-signal
- `Maxim_Integrated__Successful_PCB_Grounding_with_MixedSignal_Chips.pdf` — layout PCB
- `Texas_Instruments_SCAA082A__HighSpeed_Layout_Guidelines.pdf` — layout high-speed
- `Op_Amp_Applications_Handbook_CHAPTER_6_SIGNAL_AMPLIFIERS_Walt_Jung__Walt_Kester.pdf` — riferimento generale op-amp

---

*Backup generato come reference dopo eliminazione accidentale della conversazione originale.*
