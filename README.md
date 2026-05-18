#  Progetto: Modellazione e Controllo in Cascata di un Servomotore DC
Modellazione e controllo in cascata della velocità di un servomotore DC a magneti permanenti mediante convertitore H bridge con modulazione PWM.
Sviluppato in ambiente MATLAB & Simulink
# Modellazione e Controllo in Cascata di un Servomotore DC con Ponte H e PWM

Questo repository contiene il progetto di modellazione dinamica, analisi e simulazione in ambiente **MATLAB & Simulink** di un sistema di controllo in cascata per la regolazione della velocità di un servomotore DC a magneti permanenti (**Maxon Motor RE 35**, Versione 24V, 90W). Lo stadio di potenza è affidato a un convertitore DC-DC a ponte H (**STMicroelectronics VNH7070AS**).

## 📌 Obiettivi del Progetto
1. **Modellazione Dinamica:** Definizione delle funzioni di trasferimento dei componenti (motore, convertitore PWM, sensore).
2. **Analisi del Sistema:** Studio della stabilità ad anello aperto nel dominio del tempo e della frequenza.
3. **Progettazione del Controllo:** Sintesi di un'architettura in cascata (Anello interno di corrente + Anello esterno di velocità) con gestione delle saturazioni.
4. **Validazione:** Verifica della reiezione ai disturbi di carico e robustezza del sistema tramite simulazione.

---

## ⚙️ Parametri del Sistema

### Convertitore H-Bridge (VNH7070AS)

| Parametro | Descrizione | Valore |
| :--- | :--- | :--- |
| $V_{in}$ | Tensione di alimentazione | 24 V |
| $f_{PWM}$ | Frequenza di commutazione | 20 KHz |
| $I_{out}$ | Corrente nominale di uscita | 15 A |

### Servomotore DC (Maxon Motor RE 35)

| Parametro | Descrizione | Valore |
| :--- | :--- | :--- |
| $V_n$ | Tensione nominale | 24 V |
| $R_a$ | Resistenza di armatura | 0.658 $\Omega$ |
| $L_a$ | Induttanza di armatura | 0.191 mH |
| $K_t$ | Costante di coppia | 0.0292 N·m/A |
| $K_e$ | Costante di f.c.e.m. | 0.0291 Nm/A |
| $J$ | Costante d'inerzia del rotore | $8.12 \cdot 10^{-6}$ kg·m² |
| $b$ | Costante d'attrito viscoso | $4.61 \cdot 10^{-6}$ N·m·s/rad |

---

## 📐 Modellazione Dinamica

La funzione di trasferimento complessiva del motore ad anello aperto (dalla tensione d'armatura $V_a$ alla velocità angolare $\omega$) unisce l'equazione elettrica e quella meccanica:

$$G_{motore}(s) = \frac{\omega(s)}{V_a(s)} = \frac{K_t}{L_a J_{eq} s^2 + (R_a J_{eq} + L_a b)s + (R_a b + K_t K_e)}$$

* **Poli del motore:** Il sistema presenta due poli reali negativi distinti ($s_1 = -197$ rad/s, $s_2 = -2780$ rad/s), risultando intrinsecamente **sovrasmorzato** e stabile.
* **Convertitore:** Modellato come guadagno statico pari a $V_{in}$ associato a un ritardo del primo ordine dovuto alla modulazione PWM.
* **Sensore di Velocità:** Encoder ottico (campionamento 1000 Hz) modellato con guadagno unitario e associato a un filtro analogico passa-basso con frequenza di taglio a 100 Hz per prevenire l'anti-aliasing.

---

## 🎛️ Architettura di Controllo (Cascata)

Il controllo è strutturato su due cicli nidificati per garantire elevate prestazioni dinamiche e protezione da sovracorrenti:

1. **Anello Interno (Corrente/Coppia):** Regolatore **PI** tarato mediante il metodo della **cancellazione polo-zero** per compensare il polo elettrico del motore ($\tau_e = L_a/R_a$). Rende la risposta di corrente estremamente rapida.
2. **Anello Esterno (Velocità):** Regolatore **PI** ($K_d = 0$ per evitare l'amplificazione del rumore ad alta frequenza) tarato manualmente per minimizzare il tempo di salita.

### ⚠️ Gestione delle Saturazioni (Anti-Windup)
Per evitare il fenomeno dell'integrator windup dovuto ai limiti fisici del duty cycle (limitato tra $[-1, +1]$) e della corrente di spunto (saturazione impostata a $\pm 10$ A), in entrambi i controllori è stata implementata una logica di **Anti-Windup Clamping**. Questo blocca l'azione integrale non appena l'attuatore va in saturazione, azzerando gli overshoot distruttivi nel rientro in zona lineare.

---

## 📊 Risultati delle Simulazioni

Le simulazioni in ambiente Simulink hanno evidenziato ottime specifiche prestazionali ad anello chiuso:
* **Tempo di salita (Rise Time):** ~7 ms
* **Sovraelongazione (Overshoot):** < 5%
* **Errore a regime:** Nullo

### Risposta al Gradino di Carico
Applicando una coppia di carico pari a $0.08\text{ Nm}$ sotto forma di disturbo a gradino (all'istante $t = 2\text{ s}$), il sistema ha dimostrato un'eccellente **capacità di reiezione del disturbo**:
* La velocità subisce una flessione transitoria minima per poi tornare al set-point nominale di $250\text{ rad/s}$ in pochissimi millisecondi.
* La corrente d'armatura assorbe un picco iniziale allo spunto, per poi stabilizzarsi a regime intorno a $2.7\text{ A}$ per vincere la coppia resistente introdotta.

---

## 💻 Requisiti di Sistema
* MATLAB (Versione R2020a o successive)
* Simulink
* Control System Toolbox

## 🚀 Come Eseguire la Simulazione
1. Clonare il repository: `git clone https://github.com`
2. Aprire MATLAB e posizionarsi nella cartella del progetto.
3. Eseguire il run del live script 
