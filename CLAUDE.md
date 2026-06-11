# CLAUDE.md — Contesto del progetto "Potenza in bicicletta"

Progetto nato in una conversazione con Claude (claude.ai) e portato in
locale. Questo repository contiene e fa evolvere **solo la webapp**
(`index.html`); le altre implementazioni nate nella stessa conversazione
(CLI Python, app Tkinter, eseguibile PyInstaller, workflow CI) non ne
fanno parte. Leggi questo file e `README.md` prima di modificare
qualsiasi cosa.

## Cosa fa

Calcolatore della potenza necessaria per percorrere un tratto in bicicletta
(input: peso ciclista, peso bici, CdA, velocità o tempo, distanza, pendenza
media, vento contrario/a favore, più Crr/ρ/η come parametri avanzati).
Output: potenza, W/kg, tempo,
energia, kcal, ripartizione fra resistenza aerodinamica / rotolamento /
gravità. Ha la modalità inversa (da potenza → velocità e tempo), la modalità
**percorso a tratti** (sequenza di tratti distanza+pendenza percorsi a
potenza costante, con tetto di velocità in discesa, profilo altimetrico e
totali di tempo/energia/dislivello) e un pannello di analisi di sensibilità
a una variabile. Lingua dell'interfaccia
e dei commenti: **italiano**; identificatori di codice: **inglese**.

## File e architettura

| File | Ruolo | Note |
|---|---|---|
| `index.html` | Webapp — unico sorgente del progetto | **Unico file autosufficiente**: vanilla JS, zero dipendenze, zero rete, dark mode via `prefers-color-scheme`, responsive. Si chiama `index.html` per poter essere servito così com'è da GitHub Pages. Il core fisico è delimitato dai marcatori `// CORE-BEGIN` / `// CORE-END` (estraibile e testabile fuori dal browser): `compute(p)`, `solveSpeed(p, target)` e `simulateCourse(p, segs, target, vmaxKmh)`. Stato nell'oggetto `S`, parametri nell'array `PARAMS`, righe UI generate da `buildRow()`, rendering centralizzato in `render()`, grafico SVG costruito come stringa in `drawPlot()`, tabella di sensibilità in `fillTable()`. Modalità percorso: tratti in `SEGS` (array di `{d, g}`), tetto in `VMAX`, editor generato da `buildSegRows()`, profilo altimetrico in `drawProfile()`. Deve restare un singolo file. |
| `README.md` | Documentazione utente | Contiene formula, tabella dei default/intervalli, assunzioni del modello, riferimenti (Martin et al. 1998; calcolatore Gribble). |

## Modello fisico (fonte di verità)

```
P = (v/η) · [ m·g·(sinθ + Crr·cosθ) + ½·ρ·CdA·(v+v_w)·|v+v_w| ],   θ = arctan(pendenza%/100)
```

Convenzioni implementative da preservare:

- trigonometria **esatta** (`atan`, non l'approssimazione sinθ ≈ G/100);
- vento: `v_w` in km/h, positivo = contrario; termine aerodinamico nella
  forma **con segno** `(v+v_w)·|v+v_w|` (non il quadrato), così un vento a
  favore più forte della velocità di marcia dà un termine propulsivo
  (negativo);
- a ruota libera (somma alla ruota ≤ 0, per discesa e/o vento a favore):
  potenza richiesta = 0 e il bilancio netto mostrato è **alla ruota, senza
  dividere per η** (la trasmissione non è in presa quando non si pedala);
- energia: `E[kJ] = P[W] · t[h] · 3.6`; kcal = `E/4.184/0.24` (rendimento
  metabolico 24 %);
- modalità inversa: bisezione di `P(v) = target` su v ∈ [0.1, 200] km/h con
  espansione del bracket; la soluzione è unica (oltre la velocità terminale
  P è strettamente crescente);
- modalità percorso: ogni tratto è percorso a `v = min(solveSpeed(P_target),
  v_max)`; le grandezze del tratto (potenza effettiva, energia, componenti)
  sono ricalcolate con `compute` alla velocità finale, quindi dove il tetto
  interviene la potenza effettiva scende sotto il target (fino a 0 = ruota
  libera); dislivello = Σ d·sin(atan(g/100)) sui soli tratti in salita.

## Invarianti di progetto (non violare)

1. La webapp resta **un unico file** senza dipendenze, senza build step e
   senza accesso alla rete: deve funzionare aperta con doppio clic e
   servita così com'è da GitHub Pages.
2. Ogni modifica al core fisico (zona `CORE-BEGIN`/`CORE-END`) va
   verificata con i valori di regressione qui sotto.
3. Input manuali: accettano virgola o punto; vincolati all'intervallo del
   cursore ma **non al passo**.
4. Tempo ↔ velocità accoppiati via distanza: t ∈ [d, 12·d] minuti perché
   v ∈ [5, 60] km/h; campo tempo in formato `h:mm` (o minuti). In modalità
   percorso velocità media, tempo totale e distanza compaiono negli stessi
   campi come output di sola lettura.
5. Confronto a una variabile: esclude sempre `distance`; in diretta esclude
   `power`, in inversa esclude `speed` e include `power`, in percorso
   esclude `speed` e `grade` (definiti dai tratti). Tabella di
   sensibilità a ±Δ e ±2Δ (Δ modificabile, default = passo del cursore);
   colonne: diretta `valore | P | ΔP | tempo`, inversa `valore | v | Δv |
   tempo`, percorso `valore | tempo | ΔT [min] | v media` (y del grafico =
   tempo totale [min]; fallback del menu a tendina = `power`, altrove
   `grade`). In diretta il tempo varia solo se la variabile è la velocità
   (è corretto così: t = d/v).

## Valori di regressione (tutti verificati; tolleranza ±0.1 W)

Con default: rider 72, bike 8, CdA 0.32, 30 km/h, 40 km, 0 %, vento 0,
Crr 0.005, ρ 1.225, η 0.975.

| Caso | Atteso |
|---|---|
| Default | P = 149.9 W, 2.08 W/kg, 1 h 20 min, 719 kJ, ≈716 kcal |
| 20 km/h, piano | 56.8 W |
| 8 % a 12 km/h | 234.8 W |
| −5 % a 40 km/h | P = 0 W, netto alla ruota = −123 W |
| Velocità terminale su −5 % | 48.3 km/h |
| Inversa: 250 W in piano | v = 36.48 km/h, 1:06 sui 40 km |
| Inversa: 100 W su −5 % | v = 52.66 km/h |
| Vento +10 km/h | 240.4 W (console: 240.35) |
| Vento −10 km/h | 85.2 W (85.24) |
| Vento −40 km/h | 20.6 W (20.61; la forma quadrata errata darebbe 46.5) |
| Vento −40, 20 km/h, piano | P = 0 W, netto alla ruota = −11.8 W |
| Velocità terminale in piano, vento −40 | ≈ 23.9 km/h (`solveSpeed(p, 1)` dà 24.19: punto a 1 W, poco sopra) |
| Inversa: 250 W, vento +10 | v = 30.52 km/h |
| Percorso [40 km 0 %] a 149.87 W | v media 30.00 km/h, 80 min (parità con la diretta) |
| Percorso [10 km 0 %, 8 +6 %, 8 −6 %] a 200 W, tetto 60 | T = 62.49 min (1:02), v = [33.54, 13.12, 60 col tetto], P effettive [200, 200, 194.2] W (media 199.3), +479 m, 747 kJ |
| Stesso percorso, vento +10 | T = 70.90 min |
| Jensen: [20 km +8 %, 20 −8 %] a 250 W senza tetto | v media 21.48 km/h (T 111.7 min); a 21.48 km/h su pendenza media 0 % basterebbero 66.7 W |

## Come testare

Nel browser: aprire `index.html` e usare la console DevTools (le funzioni
del core sono globali), confrontando con la tabella sopra:

```js
const base = {rider:72, bike:8, cda:0.32, speed:30, distance:40,
              grade:0, wind:0, crr:0.005, rho:1.225, eta:0.975};
compute(base).power                                // atteso: 149.87
compute(Object.assign({}, base, {wind:10})).power  // atteso: 240.35
solveSpeed(base, 250)                              // atteso: 36.48
simulateCourse(base, [{d:10,g:0},{d:8,g:6},{d:8,g:-6}], 200, 60)
// atteso: timeh*60 = 62.49, capped = 1, climb ≈ 479
```

Se è disponibile Node (non richiesto dal progetto), il core si può
testare anche senza browser:

```bash
sed -n '/CORE-BEGIN/,/CORE-END/p' index.html > /tmp/core.js
node -e "$(cat /tmp/core.js); const r=compute({rider:72,bike:8,cda:0.32,speed:30,distance:40,grade:0,wind:0,crr:0.005,rho:1.225,eta:0.975}); console.log(r.power.toFixed(2))"
# atteso: 149.87
```

La parte UI (modalità inversa, accoppiamento tempo/velocità, pannello di
confronto) si verifica a mano nel browser.

## Limiti noti del modello (documentati nel README)

Vento costante e parallelo al moto (niente raffiche; il vento laterale, che
aumenta il CdA effettivo, non è modellato); moto stazionario (niente
accelerazioni); potenza
a velocità media su pendenza media = limite inferiore rispetto a un percorso
reale a profilo variabile (Jensen, termine v³); kcal ±15 % circa.

## Sviluppi discussi e non ancora implementati

- import GPX nella modalità percorso (ricampionamento e smoothing della
  quota) per generare i tratti automaticamente da una traccia reale.
