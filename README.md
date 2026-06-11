# Potenza in bicicletta

Calcolatore della **potenza necessaria per percorrere un tratto in bicicletta**
a partire da: peso del ciclista, peso della bici, coefficiente aerodinamico
(CdA), velocità media (o tempo di percorrenza), distanza, pendenza media del
percorso e vento (contrario o a favore). Oltre alla potenza restituisce
potenza specifica (W/kg), tempo,
energia meccanica, stima delle kcal bruciate e la ripartizione della potenza
fra le tre resistenze (aerodinamica, rotolamento, gravità).

Funziona anche **al contrario**: data la potenza che si intende esprimere,
calcola la velocità sostenibile e il tempo di percorrenza.

È una **webapp in un unico file** (`index.html`): nessun server, nessuna
libreria, funziona offline; layout responsive e tema chiaro/scuro automatico.
Ideale anche da telefono.

## Uso

Doppio clic su `index.html` e si apre nel browser. Per usarla da iPhone/iPad
conviene ospitarla (gratis) su GitHub Pages: attiva *Settings → Pages* sul
repository, poi in Safari *Condividi → Aggiungi alla schermata Home*: si
comporta come un'app.

## Il modello fisico

Modello stazionario standard delle resistenze al moto (Martin et al., 1998):

$$P \;=\; \frac{v}{\eta}\Big[\, m\,g\,\big(\sin\theta + C_{rr}\cos\theta\big) \;+\; \tfrac{1}{2}\,\rho\, C_d A\, (v+v_w)\,\lvert v+v_w\rvert \,\Big],
\qquad \theta = \arctan\!\frac{G\,[\%]}{100}$$

dove `m` è la massa totale (ciclista + bici), `v` la velocità, `Crr` il
coefficiente di attrito di rotolamento, `ρ` la densità dell'aria, `CdA` il
prodotto coefficiente di resistenza × area frontale, `η` l'efficienza della
trasmissione, `v_w` la velocità del vento (positiva se contrario). Le tre
componenti alla ruota sono:

- aerodinamica: `½·ρ·CdA·(v+v_w)·|v+v_w|·v` (senza vento cresce col cubo
  della velocità; negativa se il vento a favore supera la velocità di marcia);
- rotolamento: `m·g·Crr·cosθ·v`;
- gravità: `m·g·sinθ·v` (negativa in discesa).

Dalla potenza derivano tempo `t = d/v`, energia meccanica `E = P·t`
(1 Wh = 3,6 kJ) e kcal metaboliche stimate con rendimento del 24 %
(da cui la nota regola pratica kJ meccanici ≈ kcal bruciate). Se la spinta
di discesa e/o vento a favore supera le resistenze, la potenza richiesta è
zero (ruota libera) e il bilancio netto è riferito alla ruota, senza la
divisione per `η`.

La **modalità inversa** risolve numericamente `P(v) = P_target` per
bisezione; la soluzione è unica perché oltre la velocità terminale la
potenza è strettamente crescente in `v`.

## Funzionalità dell'interfaccia

- **Due modalità**: potenza da velocità/tempo, oppure velocità e tempo da
  potenza (la potenza diventa input, velocità e tempo diventano output).
- **Input doppio**: cursore per esplorare, campo di testo per la precisione
  (accetta punto o virgola; vincolato all'intervallo ma non al passo).
- **Tempo accoppiato**: il tempo di percorrenza (formato `h:mm`) è legato
  bidirezionalmente alla velocità tramite la distanza.
- **Preset CdA** per le posizioni tipiche in sella.
- **Pannello di confronto**: curva dell'output al variare di una sola
  variabile, con le altre congelate ai valori correnti e il punto operativo
  evidenziato; accanto, **tabella di sensibilità** a ±Δ e ±2Δ intorno al
  valore corrente (Δ modificabile, default = passo del cursore) con potenza,
  scostamento ΔP (o velocità e Δv in modalità inversa) e tempo di percorrenza.

## Parametri e valori di default

| Parametro              | Default     | Intervallo  | Note                                                                              |
| ---------------------- | ----------- | ----------- | --------------------------------------------------------------------------------- |
| Peso ciclista          | 72 kg       | 40–120      |                                                                                   |
| Peso bici              | 8 kg        | 5–25        |                                                                                   |
| CdA                    | 0.32 m²     | 0.15–0.60   | crono ≈ 0.23, presa bassa ≈ 0.30, mani sulle leve ≈ 0.36, posizione eretta ≈ 0.45 |
| Velocità media         | 30 km/h     | 5–60        | accoppiata al tempo                                                               |
| Distanza               | 40 km       | 1–300       | non influisce sulla potenza, solo su tempo ed energia                             |
| Pendenza media         | 0 %         | −10 … +15   |                                                                                   |
| Vento contrario        | 0 km/h      | −40 … +40   | positivo = contrario, negativo = a favore                                         |
| Crr                    | 0.005       | 0.002–0.012 | copertoncini da strada su asfalto                                                 |
| Densità aria ρ         | 1.225 kg/m³ | 1.00–1.30   | livello del mare, 15 °C; cala in quota                                            |
| Efficienza η           | 0.975       | 0.90–1.00   | trasmissione a catena                                                             |
| Potenza (mod. inversa) | 150 W       | 30–600      |                                                                                   |

## Assunzioni e limiti

- **Vento costante e parallelo al moto**: niente raffiche né vento laterale
  (il vento laterale aumenta il CdA effettivo e non è modellato).
- **Moto stazionario**: velocità costante, nessun termine di accelerazione.
- **Pendenza e velocità medie**: per la non linearità del termine in `v³`,
  la potenza calcolata a velocità media su pendenza media è un *limite
  inferiore* della potenza media su un percorso reale a profilo variabile
  (disuguaglianza di Jensen).
- **kcal indicative**: il rendimento metabolico reale varia tra ~20 e 25 %,
  quindi la stima ha un'incertezza dell'ordine del ±15 %.
- Trigonometria esatta: `θ = arctan(G/100)`; l'approssimazione `sinθ ≈ G/100`
  usata da molti calcolatori sovrastima dello 0,3 % all'8 % e dell'1,1 % al 15 %.

## Riferimenti

- J. C. Martin, D. L. Milliken, J. E. Cobb, K. L. McFadden, A. R. Coggan,
  *Validation of a Mathematical Model for Road Cycling Power*, Journal of
  Applied Biomechanics, 1998 (modello validato contro misure SRM, R² ≈ 0.97).
- S. Gribble, *An interactive model-based calculator of cycling power vs.
  speed* — implementazione di riferimento delle stesse equazioni.
