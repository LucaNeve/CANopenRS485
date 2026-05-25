# CANopenRS485

Progetto educativo su ESP32 per esplorare i principi fondamentali
di CANopen e dei bus di campo industriali, implementati su RS485.
Si tratta di un'esercitazione pratica: l'obiettivo non è far muovere
un servo, ma capire come funziona il sistema di controllo che lo fa
muovere. Il movimento fisico è la prova che il protocollo funziona,
non il fine del progetto.

## Contesto

Nei sistemi embedded industriali come gru, bracci robotici,
dashboard navali e veicoli pesanti, i componenti non si parlano
direttamente. Comunicano attraverso un bus condiviso seguendo
protocolli precisi. CANopen è uno degli standard più diffusi
in questo mondo: definisce come i nodi si presentano sulla rete,
come scambiano dati in tempo reale e come gestiscono stati ed errori.

Questo progetto ricostruisce quei principi su hardware accessibile
con ESP32 e moduli MAX485, per capirli dall'interno.

## Architettura

Il sistema è composto da tre nodi ESP32 collegati sullo stesso
bus RS485 fisico tramite moduli MAX485.

**Master** gestisce la rete. Inizializza i nodi, monitora il loro
stato tramite heartbeat, rileva anomalie e coordina la comunicazione.
È il corrispettivo del master CANopen che gestisce il Network
Management (NMT).

**Joystick** legge due assi analogici da un joystick fisico
collegato agli ADC dell'ESP32 e pubblica i valori sul bus come
PDO (Process Data Object) a intervalli regolari. Non sa chi
riceverà i dati, pubblica e basta.

**Attuatore** riceve i PDO del Joystick, interpreta i valori
degli assi e li traduce in segnali PWM per un servo motore.
Non sa da dove arrivano i dati, li legge dal bus e reagisce.

I tre nodi non si parlano direttamente. Ogni interazione
avviene attraverso messaggi strutturati sul bus condiviso,
esattamente come in un sistema industriale reale.

## Bus fisico

A differenza di UART punto-punto, RS485 è un bus differenziale
multi-drop: tutti i nodi sono collegati fisicamente agli stessi
due fili (A e B). Quando un nodo trasmette, tutti gli altri
ricevono il segnale. Ogni nodo decide autonomamente se il
messaggio lo riguarda.

Il bus è half-duplex, quindi un solo nodo trasmette alla volta.
La direzione viene gestita via software tramite il pin DIR
del modulo MAX485 (DE e RE collegati insieme): HIGH durante
la trasmissione, LOW durante la ricezione.

Le resistenze di terminazione da 120Ω alle estremità del bus
evitano riflessioni del segnale che causerebbero corruzione
dei dati.

## Il protocollo

Il protocollo è ispirato a CANopen e ne ricostruisce i principi
fondamentali.

**NMT (Network Management)** gestisce il ciclo di vita dei nodi:
inizializzazione, avvio, stop, reset. Ogni nodo risponde agli
stati NMT e li segnala periodicamente tramite heartbeat.

**PDO (Process Data Object)** sono messaggi ciclici per lo scambio
di dati in tempo reale. Il Joystick pubblica PDO con i valori
degli assi e l'Attuatore li consuma. Latenza bassa, overhead minimo.

**SDO (Service Data Object)** sono messaggi per la configurazione
dei parametri. Il Master può leggere e scrivere parametri
nell'Object Dictionary di ogni nodo, ad esempio la sensibilità
degli assi o i limiti di movimento del servo.

**Object Dictionary** è il dizionario di oggetti che ogni nodo
espone per descrivere le sue capacità e i suoi parametri.
È il cuore di CANopen: standardizza come i nodi si descrivono
alla rete.

**Heartbeat** è il meccanismo con cui ogni nodo trasmette
periodicamente il proprio stato. Il Master rileva i nodi
silenziosi e gestisce i timeout.

## Struttura del progetto

    CANopenRS485/
    ├── platformio.ini
    ├── src/
    │   ├── main.cpp              <- dispatcher per environment
    │   ├── master/
    │   ├── joystick/
    │   └── actuator/
    └── lib/
        └── canopenrs485_common/  <- protocollo condiviso

## Hardware necessario

- 3x ESP32
- 3x modulo MAX485 (breakout board)
- 1x joystick analogico a due assi
- 1x servo motore
- 2x resistenza 120Ω (terminazione bus)
- cavi, breadboard, alimentazione

## Stato del progetto

- [ ] Definizione protocollo e struttura frame
- [ ] Implementazione Object Dictionary
- [ ] NMT: gestione stati e heartbeat
- [ ] PDO: trasmissione valori joystick
- [ ] SDO: configurazione parametri
- [ ] Attuatore: controllo servo da PDO
- [ ] Test su hardware fisico

## Relazione con ProtoCAN

Questo progetto nasce dopo [ProtoCAN](https://github.com/LucaNeve/ProtoCAN),
una prima esercitazione sui principi base della comunicazione embedded
come frame binari, CRC e parser a macchina a stati. CANopenRS485
porta quei principi su un bus fisico reale e introduce un modello
applicativo ispirato agli standard industriali.