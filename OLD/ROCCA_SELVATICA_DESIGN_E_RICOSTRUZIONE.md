# Rocca Selvatica — design completo e guida alla ricostruzione

Versione di riferimento: 28 agosto 2026, dopo gli interventi su notte, pioggia, prestazioni, cursore, dock e castello centrale.

Questo documento serve a ricostruire il gioco da zero con un altro agente LLM, anche senza la cronologia della conversazione. Descrive sia l'esperienza richiesta sia i valori e le scelte dell'implementazione di riferimento. Non è soltanto una lista di idee.

File di riferimento, se disponibile: `rocca-selvatica.html`, singolo HTML autonomo. Impronta SHA-256: `e7adf95af660cd671fe67e7251fe179b5c383f0465fdd5cd4a24254f96bd6b45`.

## Come leggere il documento

- **Requisito:** comportamento da preservare nella nuova versione.
- **Valore di riferimento:** numero effettivamente usato nell'HTML attuale; costituisce una base riproducibile, non un divieto di migliorare il bilanciamento.
- **Limite attuale:** comportamento semplificato o non implementato; non va descritto come una funzionalità già esistente.
- Le distanze sono in **unità del mondo**, i tempi in **secondi di simulazione**, salvo indicazione diversa. La risoluzione del canvas è distinta dai pixel CSS dello schermo.

## Indice

1. Visione, vincoli e priorità
2. Esperienza iniziale e ciclo di gioco
3. Mappa, geografia e castello centrale
4. Catalogo degli strumenti
5. Costruzione, riparazione e rimozione
6. Difese, proiettili e danni
7. Coccodrilli, draghi e incendi
8. Nemici e composizione delle orde
9. Arrivi, frequenza e minuto iniziale
10. Marcia, pathfinding e distruzione degli ostacoli
11. Accampamenti e comportamenti autonomi
12. Fauna
13. Direzione artistica e animazioni
14. Giorno, notte e palette
15. Meteo, pioggia, neve e acqua
16. Interfaccia, dock e cursore
17. Comandi, touch e gestione degli eventi
18. Musica, effetti sonori e vibrazione
19. Salvataggio, caricamento e reset
20. Architettura e prestazioni
21. Limiti e ambiguità risolte
22. Piano di ricostruzione
23. Collaudo e criteri di accettazione
24. Stato delle verifiche della versione di riferimento
25. Prompt pronto per un altro agente

## 1. Visione, vincoli e priorità

Un sandbox medievale in pixel art, visto dall'alto con edifici e personaggi disegnati di tre quarti. Il giocatore costruisce liberamente sulla mappa mentre eserciti ostili avanzano, attaccano e abbattono le difese. Si possono riparare e ricostruire le mura durante la battaglia, scavare fossati e liberare coccodrilli che difendono il territorio in autonomia.

Il piacere del gioco nasce dal vedere un piccolo mondo vivo reagire alle costruzioni: soldati che aggirano o sfondano muri, cannoni che mirano, arcieri che pattugliano, coccodrilli che nuotano e mordono, animali che fuggono, neve che si accumula e pioggia che increspa l'acqua.

### Vincoli imprescindibili

- Un solo file `.html`, con HTML, CSS e JavaScript incorporati. Deve funzionare offline aprendolo nel browser.
- Nessuna dipendenza da CDN, font remoti, immagini esterne, account, server, installazione o procedura di build per il giocatore.
- Nessun turno e nessuna fase di costruzione che interrompa periodicamente l'assedio.
- Nessuna economia: niente monete, materiali, energia, acquisti o timer artificiali per costruire.
- Nessuna griglia visibile e nessun aggancio obbligatorio delle costruzioni a caselle.
- Nessun limite di gioco al numero di costruzioni o di coccodrilli; restano i confini della mappa, le collisioni e i limiti fisici del dispositivo.
- Selezione prevalentemente grafica in un dock compatto a schermo, non una grande sidebar piena di testo.
- Zoom con rotella, costruzione continua tenendo premuto, spostamento della visuale semplice.
- Nemici e difese devono interagire davvero. Non basta animare decorazioni senza danni, collisioni o bersagli.
- Il castello iniziale è al **centro dell'intera mappa**, non soltanto al centro della visuale.
- La notte deve essere più scura ma leggibile, attraverso una vera palette: niente velo opaco uniforme, foschia blu applicata a tutto o flare diffusi.
- Fluidità prioritaria: non introdurre costose elaborazioni dell'intero schermo a ogni fotogramma.

### Tono

Fantasia medievale giocosa, artigianale e leggibile. Cavalieri, cavalli, pietra, legno, bandiere, tende e bombarde stilizzate. La presenza di coccodrilli è voluta anche se non storicamente realistica. Niente violenza realistica: i nemici sconfitti spariscono in piccoli pixel e sbuffi.

Il riferimento verbale originale era «la grafica pixellosa del draghetto». Non è disponibile una specifica visiva di quel gioco: non inventare di averne visto gli asset. La descrizione artistica e la palette qui sotto sono il riferimento concreto.

## 2. Esperienza iniziale e ciclo di gioco

Aprendo una partita nuova si vede subito un regno già arredato: castello, mura, sei torri, fossato completo, ponte, cannoni, catapulta, qualche arciere e quattro coccodrilli. Non si parte da un terreno vuoto.

Per i primi **60 secondi di simulazione non appare alcun nemico**, comprese le rare pattuglie interne. Il conto alla rovescia è visibile. Il giocatore può osservare, zoomare e costruire liberamente.

Dopo il minuto iniziale arrivano orde dai veri bordi del mondo. Il ciclo è continuo:

1. Scegliere un oggetto nel dock.
2. Cliccare o dipingere sulla mappa tenendo premuto.
3. Osservare difese e nemici agire autonomamente.
4. Ripassare sugli oggetti danneggiati per ripararli, o ricostruire quelli distrutti.
5. Ampliare castelli, fossati e altri punti difensivi.

La caduta del mastio **non conclude la partita**. Compare un messaggio, ma si può continuare a costruire. Se non restano bersagli, i nemici possono tornare ai campi; quando ricompaiono edifici da attaccare ripartono.

Non ci sono vittoria finale, livelli obbligatori, avatar controllato direttamente, missioni o schermata di game over bloccante.

## 3. Mappa, geografia e castello centrale

### Coordinate

| Parametro | Riferimento |
|---|---:|
| Larghezza mondo | 5200 |
| Altezza mondo | 3800 |
| Centro `HOME` | `(2600, 1900)` |
| Mastio iniziale, punto di appoggio | `(2600, 1883)` |
| Centro iniziale camera | `(2600, 1894)` |
| Scala delle bitmap del terreno | 0,5 |
| Risoluzione delle bitmap di mappa | 2600 × 1900 |

Il mastio usa un appoggio 17 unità più in alto del centro del castello; è una convenzione del disegno, non uno spostamento del castello fuori dal centro della mappa.

La mappa precedente era 2600 × 1900: quella attuale ha quattro volte la superficie. Non riportarla alla dimensione precedente.

### Struttura del castello preassemblato

Rispetto al punto di posa `(cx, cy)`:

- Mastio: `(cx, cy - 17)`.
- Mura: rettangolo con semilarghezza 153 e semialtezza 101.
- Passo dei segmenti: 17. Sui lati orizzontali si parte da `-153 + 17` e si procede finché si resta sotto `153`.
- Sul lato sud lasciare il varco centrale, omettendo i segmenti con `abs(offsetX) <= 23`.
- Torri ai quattro angoli e altre due ai lati dell'ingresso, in `(cx ± 29, cy + 101)`.
- Cannoni in `(cx - 106, cy + 48)` e `(cx + 105, cy + 48)`.
- Catapulta in `(cx + 67, cy - 37)`.
- Fossato ad anello rettangolare arrotondato: semiassi esterni di riferimento 210 e 148, raccordi di raggio 39; punti d'acqua sovrapposti di raggio 22.
- Ponte a sud, disegnato come tavole e parapetti. È associato a ciascun mastio vivo, non a coordinate globali fisse.
- Area attraversabile del ponte: `abs(x - keep.x) < 17` e `keep.y + 123 < y < keep.y + 202`.
- Coccodrilli iniziali: offset `(-198,-65)`, `(190,76)`, `(-90,148)`, `(105,-148)`.

Una partita nuova contiene **63 costruzioni**: 53 muri, 6 torri, 1 mastio, 2 cannoni e 1 catapulta. Contiene **22 arcieri**: 10 inizialmente assegnati a muri e 12 alle torri. Non deve esserci un arciere su ogni segmento.

### Paesaggio

Prati a più toni, piccoli ciuffi, fiori puntiformi, sentieri irregolari, selve dense e rade, pini, alberi tondeggianti, rocce, cespugli, terrazze rocciose, campi coltivati, qualche casa, pozzo, fieno, carro e cartelli. Le strade e gli elementi vicini alla rocca devono seguire il nuovo centro.

Il terreno di riferimento usa un generatore deterministico con seed `4119`; la neve sul terreno usa `617`, le increspature `918`, la fauna `55103`. La variante del generatore è quella comunemente chiamata Mulberry32: il contatore viene incrementato di `0x6D2B79F5` e mescolato con operazioni intere. La forma precisa del terreno può essere ricreata diversamente, purché densità, varietà e leggibilità restino comparabili.

Quantità generate nella versione di riferimento:

- 42.000 piccole macchie erbose sovrapposte.
- 65 macchie più ampie.
- 136.000 dettagli minuti del prato e 6.800 fiorellini o punti chiari.
- 3.120 tentativi di posa di alberi/rocce, filtrati per creare boschetti e lasciare spazio alla rocca.
- Esclusione della vegetazione presso `abs(x-HOME.x) < 270 && abs(y-HOME.y) < 215`; ulteriore apertura per l'accesso sud.
- Dopo i filtri e gli accampamenti, lo snapshot attuale contiene 1.583 alberi decorativi, 282 rocce, 10 case, 3 fuochi decorativi, 28 cespugli, un pozzo, un mucchio di fieno, un carro e un cartello.
- 175.000 piccole impronte di neve in una bitmap separata: non crearle a ogni frame.

### Acque e punti d'interesse

Fiume occidentale, campionato con punti sovrapposti di raggio 25 lungo:

```text
(260,0) → (355,210) → (320,420) → (435,590) → (398,815)
→ (525,1080) → (436,1390) → (608,1900) → (770,2320)
→ (820,2680) → (1100,3150) → (1000,3800)
```

Laghi ellittici in `(3830,1180)` con raggi 170 × 105 e in `(3170,2910)` con raggi 130 × 90. Lo snapshot iniziale contiene complessivamente 673 campioni d'acqua, inclusi fiume, laghi e fossato.

Accampamenti ostili iniziali:

```text
(2960,1725), (2245,2110), (2420,1565),
(3900,1850), (3950,2950), (1250,2860)
```

Villaggi lontani attorno a `(3150,1950)`, `(4160,3260)` e `(2760,420)`. Alcune case decorative possono essere rimosse dalla pulizia dello spazio dei campi. Non richiedere un conteggio prima dei filtri uguale a quello dopo i filtri.

Per riprodurre i quattro sentieri locali del prototipo, aggiungere `(1300,950)` ai seguenti punti storici:

```text
[(250,1250),(640,1180),(930,1240),(1130,1170),(1300,1150),(1300,940)]
[(1300,1170),(1600,1135),(1810,1360),(2400,1420)]
[(1350,930),(1590,720),(1740,610),(1850,260)]
[(730,260),(900,480),(1050,690),(1250,850)]
```

Il secondo prosegue verso `(4100,2540)`, `(4460,2820)`, `(4990,3000)`; il terzo verso `(3440,660)`, `(4580,560)`. Un altro percorso attraversa `(900,1600)`, `(1160,2170)`, `(2100,2520)`, `(3060,2930)`, `(4180,3340)`.

## 4. Catalogo degli strumenti

Il dock contiene 16 strumenti nell'ordine seguente. Le icone devono essere piccoli disegni degli oggetti veri, con una breve etichetta e un'indicazione della selezione.

| ID interno | Etichetta | Tasto | Passo del pennello | HP | Raggio fisico |
|---|---|---|---:|---:|---:|
| `wall` | Muro | 1 | 17 | 260 | 10 |
| `tower` | Torre | 2 | 34 | 540 | 16 |
| `water` | Fossato | 3 | 12 | — | campione 21 |
| `croc` | Coccodrillo | 4 | 21* | 240 | 10 |
| `dragon` | Drago | — | temporale: 0,65 s | 420 | 15 |
| `cannon` | Cannone | 5 | 30 | 220 | 11 |
| `catapult` | Catapulta | 6 | 35 | 240 | 15 |
| `mortar` | Bombarda | 7 | 32 | 250 | 12 |
| `bomb` | Miccia | 8 | 18 | 1 | 6 |
| `castle` | Castello | 9 | 430 | insieme di edifici | insieme di sagome |
| `tree` | Albero | — | 25 | 150 | 9 |
| `rock` | Roccia | — | 22 | 360 | 12 |
| `fire` | Falò | — | 24 | 90 | 5 |
| `torch` | Fiaccola | — | 18 | 55 | 3 |
| `camp` | Tende | — | 65 | 300 | 20 |
| `erase` | Rimuovi | E | 10 | — | pennello variabile |

Il mastio (`keep`) non è un pulsante separato: viene creato dal castello e ha 2100 HP, raggio 29. L'asterisco sul coccodrillo indica che il rilascio continuo è principalmente temporale: uno ogni 0,30 secondi, anche tenendo il mouse fermo.

Usare **Coccodrillo**, non «cocco». Le micce sono bombe con una miccia riconoscibile, non oggetti vegetali o fili decorativi senza effetto.

## 5. Costruzione, riparazione e rimozione

### Posa libera

Le coordinate di gioco sono continue. Il disegno può arrotondare al pixel per lo stile grafico, ma la costruzione non si aggancia alla griglia del pathfinding.

Al primo `pointerdown` valido si posa subito. Durante il trascinamento si interpolano nuovi oggetti tra l'ultima posa e il puntatore, usando il passo dello strumento. Il riferimento usa un'attesa di 0,055 s tra i campionamenti normali; quando il mouse resta vicino all'ultima posa riprova ogni 0,15 s. Salti di puntatore superiori a 400 unità non producono una lunghissima striscia involontaria.

Rilasciare il pulsante, perdere il focus o annullare un puntatore deve fermare il pennello. Nessuna costruzione può continuare «da sola» dopo il rilascio.

### Collisioni e riparazione

- Margine generale di posa: almeno 18 unità dal bordo della mappa.
- Per oggetti dello stesso tipo, una posa entro `1,5 × raggio` evita il duplicato. Se l'oggetto è danneggiato lo riporta subito a HP massimi.
- Per tipi diversi, evitare sovrapposizioni usando la distanza tra i centri e i raggi degli ingombri.
- Una torre posata entro 22 unità da un muro può sostituirlo.
- Le micce possono essere collocate vicino o dentro altre difese: non applicare loro gli stessi divieti di sovrapposizione degli edifici.
- La posa può rimuovere decorazioni naturali vicine per fare spazio.
- Un muro distrutto deve essere ricreato con un nuovo oggetto; non basta una texture di «riparazione».
- Ogni modifica che cambia gli ostacoli deve invalidare la navigazione.

### Castelli completi

Il castello preassemblato è una scorciatoia del pennello, non un'immagine unica con collisione rettangolare fittizia. Deve creare le singole mura, torri, armi, acqua, ponte e coccodrilli, che continuano a funzionare autonomamente.

Margini del punto di posa: X almeno 245 dai bordi; Y almeno 190 dal bordo nord e 200 dal bordo sud. Un mastio già presente entro 370 unità impedisce castelli quasi sovrapposti. Se si clicca entro 55 unità da un mastio danneggiato con lo strumento Castello, si ripara quel mastio. Non viene automaticamente ricostruita tutta la cinta distrutta.

### Gomma

Rimuove costruzioni, coccodrilli, campioni d'acqua e decorazioni raggiunte dal pennello. Riferimenti: oggetti entro `raggio + 11`, coccodrilli entro 22, centri dei campioni d'acqua entro 25, decorazioni entro 18. Ricostruire gli indici spaziali dell'acqua quando cambia l'elenco.

## 6. Difese, proiettili e danni

Le armi scelgono automaticamente un nemico vicino, si orientano, sparano e si ricaricano. Non richiedono che il giocatore prema un tasto di fuoco. I proiettili devono esistere nella simulazione e arrivare al bersaglio dopo un tempo di volo.

| Difesa | Portata | Danno | Ricarica | Velocità colpo | Raggio area | Altezza arco |
|---|---:|---:|---:|---:|---:|---:|
| Arciere su muro | 137 | 10 | casuale 5,8–8 s | 150 | 0 | 10 |
| Arciere su torre | 205 | 17 | casuale 2,3–3,2 s | 170 | 0 | 14 |
| Cannone | 235 | 66 | 4,4 s | 170 | 26 | 10 |
| Catapulta | 330 | 100 | 6,8 s | 115 | 39 | 70 |
| Bombarda | 280 | 95 | 5,8 s | 104 | 54 | 86 |

L'altezza è una componente visiva della parabola, non un asse Z completo del mondo. Un modello riproducibile è `y = lerp(sy,ty,p) - sin(p*PI)*arco`, con `p` tra 0 e 1. Tempo minimo di volo: 0,22 s. Frecce da muro a quota visuale -22, da torre -38, armi a -12.

I colpi difensivi ad area danneggiano i nemici, non il proprio castello. Il danno decresce dal centro verso il bordo, fino a circa metà del valore nominale. I colpi semplici verificano se il bersaglio è ancora vicino al punto d'impatto: tolleranza 27; in alternativa cercano un altro nemico entro 13. Il bersaglio non viene inseguito magicamente durante il volo.

Le esplosioni hanno piccoli pixel di fuoco, fumo e, se serve, spruzzi d'acqua. Non aggiungere anelli espansivi luminosi, flash a schermo intero o grandi gradienti radiali.

### Arcieri e pattuglia

Una torre nasce con due arcieri. Solo alcuni muri ne hanno uno. Se serve riprodurre la distribuzione deterministica attuale:

```js
const staffed = ((Math.round(x)*31 + Math.round(y)*17)%5 + 5)%5 === 0;
```

Gli arcieri non sono dipinti nella texture del muro: hanno posizione, orientamento, bersaglio e cooldown. Possono scegliere muri/torri adiacenti entro 28 unità e camminare verso di essi a circa 5 unità/s. Da fermi si spostano leggermente sul tratto. Se la struttura su cui si trovano viene distrutta, vengono rimossi. Le barre degli HP compaiono quando gli oggetti sono danneggiati.

### Micce

Bombe piazzate sul terreno. Un nemico entro 37 attiva una miccia di 1,1 s. Esplosione con raggio 76 e danno nominale 160. Bombe vicine entro `raggio esplosione + 24` si attivano con micce abbreviate casuali di 0,12–0,40 s. La reazione a catena deve essere temporizzata, non una ricorsione incontrollata nello stesso frame.

L'esplosione rivela le ombre per 4 s. Una miccia accesa emette piccoli pixel caldi. Le bombe si consumano quando esplodono.

## 7. Coccodrilli, draghi e incendi

Ogni coccodrillo conserva un punto di origine `homeX/homeY` e sa se è stato posato in acqua (`waterHome`). La posa introduce un piccolo scarto casuale, massimo circa ±7 unità.

| Comportamento | Valore di riferimento |
|---|---:|
| HP | 240 |
| Rigenerazione | 1,4 HP/s |
| Velocità tranquilla | 7,5 |
| Velocità di inseguimento | 27 |
| Ricerca nemici | raggio 83 |
| Limite del bersaglio rispetto all'origine | 108 |
| Distanza per mordere | sotto 19 |
| Danno morso | 110 |
| Attesa fra morsi | 1,6 s |
| Nuove mete casuali | ogni 1,5–4,5 s |
| Raggio delle mete tranquille | 12–66 dall'origine |

Un coccodrillo nel fossato sceglie mete in acqua e non abbandona il fossato per inseguire soldati lontani sulla terra. Può aggredire una preda molto vicina alla riva. Un coccodrillo posato sulla terra gira nella zona scelta.

Animare zampe, coda, orientamento e apertura della bocca. Il morso può mostrare un piccolo «GNAM!» per circa 1,1 s. I nemici possono contrattaccarlo; il coccodrillo può morire. Il contatto con un bersaglio rivela un nemico ombra.

Non devono inseguire all'infinito verso l'altro lato della mappa. La fauna neutrale si spaventa vicino a loro, ma non è il loro normale bersaglio da divorare.

### Draghi posabili

Il nuovo strumento **Drago** si trova dopo Coccodrillo nel dock grafico. I tasti 1–9 restano invariati. Il drago ha icona, anteprima e didascalia del cursore. Tenendo premuto si crea un drago ogni **0,65 secondi**, senza limiti di quantità. Il nuovo livello non contiene draghi iniziali.

Ogni posa sceglie casualmente uno dei sei colori predefiniti, che resta stabile durante la partita e nel salvataggio: **Smeraldo, Rubino, Zaffiro, Ametista, Ambra, Ghiaccio**. Tutti sputano fuoco, incluso Ghiaccio: il colore non cambia le statistiche.

- 420 HP, raggio fisico 15. I nemici vicini e le esplosioni ostili possono ferirlo.
- Volo basso autonomo; supera mura, acqua e rocce senza usare il pathfinding terrestre.
- Vagabondaggio a velocità 18 intorno alla posa, con mete tra 20 e 86 unità dalla posizione iniziale; nuova meta ogni 2–4,5 secondi.
- Cerca nemici entro 195 unità, purché entro 225 dalla posizione di posa. Insegue a velocità 32; non attraversa tutta la mappa.
- In assenza di nemici può incendiare alberi entro 120 dalla posa. Cerca solo in un indice spaziale, non nell'intera foresta per ogni frame.
- Soffio lungo 132 unità, a cono, per 0,85 secondi. Ricarica di 4,8–5,55 secondi. La direzione resta fissata durante il soffio; sono colpiti tutti i bersagli dentro il cono, non quelli dietro.
- Le vittime restano autonome: non diventano immobili o proiettili. La grafica delle fiamme segue il loro movimento.

### Incendi e alberi carbonizzati

L'accensione non uccide istantaneamente. I nemici perdono **9 HP al secondo per 10 secondi**. Gli alberi perdono **6 HP al secondo**, con durata di fuoco di 34 secondi: un albero posato da 150 HP si consuma in circa 25 secondi; un albero naturale da 180 HP in circa 30. Ferite precedenti possono abbreviare questi tempi.

Una nuova fiammata rinnova la durata, senza moltiplicare il danno per il numero di draghi. Le ombre vengono rivelate mentre bruciano. L'accensione interrompe la sosta al campo; la guarigione al campo è sospesa durante l'incendio. Gli alberi naturali diventano combustibili al primo incendio; non sono soltanto una decorazione visiva.

Gli alberi si scuriscono progressivamente in tre stadi e spariscono quando esauriscono gli HP. La navigazione viene invalidata e il passaggio si riapre. Riparare un albero costruito lo spegne e lo ripristina. Rimuovere un albero cancella anche il suo incendio. Il fuoco non si propaga spontaneamente tra alberi: è il soffio a incendiarli. Pioggia e neve non spengono automaticamente questo fuoco. Mura, rocce, alleati e fauna non vengono incendiati.

Fiamme a pixel arancioni e gialli, scintille piccole, niente flare, aloni radiali o vibrazioni nuove. Niente lampeggio bianco a ogni tick di danno. Sprite del drago 92×80, sei palette, quattro pose delle ali e bocca aperta/chiusa: massimo 48 varianti, più cache notturne. Gli alberi carbonizzati riutilizzano piccole texture, non una ricolorazione dello schermo. Effetto sonoro breve, nel canale SFX: intervallo minimo 1,8 secondi tra suoni dei draghi, oltre ai limiti audio globali.

### Persistenza dei draghi e del fuoco

Il formato resta v2 ed è compatibile con salvataggi senza questi campi. Salvare `dragons` con colore, HP, posizione, origine e timer, escludendo il riferimento al bersaglio. Al caricamento ricercare il bersaglio e non riprendere un soffio interrotto a metà.

Gli alberi costruiti conservano i campi di combustione in `objects`. Per la foresta salvare `burningDecor` con ID, HP, durata residua e stato carbonizzato. Gli alberi consumati restano rimossi tramite gli ID decorativi già persistenti. I nemici non sono ripristinati, come nelle versioni precedenti. Reset elimina draghi e incendi e ricrea la foresta. La gomma può rimuovere i draghi.

### Rocce e percorsi alternativi

Le rocce, comprese quelle posate dal giocatore, sono **indistruttibili per i nemici**: attacchi a contatto, dardi e bombe dei sabotatori non sottraggono HP. Il valore tecnico 360 resta nel formato dati per compatibilità. La gomma continua a rimuoverle normalmente. Non sono combustibili.

A* le tratta come ostacoli non sfondabili. Il nemico marcia diritto fino a incontrare un ostacolo, poi cerca una deviazione verso il castello. Se il mastio è completamente chiuso dalle mura, una ricerca davanti a una roccia può restituire un percorso parziale verso il punto raggiungibile più vicino al bersaglio: il soldato aggira le rocce e raggiunge una parete distruttibile. La chiave della cache distingue ostacoli sfondabili e non sfondabili. Restano i budget di 3 ricerche per aggiornamento e 1.400 nodi per ricerca.

Una barriera di rocce con un passaggio deve far cambiare strada ai nemici. Un anello di rocce completamente chiuso può rendere il castello irraggiungibile: non aprire varchi finti, non far attraversare la pietra e non continuare a colpirla inutilmente. Questa è una conseguenza intenzionale delle costruzioni libere.


## 8. Nemici e composizione delle orde

| ID | Tipo visivo | HP | Velocità in combattimento | Danno da contatto |
|---|---|---:|---:|---:|
| `knight` | Fante/cavaliere con elmo, scudo e spada | 85 | 15 | 24 |
| `horse` | Cavaliere montato, cavallo e lancia | 145 | 27 | 34 |
| `brute` | Soldato pesante, più largo e corazzato | 250 | 10 | 60 |
| `shadow` | Nemico ombra semitrasparente | 65 | 23 | 21 |
| `raider` | Predone rapido, colori cuoio | 68 | 24 | 19 |
| `crossbow` | Balestriere, arma orizzontale evidente | 92 | 14 | 19 |
| `sapper` | Guastatore con esplosivo e miccia | 92 | 18 | 14 |
| `ram` | Ariete su ruote, sagoma molto più grande | 460 | 8 | 90 |

Raggi: 6 per i fanti, 9 per il cavaliere montato, 18 per l'ariete. Attacco da contatto ogni 0,82 s; bruto ogni 1,2 s; ariete ogni 1,85 s.

La velocità lontano dal combattimento è maggiore: circa 80 per i fanti, 104 per i cavalli, 55 per gli arieti, con variazione ±4. Si passa gradualmente alla velocità lenta vicino all'obiettivo. Riferimento: interpolare con `clamp((distanza-280)/300,0,1)` quando il gruppo non è ancora in allarme. Costruzioni difensive vicine, circa 220 unità più il loro raggio, fanno scattare l'allarme. Gli spawn manuali usati nei test possono restare alla velocità base.

Distribuzione ciclica di riferimento, 16 posti:

```text
knight, raider, knight, crossbow, horse, knight, brute, raider,
sapper, knight, shadow, ram, horse, crossbow, raider, knight
```

Le varie parti dell'orda possono usare offset diversi nel ciclo, così da non creare file identiche.

### Abilità particolari

- **Balestriere:** cerca edifici entro circa 145; si ferma e lancia veri dardi ogni 3,1 s, velocità 145, arco 5. Non bersaglia bombe, alberi, rocce, fuochi o fiaccole con questo tiro speciale.
- **Guastatore:** arrivando a un ostacolo attaccabile accende una miccia di 1,7 s. Esplode, si consuma, raggio 48, potenza 205. Gli edifici prendono circa `205 × 0,8`; i coccodrilli circa `205 × 0,6`.
- **Ombra:** alpha visiva di riferimento 0,26 quando non rivelata; colori violacei. Arcieri, coccodrilli ed esplosioni possono renderla visibile. Non è invulnerabile e non serve un sistema complesso di furtività.
- **Ariete:** molta vita, lento e con colpi pesanti; ruote e trave animata, non un normale soldato ingrandito.

I nemici cercano in primo luogo il mastio vivo più vicino. Senza masti, scelgono altre costruzioni vive, escludendo le bombe. Aggiornano il bersaglio circa ogni 1–2,5 s o quando quello precedente muore.

## 9. Arrivi, frequenza e minuto iniziale

### Opzioni

| Opzione | Intervallo | Passo | Default |
|---|---:|---:|---:|
| Frequenza orde | 0–8 al minuto | 1 | 3 |
| Nemici per orda | 20–240 | 10 | 120 |
| Preparazione iniziale | fissa | — | 60 s |
| Tetto tecnico nemici simultanei | 640 | — | 640 |

Intervallo nominale tra orde: `60 / frequenza`. A zero non arrivano nuove orde né pattuglie interne, ma i nemici già presenti continuano ad agire. Le dimensioni scelte valgono per le prossime orde. Gli slider non eliminano né trasformano quelle già in campo.

Se lo spazio sotto il limite tecnico non basta per **l'intera orda**, aspettare e riprovare dopo circa 2 s. Non ridurre di nascosto l'orda selezionata e non accumulare una coda enorme di gruppi da scaricare tutti insieme.

### Origine fisica

I quattro fronti sono i veri margini della mappa: nord, est, sud e ovest. Mai il bordo della camera. Ogni unità deve percorrere realmente la distanza fino al castello.

La prima orda parte dai due fronti più vicini al bersaglio. Con il castello centrale sono nord e sud. Le successive ruotano sui quattro fronti; ogni terza orda può usare anche il lato opposto.

Per un fronte, scegliere un punto lungo il bordo vicino alla proiezione del bersaglio, con scarto circa ±260 e margini di 150 dagli angoli. Disporre i soldati in gruppi larghi 14: passo laterale 17, jitter ±3, profondità iniziale di 8–circa 42 dal confine. Non devono nascere dentro il castello.

La prima orda viene rilasciata subito al termine della preparazione. Le successive entrano in piccole raffiche di 8 soldati ogni 0,18 s, mantenendo l'impressione di un gruppo compatto. Le raffiche non sono turni: tutto il resto continua a muoversi.

### Eccezioni interne rare

Piccole pattuglie da 3–5 predoni, ombre o balestrieri possono comparire in campi lontani, non prima della quarta orda. Intervalli di controllo 135–210 s, probabilità 65% al controllo.

Il campo deve essere fuori dalla vista, a più di 460 da ogni mastio, lontano dai bordi, fuori dall'acqua e non occupato da costruzioni entro 100. La percentuale di nemici nati così deve restare molto bassa: il riferimento di collaudo è meno del 2% su una simulazione lunga con spazio disponibile.

La preparazione blocca **anche** questa eccezione. Pausa e menu impostazioni fermano il conto alla rovescia.

## 10. Marcia, pathfinding e distruzione degli ostacoli

La regola voluta è precisa: **prima avanzare dritti; soltanto quando si incontra un ostacolo cercare un percorso**. Non calcolare A* per ogni nemico già appena nato al bordo.

```text
bersaglio scelto
  → tratto davanti libero: marcia diretta
  → ostacolo davanti: cerca un percorso praticabile
      → percorso trovato: aggira e continua
      → percorso non trovato + ostacolo distruttibile: attaccalo
      → ostacolo decorativo non distruttibile: cerca una deviazione locale
  → varco aperto o percorso invalidato: rivaluta
```

Il sondaggio davanti al soldato arriva a 65 unità; nei test con spawn manuali usa 34. Tracciare il primo vero ingresso del segmento nel cerchio dell'ostacolo, non soltanto la distanza dal suo centro. Gonfiare gli ostacoli con il raggio del soldato e un margine di circa 2.

### A* interno di riferimento

- Celle da 16, mai visualizzate e mai usate per agganciare il pennello.
- Otto direzioni; costo diagonale circa 1,4142.
- Vietato tagliare diagonalmente attraverso angoli bloccati.
- Costo dell'acqua attraversabile 2,25 volte il terreno, ponte escluso.
- Tre fasce di ingombro: clearance 6, 10 e 18; campo di ostacoli gonfiato di ulteriori 4.
- Massimo tre nuove ricerche per aggiornamento dei nemici.
- Massimo 1.400 nodi espansi per ricerca.
- Euristica euclidea verso una zona d'arrivo, fattore 1,12.
- Se il bersaglio è a più di 650, trovare prima una meta locale oltre l'ostacolo, anziché cercare una rotta su tutta la mappa.
- Cache delle rotte per circa 2 s, indicizzata per clearance, cella iniziale e meta quantizzata; circa 384 voci prima dello svuotamento.
- Verificare sempre che il primo tratto verso la rotta sia realmente libero: il soldato non va teletrasportato alla prima cella.
- Saltare waypoint intermedi solo se il segmento è libero. Raggiungere un waypoint entro circa 3 unità è sufficiente.
- Invalidare rotte e campi quando si costruisce, demolisce, scava o distrugge un ostacolo.

Non equiparare «budget di ricerca esaurito in questo frame» a «nessun percorso possibile». Nel primo caso attendere brevemente, nel secondo può iniziare lo sfondamento.

Se un muro chiude completamente la strada, il nemico lo attacca a distanza di contatto fino ad aprire una breccia. Se si può aggirare, lo fa: non deve distruggere ogni oggetto incontrato senza provare la deviazione.

La separazione della folla è lieve e si applica soprattutto nelle deviazioni. Se la repulsione tra soldati spingerebbe il prossimo passo dentro un muro, provare il passo puro lungo la rotta; altrimenti fermarsi e rivalutare. Non lasciare che la separazione annulli le collisioni.

Falò, fiaccole e bombe non bloccano la navigazione come mura. Un falò o una fiaccola scelti come bersaglio devono comunque poter essere danneggiati a contatto.

## 11. Accampamenti e comportamenti autonomi

Gli accampamenti ostili comprendono due tende, bandiera, casse e fuoco. La loro grafica è distinguibile dalle difese del giocatore.

Stati dei nemici:

| Stato | Comportamento |
|---|---|
| `marching` | Avanza verso un bersaglio e attacca |
| `rallying` | Raggiunge fisicamente un campo di raccolta |
| `camping` | Sosta, si muove di poco attorno al fuoco, recupera HP |
| `returning` | Si ritira verso un campo perché ferito |

Circa il 38% dei gruppi successivi al primo può essere indirizzato a un campo compatibile con la traiettoria, senza deviazioni spropositate. La distanza totale passando dal campo non deve superare di oltre circa 200 la rotta diretta; il campo deve stare ad almeno 320 dal bersaglio.

Un gruppo arrivato entro 24 dal campo si ferma per 5–9 s. Un ferito rientrato riposa per 7–12 s. Al campo si recuperano 5 HP/s; piccoli movimenti a velocità 6 evitano pose completamente immobili.

Fanti, predoni e balestrieri sotto il 30% degli HP possono ritirarsi una volta, se un campo è entro 520. Il flag di rientro evita cicli infiniti di fuga e cura.

**Limite attuale:** i campi usati normalmente dall'AI sono predefiniti. L'autonomia riguarda raggiungerli, accamparsi, recuperare e ripartire. Non è implementata una completa AI di costruzione di nuovi accampamenti. Le Tende del giocatore sono oggetti decorativi/distruttibili, non caserme che generano alleati.

## 12. Fauna

Sei specie, per un totale iniziale di 112 animali. Lo snapshot di riferimento contiene 35 cervi, 18 conigli, 18 volpi, 18 cinghiali, 17 lupi e 6 anatre.

| Specie | Velocità normale | Raggio |
|---|---:|---:|
| Cervo | 13 | 10 |
| Cinghiale | 10 | 9 |
| Volpe | 15 | 8 |
| Coniglio | 12 | 5 |
| Lupo | 16 | 9 |
| Anatra | 8 | 5 |

Gli animali vagano, si fermano e cambiano meta. Mete entro 15–115 dall'origine locale; pause 0,3–2,6 s; aggiornamento delle decisioni circa ogni 0,25–0,45 s. La paura moltiplica la velocità per 1,85.

Si spaventano per nemici entro circa 70, coccodrilli entro 38 e scoppi vicini. Scelgono una meta di fuga nella direzione opposta, circa 90 unità più lontano. Le esplosioni possono dare 3 s di paura.

Le anatre restano in acqua; gli animali terrestri evitano acqua e costruzioni, usando piccole deviazioni angolari quando la direzione è bloccata. Se si allontanano oltre 210 dalla loro origine, possono fissare una nuova zona di passeggio.

Non sono unità difensive né nemici. Lupi e volpi non introducono automaticamente caccia fra specie, e gli animali non hanno un sistema di danno completo nella versione attuale.

## 13. Direzione artistica e animazioni

### Linguaggio visivo

Pixel art costruita con piccoli rettangoli, poligoni a gradini e poche linee. Mantenere proporzioni e silhouette coerenti; evitare grandi forme piatte senza dettagli. Il mondo è visto dall'alto, ma i fronti verticali degli edifici sono visibili. Non serve una griglia isometrica a rombi.

Il canvas di gioco usa normalmente metà della larghezza e dell'altezza CSS della finestra, poi viene ingrandito con interpolazione disabilitata. A 1440 × 900 CSS il canvas è 720 × 450. `imageSmoothingEnabled = false` e `image-rendering: pixelated` sono parte dello stile.

**Il testo dell'interfaccia non deve ereditare la bassa risoluzione del mondo.** In particolare, il nome accanto al cursore è testo DOM a risoluzione piena.

### Palette diurna di base

| Materiale | Colori utili |
|---|---|
| Prato | `#7e9b61`, `#829e63`, `#7b985f`, `#7e9a60`, `#869f64`, `#78955d`, `#89a268` |
| Pini e fogliame | `#284d3c`, `#3b6844`, `#608548`, `#345938`, `#4e7844`, `#76984e` |
| Tronchi | `#514c36`, `#7b6946` |
| Pietra, corpo e ombra | `#aaac8d`, `#616e62`, `#758370` |
| Pietra, bordi chiari | `#d6cfaa`, `#c5c3a1`, `#e1d9b3` |
| Acqua | `#316a67`, `#448c84` |
| Rive | `#657a51`, `#b2a57a` |
| Tetti | `#914837`, `#ae5a3e`, `#c7794e`, `#de9a61` |
| Stendardi alleati | `#345e70`, `#2e6376` con oro `#e5c887` |
| Coccodrilli | `#36563a`, `#426d3d`, `#739c51`, `#8cb060` |

Non basta applicare un filtro «pixelato» a icone generiche. Disegnare merli, giunti delle pietre, finestre strette, porte, trame dei tetti, mozzi delle ruote, corde delle catapulte, scudi, elmi, orecchie e code degli animali.

### Sagome e dettagli

- **Muro:** corpo circa 21 × 24, merli che salgono fino a 27 sopra l'appoggio; lato in ombra, sommità chiara, giunti alternati. Crepe quando molto danneggiato.
- **Torre:** corpo circa 33 × 38, merli fino a 47, piccola feritoia, bandiera e piano degli arcieri. Deve spiccare sopra i muri.
- **Mastio:** corpo centrale con porta, finestre, due torrette e tetti inclinati fino a circa 84 sopra l'appoggio. Stendardo blu/oro ben riconoscibile.
- **Cannone:** ruote, affusto, canna orientata verso il bersaglio e breve rinculo.
- **Catapulta:** struttura triangolare, braccio e cucchiaio; arco del masso nettamente più alto di quello di una freccia.
- **Bombarda:** bocca larga e inclinata verso l'alto, proiettile arcuato.
- **Alberi:** pini a triangoli spezzati; chiome arrotondate composte da gruppi di pixel, con varianti e qualche tono autunnale. **Niente bolle bianche sulle chiome.** La neve è un bordo sottile, non palline decorative.
- **Rocce:** lati di diversa luminosità, facce irregolari, qualche pixel di muschio.
- **Coccodrilli:** corpo orizzontale basso, coda lunga, dorso a scaglie, occhi e muso; denti visibili quando mordono.
- **Fauna:** cervi con corna, conigli con orecchie, volpi con coda chiara, cinghiali compatti, lupi distinti dai cani generici, anatre con becco e scia.

### Animazioni da mantenere

Camminata dei soldati e dei cavalli, passo degli animali, coda e zampe dei coccodrilli, morso, bandiere mosse dal vento, rinculo, braccio delle catapulte, frecce e proiettili in volo, piccoli impatti, fiamme variabili, scintille che salgono, acqua animata, neve e pioggia.

Disegnare gli oggetti ordinati per coordinata Y dell'appoggio, con priorità coerenti in caso di parità. Un soldato dietro una torre non deve sempre apparire davanti. Gli arcieri hanno una quota visuale legata al supporto.

Ombre locali molto discrete sono ammesse. Non usare un velo o una vignettatura generica sopra tutta la mappa: la precedente vignetta a schermo intero è stata rimossa.

## 14. Giorno, notte e palette

### Comportamento

Tre modalità: `auto`, `day`, `night`. Il pulsante sole/luna e il tasto N scelgono giorno o notte fissi; nelle impostazioni si riattiva l'alternanza automatica. Il meteo è indipendente.

Il ciclo automatico dura 240 s. Riferimento:

```js
clock = (clock + dt) % 240;
target = mode === 'night' ? 1 : mode === 'day' ? 0
  : clamp((Math.sin(clock / 240 * 2*Math.PI - Math.PI/2) + 0.18)*1.2, 0, 1);
night += clamp(target-night, -dt*0.3, dt*0.3);
```

La transizione manuale completa richiede circa 3,3 s. La simulazione in pausa congela anche il ciclo.

### Aspetto approvato

La notte ha prati e boschi scuri, acqua blu, pietra fredda con bordi chiari, tetti bordeaux e incavi molto scuri. È stata poi richiesta **ancora più scura mantenendo questo stesso stile**. Conservare il contrasto tra materiali e i punti di luce minuti.

Non usare un rettangolo semitrasparente sopra la scena. Non alzare tutti i neri verso un grigio blu: produce la foschia che era stata esplicitamente rifiutata. Una vera nebbia è un meteo separato, non il modo normale di ottenere la notte.

### Rampe notturne attuali

Campionare la luminanza di partenza con pesi `0,2126 R + 0,7152 G + 0,0722 B`. Le colonne corrispondono alle luminanze originali `0, 40, 70, 105, 145, 185, 225, 255`.

| Rampa | 0 | 40 | 70 | 105 | 145 | 185 | 225 | 255 |
|---|---|---|---|---|---|---|---|---|
| Pietra/neutri | `000000` | `070d18` | `0e1926` | `1c2c3c` | `32465a` | `586f86` | `95b2ca` | `c5dde9` |
| Vegetazione | `000000` | `040e15` | `071920` | `0d2c2c` | `183833` | `2f5e4f` | `629879` | `abc6a9` |
| Terra/legno | `000000` | `11101b` | `211f2b` | `36303a` | `51484f` | `786b78` | `b0a3af` | `d5d0da` |
| Rossi | `000000` | `190c1b` | `30172a` | `502239` | `79394e` | `a86075` | `d499a8` | `e6ccd6` |
| Acqua/blu | `000000` | `051020` | `082036` | `10364f` | `1d5370` | `417c9a` | `80b1cc` | `bfdbea` |
| Viola | `000000` | `0e0d1d` | `1c1830` | `322745` | `504267` | `7c6e99` | `afa4ca` | `d2cfe4` |

I colori della tabella sono esadecimali RGB, da precedere con `#`. Interpolare fra le colonne. Per scegliere la famiglia usare la tonalità HSV, con nodi:

```text
0 rosso, 10 rosso, 35 terra, 62 terra, 87 vegetazione,
140 vegetazione, 178 acqua, 230 acqua, 280 viola, 325 rosso, 360 rosso
```

Interpolare fra i nodi di tonalità; miscelare con la rampa neutra secondo `clamp((saturazione - 0,14)/0,22, 0, 1)`. In questo modo la pietra poco satura resta diversa dal prato.

Correzioni puntuali per i bordi del fogliame e per non perdere i coccodrilli contro l'acqua:

```text
4e7844→193831  76984e→294e42  9aaf62→4e7057  9bb461→587e62
3b6844→102d30  608548→1e3d36  71924c→2b5240  86a35a→486f55
9b9b4e→424f3b  bcbc63→687852  d8c47b→999c74
426d3d→193627  739c51→426748  8cb060→779968  73944b→3b6142
7e9c50→4e7249  a3b564→8eab72  8caf57→7d9e61  aac080→91b077
a0b465→759764  a4ae67→809c6e
```

Fiamme e micce mantengono i colori emissivi, senza illuminare artificialmente grandi cerchi attorno a loro:

```text
de793d f3bb61 fff0b0 e69a46 fce4a0 ffc76d ffe2a0
ffd48a fff0ad ff9c50 eaae5c f8cd78 ffd985
```

### Applicazione senza rallentamenti

Una tabella RGB con 6 bit per canale contiene `64³` valori. Si può precalcolare una volta, ma **non va applicata leggendo il canvas principale a ogni frame**.

Nella versione attuale:

- I colori delle primitive vengono convertiti e memorizzati in una piccola cache.
- Alberi e soldati hanno varianti notturne memorizzate.
- Le bitmap grandi vengono colorate per porzioni da 256 × 256 pixel della bitmap, riutilizzate durante la navigazione.
- Nei crepuscoli si interpolano colori o immagini corrispondenti di giorno/notte, non un velo a schermo intero.
- Il mix visivo usa 64 passi intermedi; a notte piena si disegna direttamente la variante notturna.
- Quando si colora uno sprite, si preserva il suo alpha. Non trasformare i pixel trasparenti in rettangoli neri.
- Il cursore e l'interfaccia vengono disegnati dopo la scena e restano leggibili.

Un'altra implementazione può usare una tecnica GPU equivalente, ma deve dimostrare sia la resa sia la fluidità. Non sostituire la palette con una semplice opacizzazione per far passare un test prestazionale.

## 15. Meteo, pioggia, neve e acqua

### Stati e controlli

`sun` = Sereno, `rain` = Pioggia, `snow` = Neve, `fog` = Nebbia. Il pulsante meteo li percorre in questo ordine. Il checkbox Meteo automatico abilita o ferma i cambi autonomi; un cambio manuale resta sempre possibile.

Primo intervallo automatico circa 63 s; successivi casuali 48–78 s. Sequenza automatica: sereno → pioggia o neve; pioggia → nebbia; neve → sereno; nebbia → sereno. Al sereno la scelta pioggia ha circa il 55% di probabilità.

### Pioggia aggiornata

La richiesta è «pioggia più bella e randomizzata, piccole gocce su vetro, ma non troppo».

- 168 scie di pioggia, con fase, profondità, velocità, lunghezza e piccolo scarto laterale diversi.
- Velocità circa 135–310 pixel del canvas/s; lunghezze nominali 3–10; spessore 0,65–1.
- Opacità circa 0,13–0,42, ulteriormente contenuta di notte.
- Vento comune lentamente variabile più una piccola variazione per goccia.
- Quando una scia ricompare in alto, la sua posizione laterale cambia; niente colonne equidistanti o righe geometriche ripetute.
- La casualità grafica è deterministica per goccia/ciclo e **non consuma il generatore casuale dell'AI**.
- Una trentina di possibili piccoli impatti, ognuno visibile solo per circa 0,22 s: pixel che rimbalzano sulla terra, brevi increspature segmentate sull'acqua.
- Non oscurare tutto con un rettangolo piovoso semitrasparente. La versione corrente ha rimosso anche quel velo uniforme.

### Gocce sul vetro

Al massimo sei descrittori; normalmente sono visibili soltanto due o tre piccole gocce. Periodo personale 8–15 s, vita 3,2–5,6 s, larghezza base circa 1,4–2,8 pixel del canvas. Un pixel chiaro, un bordo tenue, un'ombra minima e un piccolo scorrimento verticale bastano.

Le gocce sono legate allo schermo, non al terreno: spostare la camera non deve trascinarle con il castello. Comparsa e scomparsa morbide, scorrimento totale circa 3–10 pixel del canvas; talvolta una brevissima traccia. Nessuna lente gigante, distorsione estesa, sfocatura dell'intera scena o gran quantità di aloni.

Seed di riferimento: `8102` per le scie, `62031` per le gocce sul vetro. Usare funzioni stabili dipendenti dall'indice e dal ciclo temporale. A tempo fermo la pioggia non deve rigenerarsi casualmente a ogni rendering.

### Neve

118 fiocchi, diversi per velocità e profondità. Fiocchi normali da 2–3 pixel del canvas, circa un quarto da 4 × 4 con piccoli bracci laterali. Devono essere più visibili dei minuscoli punti iniziali, ma non coprire il gioco.

Accumulo `snow` in `[0, 0,92]`:

- Durante la neve: +0,015/s.
- Al sereno di giorno: -0,024/s.
- Al sereno di notte: -0,012/s.
- Con pioggia: -0,012/s.
- Con nebbia: -0,004/s.

La neve si posa su prato, merli, tetti, rocce e qualche bordo delle chiome. Il layer del terreno usa alpha circa `snow × 0,8`, mantenendo dettagli sottostanti. Sulle chiome arrotondate usare pochi bordi orizzontali chiari di circa 2 pixel di spessore: niente bolle bianche. Tornando al sereno l'accumulo deve scendere fino a zero, non sparire soltanto l'animazione dei fiocchi.

### Acqua

Il pennello aggiunge dischi sovrapposti di raggio 21, con bordo di terra e riva. Evitare campioni quasi duplicati entro 9 unità. Disegnare prima i bordi, poi il corpo dell'acqua, poi l'interno più chiaro per ottenere corsi continui.

Increspature brevi con fase diversa, oscillazione lenta e alpha variabile; alcuni dettagli verdi lungo le rive. L'acqua non deve essere una tinta piatta immobile.

I nemici la attraversano, ma a velocità moltiplicata per 0,37; cavalli e arieti per 0,28. Il ponte elimina il rallentamento. Il primo ingresso genera pochi spruzzi e aggiorna una metrica utile ai test.

### Nebbia

È un meteo esplicito, con bande molto leggere e mobili. Non confonderla con la notte. Per giudicare la palette notturna, fare sempre anche una prova con **Notte + Sereno**.

## 16. Interfaccia, dock e cursore

### Disposizione

- Mondo a tutto schermo.
- In alto a sinistra piccolo marchio, titolo «Rocca Selvatica» e sottotitolo.
- In alto a destra: meteo, giorno/notte, musica, effetti, Reset e impostazioni.
- Indicatori compatti: costruzioni infinite, nemici respinti, nemici in arrivo, stato delle orde/preparazione, orologio.
- Dock orizzontale in basso al centro, con separatori prima di Castello e Rimuovi.
- Minimap in basso a destra, con terreno, acqua, castelli, nemici, coccodrilli, campi, frecce dei fronti e rettangolo della camera.
- Suggerimenti discreti per rotella, trascinamento destro e ritorno al castello.
- Menu impostazioni/istruzioni modale, non una sidebar sempre aperta.

Palette UI: crema `#f2ecd8`, testo secondario `#afba9c`, oro `#edc879`, pannelli verde scuro `#22382fee`. Piccoli bordi e trasparenze sono ammessi **nei pannelli**, non come trattamento dell'intera scena notturna.

Dimensioni desktop di riferimento: strumenti 51 × 58 pixel CSS, icone 40 × 40, dock con 7 di padding e 3 di gap. Su schermi piccoli strumenti circa 43 × 54, icone 35 × 35 e dock scorrevole orizzontalmente. Le etichette secondarie in alto si possono nascondere; i pulsanti essenziali devono restare accessibili.

### Regione trasparente del dock: correzione obbligatoria

Il contenitore del dock comprende anche una riga di spiegazione sopra i pulsanti. Quella riga e i margini vuoti devono lasciare passare i clic alla mappa. Non basta rendere trasparente il testo se il suo genitore continua a intercettare gli eventi.

```css
#dock-wrap { pointer-events: none; }
#selection { pointer-events: none; }
#dock { pointer-events: auto; }
#tooltip, #cursor-label { pointer-events: none; }
```

Il dock visibile conserva la propria interazione, compreso lo scorrimento. Cliccare un pulsante sceglie lo strumento e non costruisce sotto al pulsante. Cliccare nell'area trasparente sopra il dock deve invece **iniziare subito** il deployment.

### Cursore

Un piccolo mirino rosso acceso, non un grande cerchio che nasconde il punto di posa. Riferimento: quattro tratti da 2 a 5 pixel del canvas dal centro, tratto esterno chiaro e tratto rosso `#ff202c`; dimensione compensata con `1/zoom` per restare costante sullo schermo.

Mostrare l'anteprima dell'oggetto reale, con buona visibilità, e una sagoma tratteggiata sottile del suo ingombro. Per il castello usare un rettangolo circa 460 × 354 nel mondo e un'anteprima tenue del mastio. Non trasformare il cursore in un grande disco pieno.

Il nome, per esempio **Bombarda**, è testo DOM a risoluzione piena:

- Font di sistema, 13 px CSS, peso 600, interlinea 1,3.
- Fondo opaco `#111e29`, testo `#fff8e9`.
- Padding 5 × 8, bordo sottile e accento rosso di 3 px a sinistra.
- Offset circa 18 px CSS a destra e sotto il puntatore.
- Limitare la posizione per mantenerlo dentro la finestra con almeno 6 px di margine.
- Dimensione del testo indipendente dallo zoom.
- Nessuna intercettazione di mouse o touch.
- Nasconderlo fuori dalla mappa, durante il pan, senza strumento o con menu aperto.
- Misurare la dimensione quando cambia il testo, non forzare un nuovo calcolo del layout a ogni frame.

## 17. Comandi, touch e gestione degli eventi

| Comando | Azione |
|---|---|
| Clic sinistro | Posa/ripara/rimuovi con lo strumento |
| Sinistro tenuto o trascinato | Pennello continuo |
| Rotella | Zoom centrato sul punto sotto il mouse |
| Destro + trascina | Sposta la visuale |
| Tasto centrale + trascina | Sposta la visuale |
| Spazio + trascina | Sposta la visuale |
| WASD / frecce | Sposta la visuale |
| 1–9 | Seleziona i primi nove strumenti |
| E | Rimuovi |
| F | Torna a un mastio vivo e allo zoom iniziale |
| N | Passa a giorno/notte fissi |
| P | Pausa/riprendi |
| Esc | Deseleziona; con menu aperto lo chiude |
| ? | Apre istruzioni/impostazioni |
| Clic minimap | Porta la camera in quel punto |

Zoom di riferimento: `zoomBase = clamp(min(canvasWidth/790, canvasHeight/575),0.34,1.1)`; intervallo da `0,25 × zoomBase` a `3,9 × zoomBase`. Rotella con fattore `exp(-deltaY × 0,0013)`. Conservare il punto del mondo sotto il cursore durante lo zoom. Velocità pan da tastiera circa `260 / zoom` unità/s.

Touch: un dito costruisce, due dita fanno pan e pinch. Ritardare di circa 120 ms l'inizio del pennello touch per riconoscere l'arrivo del secondo dito. Non lasciare oggetti involontari quando comincia un pinch. Gestire `pointerup`, `pointercancel`, perdita della cattura e perdita del focus.

Usare pointer capture sulla mappa quando inizia un gesto. La rotella della mappa non deve scorrere la pagina. `touch-action: none` sul canvas.

### Nessun menu del tasto destro

Il menu contestuale del browser è disabilitato **su tutta la pagina**, non soltanto sul canvas:

```js
document.addEventListener('contextmenu', event => event.preventDefault());
```

Deve valere anche su etichette, icone, dock, campi del menu e altri elementi DOM. Il trascinamento destro sulla mappa deve continuare a funzionare.

Le scorciatoie non devono attivarsi mentre si interagisce con uno slider o un input. Nel menu gestire Esc e la navigazione Tab tra i controlli. Perdere il focus azzera tasti premuti, trascinamenti e touch pendenti.

## 18. Musica, effetti sonori e vibrazione

### Due canali separati

Musica ed effetti hanno due pulsanti e due stati indipendenti. Non usare un unico interruttore audio. Default di riferimento: musica abilitata, effetti disabilitati. L'audio parte dopo il primo gesto dell'utente, rispettando le regole di autoplay del browser; non promettere musica già udibile senza interazione.

Il gioco non dipende da MP3 esterni. La musica è generata con Web Audio: melodia modale, flauto, corde pizzicate e basso lieve. Deve essere udibile, non solo esistere un oscillatore muto.

Melodia originale di riferimento, numeri MIDI, 0 = pausa:

```js
MELODY = [62,0,65,67,69,0,67,65,64,0,62,0,65,0,67,69,
          72,0,71,69,67,0,65,0,69,0,72,74,72,0,69,67,
          65,0,64,0,67,0,65,64,62,0,64,65,62,0,0,0];
CHORDS = [[50,57,65],[48,55,64],[53,60,69],[50,57,65],
          [55,62,71],[53,60,69],[48,55,64],[50,57,65]];
```

Passo 0,32 s. Flauto sinusoidale; corde e basso triangolari. Volume per nota circa 0,045 / 0,032 / 0,03; bus musica 0,7. Il flauto usa durate di circa 0,28 o 0,59 s; corde 0,40; basso 1,60. Massimo 14 voci musicali contemporanee. Pianificare poco in anticipo, circa 0,23 s, usando l'orologio audio.

Attacchi e decadimenti devono evitare clic. Rimuovere e disconnettere gli oscillatori terminati. Nascondere la scheda arresta la musica; tornando visibili riparte se abilitata. Pausa di gioco e menu, nella versione attuale, non spengono necessariamente la musica.

### Effetti controllati

Massimo tre voci SFX contemporanee; intervallo globale minimo 0,14 s. Cannone ed esplosione condividono una categoria «pesante» con almeno 0,8 s tra suoni. Intervalli aggiuntivi: costruzione 0,18; freccia 0,4; morso 0,35; colpo 0,45.

| Effetto | Frequenza iniziale → finale | Durata | Gain massimo |
|---|---|---:|---:|
| Costruzione | 290 → 160 Hz | 0,055 s | 0,018 |
| Freccia | 740 → 310 Hz | 0,055 s | 0,006 |
| Cannone | 96 → 31 Hz | 0,20 s | 0,036 |
| Morso | 130 → 55 Hz | 0,09 s | 0,025 |
| Colpo | 180 → 70 Hz | 0,055 s | 0,012 |
| Esplosione | 70 → 25 Hz | 0,27 s | 0,043 |

Gli eventi lontani e fuori dalla vista non devono produrre un muro continuo di suono. Ridurre il volume in funzione della distanza dalla camera. Il throttling audio non deve impedire il danno o l'animazione dell'evento.

### Vibrazione schermo

Checkbox dedicato per disabilitarla del tutto. Quando è disabilitata, ampiezza immediatamente zero e nessuna nuova scossa. Non confonderla con l'audio.

Con vibrazione abilitata: soltanto esplosioni visibili con raggio almeno 38, massimo una scossa ogni 1,6 s, niente scossa durante il pan. Ampiezza di riferimento appena 0,65 pixel del canvas, decadimento 4,8/s. Evitare vibrazioni per ogni freccia, costruzione o colpo di spada.

## 19. Salvataggio, caricamento e reset

Salvataggio locale, nessun backend. Chiave storica `rocca-selvatica-save-v1`, formato dati attuale con campo `v: 2`; chiave e versione non coincidono intenzionalmente. Riconoscere anche il formato precedente v1 se si vuole compatibilità con l'HTML di riferimento.

### Dati persistenti

Costruzioni vive, acqua, coccodrilli vivi, draghi vivi con il loro colore, incendi degli alberi, fauna, campi, tempo, nemici respinti, metriche, ID progressivo, meteo e relativa età, neve accumulata, modalità giorno/notte e fase, preferenze musica/SFX/vibrazione/meteo automatico, frequenza e numero delle orde, numero dell'ultima orda, conto alla rovescia iniziale e attesa del prossimo assalto.

Autosave circa ogni 18 s, inoltre su uscita/perdita del focus e modifiche importanti alle preferenze. Errori di `localStorage` o dati corrotti non devono impedire di giocare: usare una nuova partita e mostrare un messaggio comprensibile.

### Dati ricostruiti

Non vengono ripristinati i nemici già presenti, i proiettili e le particelle. Al caricamento l'assalto riparte dai bordi, conservando la progressione temporale e gli slider. Gli arcieri vengono ricreati sulle strutture valide; non serializzare riferimenti circolari a oggetti. Gli indici spaziali, le rotte, le bitmap e tutte le cache sono ricostruiti.

Validare tipi noti, coordinate finite e nei confini, raggi acqua sensati e numeri limitati. Non trasformare `objects: []` in un castello nuovo: un regno distrutto è un salvataggio valido.

Nel campo storico `removedDecor` l'implementazione salva in realtà gli ID negativi delle decorazioni **ancora presenti**. Il nome è fuorviante: mantenere la semantica se si importa quel formato, oppure usare un nome/versione nuovi nella ricostruzione.

### Reset

Un pulsante Reset visibile in alto e un comando equivalente nelle impostazioni. Chiedere conferma perché si sostituiscono la partita e il salvataggio. Annullare deve lasciare tutto identico.

Con conferma: pulire puntatori, tasti, drag, pinch, timer touch, nemici, particelle e dati della partita; ricreare terreno, castello, fossato, fauna e campi; conto alla rovescia 60, respinti 0, strumento Muro, visuale iniziale. Salvare il nuovo stato.

Conservare le preferenze audio, vibrazione e slider. Il meteo torna sereno; il checkbox meteo automatico conserva lo stato. Se il ciclo giorno/notte era automatico resta automatico a inizio giorno; se era fisso, il nuovo regno parte di giorno fisso. Questo dettaglio distingue preferenza di alternanza e stato attuale della scena.

### Castello centrale e vecchi salvataggi

La nuova partita usa `(2600,1900)`. Le partite salvate quando il castello era a `(1300,950)` **non vengono cancellate o traslate automaticamente**. Il caricamento porta la camera sul mastio realmente salvato. Per ottenere il nuovo livello iniziale centrato, l'utente usa Reset.

Non rompere una partita esistente solo perché è cambiata la posizione iniziale del nuovo livello.

## 20. Architettura e prestazioni

### Organizzazione suggerita nel singolo file

1. CSS e markup UI.
2. Costanti, cataloghi e configurazione.
3. Utilità geometriche, generatore casuale e indici spaziali.
4. Dati di simulazione e creazione del mondo.
5. Input e trasformazioni schermo/mondo.
6. Orde, navigazione e AI.
7. Combattimento, costruzione e fauna.
8. Disegno degli sprite, palette e cache.
9. Meteo, UI e audio.
10. Salvataggio, reset, bootstrap e ciclo principale.

Nessun framework è necessario. Canvas 2D + DOM + Web Audio sono sufficienti. La ricostruzione può organizzare diversamente il codice, ma la consegna al giocatore resta un solo HTML.

### Entità e indici

Array distinti per costruzioni, nemici, coccodrilli, draghi, arcieri, acqua, proiettili, particelle, testi flottanti, decorazioni, fauna e campi. Ogni entità rilevante ha ID stabile, coordinate, stato vivo/morto e i campi necessari al suo comportamento.

Usare hash spaziali con celle di riferimento da 64 per ricerche locali. Aggiornare gli indici mobili circa ogni 0,09 s; inserire immediatamente le nuove entità quando devono essere interrogabili nello stesso frame. Evitare ricerche di tutti contro tutti per ogni soldato.

Pulire gli oggetti morti circa ogni 0,3 s insieme all'aggiornamento HUD; particelle e proiettili scaduti vanno rimossi. Non far crescere indefinitamente array di cadaveri, suoni finiti o rotte obsolete.

### Loop

`requestAnimationFrame`, `dt` in secondi con massimo 0,05 per non fare enormi salti dopo una sospensione. Il riferimento aggiorna nell'ordine: orde, nemici, armi, arcieri, coccodrilli, draghi, combustione, fauna, proiettili, giorno/notte, pennello, meteo/accumulo, pulizie e salvataggio.

Disegno: terreno/acqua/ponte/neve → oggetti e personaggi ordinati per Y → proiettili/particelle → meteo → anteprima e mirino. Il testo DOM del cursore è separato.

Pausa, menu aperto e scheda nascosta fermano l'avanzamento della simulazione. La camera può restare esplorabile in pausa; il menu deve impedire costruzioni accidentali. Non usare il tempo reale trascorso in una scheda nascosta per generare un arretrato di orde al ritorno.

### Cache grafiche

- Terreno, acqua e neve sono bitmap di mappa a scala 0,5, non migliaia di dettagli ridisegnati ogni frame.
- Alberi/rocce: sprite circa 88 × 100, con varianti.
- Soldati: sprite circa 80 × 72, indicizzati per tipo, verso, frame di camminata, attacco, acqua e miccia; otto frame di camminata sono sufficienti.
- Le varianti notturne degli sprite sono riutilizzate.
- Per le grandi bitmap notturne usare **tile da 256 × 256 pixel della bitmap**, invisibili al giocatore. Disegnare solo le porzioni che intersecano la visuale. Questa suddivisione tecnica non è una griglia di costruzione.
- Preparare i tile della visuale iniziale e un piccolo margine. Creare gli altri solo quando servono.
- Quando cambia il fossato, invalidare solo le aree interessate nei tile già presenti.
- Applicare l'invalidazione della cache **dopo** l'aggiornamento della bitmap d'acqua. Colorare il vecchio bitmap prima del rebuild e poi dimenticare la modifica crea acqua notturna obsoleta.
- Una variazione del mix giorno/notte non deve rigenerare tutte le texture.
- La cache dei colori delle primitive può essere svuotata al cambio del passo di transizione, non a ogni singolo rettangolo.

### Regressione prestazionale da non reintrodurre

Una versione leggeva il canvas completo con `getImageData` e lo riscriveva con `putImageData` a ogni frame per cambiare la palette. L'utente ha segnalato forti scatti. La soluzione finale colora asset e piccoli tile riutilizzabili; il normale rendering **non legge i pixel del canvas principale**.

Anche disegnare ripetutamente grandi canvas colorati può costare: il passaggio a tile più piccoli ha ridotto sensibilmente il costo nel test locale. Non limitarsi a spostare il grande readback da una funzione all'altra.

Il testo DOM accanto al cursore non deve provocare misurazioni e modifiche di layout inutili a ogni frame. Aggiornare contenuto/dimensioni quando cambia lo strumento e posizione quando cambia davvero.

### Obiettivi di fluidità

Puntare a 60 fps sul desktop con una situazione normale, e rimanere utilizzabile con centinaia di nemici. Misurare separatamente simulazione, navigazione e rendering; riportare mediana e picchi, non soltanto una media favorevole.

Provare almeno: giorno, notte, pioggia notturna, neve accumulata, 500–640 nemici, molte bombe, pennello d'acqua continuo e zoom/pan. Il fatto che una schermata immobile sia fluida non basta.

Ridurre effetti decorativi, ricerche ridondanti e lavoro grafico prima di alterare le regole del gioco. Non introdurre un limite alle costruzioni o eliminare i nemici fuori dalla camera per nascondere un problema di prestazioni.

## 21. Limiti e ambiguità risolte

Questa sezione impedisce a un nuovo agente di attribuire al prototipo meccaniche che non possiede o di reintrodurre scelte già rifiutate.

### Interpretazioni consolidate

- «Sandbox attacco castello» significa costruire e difendere un regno in tempo reale, non controllare manualmente un singolo cavaliere.
- «Cannoni che sparano a caso» è stato precisato in cannoni che **mirano automaticamente ai nemici**.
- «Micce» è stato realizzato come bombe posabili, con attivazione di prossimità e reazioni a catena. Non esiste una classe separata di alleati che lancia granate a comando.
- «Nemici invisibili» è interpretato con soldati ombra semitrasparenti e temporaneamente rivelabili, non con sprite permanentemente impossibili da individuare.
- «Costruzioni senza limiti» elimina costi e tetti di gameplay; non elimina collisioni, confini o necessità di ottimizzazione.
- «Nessuna griglia» riguarda costruzione e aspetto. Una griglia interna invisibile per A* è ammessa.
- «Si accampano in autonomia» significa che arrivano, sostano, recuperano e ripartono da campi; la versione corrente non simula un esercito che progetta e costruisce un villaggio da zero.
- «Notte» non significa nebbia. Il cambiamento riguarda i colori dei materiali, con ombre scure e dettagli leggibili.

### Limiti tecnici attuali da dichiarare

- Gli alberi costruiti sono distruttibili. Anche gli alberi naturali bruciano con il soffio del drago. Le rocce sono indistruttibili per i nemici e restano rimovibili dal giocatore. Gli altri ostacoli naturali vengono aggirati.
- I campi ostili predefiniti non sono gestiti come le costruzioni del giocatore con lo stesso sistema completo di HP.
- La fauna ha vagabondaggio e paura, non un ecosistema completo o una progressione.
- I coccodrilli usano movimenti locali e controllo dell'acqua; non hanno lo stesso A* completo dei soldati per aggirare tutte le costruzioni.
- Non esiste un sistema generale di copertura/linea di tiro: frecce e colpi ad arco possono oltrepassare le mura.
- Il fossato è un insieme di campioni geometrici, non una simulazione idraulica. La pioggia non modifica realmente il livello dell'acqua.
- I ponti sono parte del castello preassemblato; non c'è uno strumento Ponte separato.
- Non ci sono lavoratori, produzione, logistica, porte apribili a comando, editor dei livelli, multiplayer, campagna, tecnologia o albero di ricerca.
- I nemici correnti non sono ripristinati esattamente dopo un caricamento.
- La persistenza delle rimozioni decorative riguarda principalmente gli oggetti con ID stabili; le decorazioni senza ID non hanno tutte la stessa precisione di ripristino.
- La costruzione continua di moltissimi oggetti può comunque consumare memoria. Nessuna promessa di quantità fisicamente infinite.

Questi limiti possono essere migliorati in una ricostruzione, ma devono restare separati dalle funzionalità obbligatorie. Non sottrarre tempo alle interazioni essenziali per aggiungere sistemi non richiesti.

### Regressioni esplicitamente da evitare

1. Sidebar testuale grande e invasiva.
2. Gioco a turni o orde che congelano la costruzione.
3. Necessità di cliccare una volta per ogni singolo muro o coccodrillo.
4. Spawn frequenti dentro la mappa o sul bordo della sola camera.
5. Primi nemici prima dei 60 s di preparazione.
6. Orde nominali da 240 che diventano gruppetti perché lo spazio disponibile è basso.
7. A* calcolato per tutte le unità anche su un campo completamente libero.
8. Soldati che passano attraverso i muri, restano bloccati su un punto o attaccano decorazioni indistruttibili per sempre.
9. Un arciere disegnato su ogni muro.
10. Coccodrilli immobili o che abbandonano il fossato per inseguire lontano.
11. Alberi con bolle bianche sopra la chioma.
12. Neve che non si scioglie tornando al sereno.
13. Notte lattiginosa, aloni e flare su ogni sorgente luminosa.
14. Gocce sul vetro troppo grandi o numerose, pioggia disposta in righe regolari.
15. Scosse e suoni per ogni minimo evento, senza possibilità di disattivarli.
16. Musica e SFX controllati da un solo pulsante, oppure musica tecnicamente attiva ma inudibile.
17. Mirino enorme che nasconde la posa, o testo del cursore sfocato dal canvas a bassa risoluzione.
18. Rettangolo trasparente sopra il dock che impedisce di iniziare a costruire.
19. Menu contestuale del browser che appare sui componenti DOM col tasto destro.
20. Castello nuovo rimasto nell'angolo nord-occidentale invece che al centro del mondo.
21. Lettura e riscrittura dell'intero canvas a ogni frame per la notte.

## 22. Piano di ricostruzione

### Fase A — fondazioni giocabili

Creare HTML offline, mondo 5200 × 3800, camera, zoom sul puntatore, pan, dock grafico, piccolo cursore e testo DOM. Posizionare il castello centrale. Implementare subito posa libera, pennello continuo, riparazione, gomma e Reset.

Prima di proseguire, verificare che clic e trascinamenti partano anche sotto la riga trasparente del dock e che il menu del tasto destro non compaia. Sono piccoli difetti che compromettono l'uso dell'intero gioco.

### Fase B — interazioni reali

Introdurre HP, danno, morte, hash spaziali, un nemico semplice, mura bloccanti, acqua rallentante, arco del proiettile e cannone autonomo. Poi arcieri mobili e coccodrilli. Un soldato deve poter raggiungere un muro, colpirlo, aprire una breccia ed entrare mentre il giocatore ricostruisce.

### Fase C — orde e navigazione

Preparazione di 60 s, slider, arrivi dai bordi e gruppi numerosi. Marcia dritta prima dell'ostacolo; A* limitato e cache dopo l'incontro. Gestire cancellazioni e nuove costruzioni durante la marcia, collisioni con gli ingombri dei cavalli/arieti e fallback allo sfondamento.

### Fase D — varietà

Otto tipi di nemici, campi, ritirata/cura/ripartenza, bombe con miccia e catene, catapulte, bombarde, castelli multipli, fauna. Controllare anche un mondo senza più masti e poi la ricostruzione.

### Fase E — resa grafica

Affinare palette diurna, materiali, silhouette e animazioni. Poi introdurre la notte con rampe di colore, asset memorizzati e tile; nessun filtro di foschia. Aggiungere acqua animata, accumulo/scioglimento della neve e pioggia variata con pochissime gocce sul vetro.

Non rimandare l'ottimizzazione della notte alla fine se si sta introducendo una lettura completa dei pixel: la scelta di rappresentazione grafica deve già evitare quel costo.

### Fase F — audio, salvataggi e collaudo

Musica originale sintetizzata, SFX indipendenti, limiti alle voci e alle scosse, preferenze persistenti, gestione autoplay, perdita di focus, schermate piccole e salvataggi corrotti.

Eseguire prove automatiche della simulazione e prove vere nel browser. Consegnare il file solo dopo aver risolto i problemi critici oppure dichiarare precisamente quali parti restano non verificate.

## 23. Collaudo e criteri di accettazione

### Regole per chi testa

Usare una nuova partita controllata, un seed noto quando serve, passi temporali di 1/60 o 0,1 s e scene piccole per isolare i comportamenti. Un test sulla geometria non deve dipendere casualmente da un albero decorativo, una pattuglia o un arciere fuori dalla scena di prova.

Distinguere chiaramente:

- **Test unitari/logici:** valori, condizioni, stato, tempi, danni e percorsi.
- **Test canvas:** pixel, immagini, sprite, palette e assenza di operazioni costose.
- **Test DOM reali:** hit testing, dimensioni, layout responsive, eventi catturati, tooltip e testo.
- **Test percettivi:** bellezza, leggibilità, suoni effettivamente udibili e sensazione di fluidità.

Un DOM simulato non dimostra che il browser lasci davvero passare un clic sotto un elemento trasparente. Una sequenza di note pianificate non dimostra da sola che l'utente senta la musica. Una media di rendering su canvas nativo non garantisce gli stessi FPS nel browser dell'utente.

### Matrice funzionale minima

| ID | Prova | Esito necessario |
|---|---|---|
| T01 | Aprire il solo HTML senza rete | Gioco utilizzabile, nessuna risorsa esterna necessaria |
| T02 | Nuova partita | Mondo 5200 × 3800, castello centrato, 63 costruzioni, 4 coccodrilli, 0 nemici |
| T03 | Attendere 59,9 s | Ancora zero nemici, comprese pattuglie interne |
| T04 | Superare 60 s | Inizia la prima orda dai bordi reali, non dentro il fossato |
| T05 | Pausa durante preparazione | Il tempo residuo non scende |
| T06 | Dipingere una linea obliqua di muri | Segmenti continui, coordinate libere, nessuna griglia visibile |
| T07 | Tenere fermo il mouse su Coccodrillo | Più coccodrilli nel tempo; il rilascio ferma subito la creazione |
| T08 | Ripassare un muro sano e poi danneggiato | Nessun duplicato; riparazione a HP pieni |
| T09 | Torre sopra un muro | Sostituzione valida e arcieri corretti |
| T10 | Costruire almeno 450 oggetti aggiuntivi | Nessun costo o limite numerico artificiale |
| T11 | Creare un secondo castello lontano | Vere mura, acqua, torri, armi, ponte e coccodrilli |
| T12 | Clic sul mastio danneggiato con Castello | Riparazione del mastio, non duplicazione dell'intero castello |
| T13 | Gomma su acqua, muro, coccodrillo e albero | Scompaiono dati, grafica e ostacoli coerenti |
| T14 | Un nemico davanti a muro chiuso | Attacca, il muro perde HP, si apre un varco |
| T15 | Muro isolato con passaggio attorno | Il nemico calcola una deviazione e non lo attraversa |
| T16 | Bersaglio lontano in campo libero | Traiettoria dritta, nessuna ricerca A* |
| T17 | Aggiungere un muro su una rotta attiva | Rotta invalidata e rivalutata, nessuna compenetrazione |
| T18 | Eliminare l'ostacolo | Il passaggio torna disponibile senza attendere un reset |
| T19 | Acqua contro terreno asciutto | Avanzamento visibilmente più lento in acqua |
| T20 | Attraversare il ponte | Nessun rallentamento da acqua sul ponte |
| T21 | Coccodrillo nel fossato per 20 s | Resta nella zona acquatica scelta e si muove |
| T22 | Nemico a pochi passi dal coccodrillo | Morso, danno, eventuale morte e animazione |
| T23 | Nemico molto lontano dal punto di posa | Il coccodrillo non parte per una caccia infinita |
| T24 | Cannone, catapulta e bombarda con bersagli | Rotazione, proiettile reale, tempo di volo, danno e ricarica |
| T25 | Due micce vicine e un nemico | Prima attivazione automatica, esplosione e seconda miccia accorciata |
| T26 | Otto tipi di nemico | Sagome distinte; balestriere, guastatore e ariete hanno effetti diversi |
| T27 | Arcieri per 15 s | Alcuni si spostano sulle mura; nessun arciere fisso su ogni segmento |
| T28 | Due slider, es. 2 orde/min da 40 | Due gruppi da 40 con cadenza nominale 30 s, salvo capacità |
| T29 | Frequenza zero | Nessun nuovo arrivo; nemici già presenti ancora attivi |
| T30 | 8/min, 240/orda, limite vicino | L'orda attende intera; niente riduzione silenziosa o valanga arretrata |
| T31 | Cambiare camera ai confini | Gli spawn restano ancorati al mondo, non alla visuale |
| T32 | Simulazione lunga delle pattuglie | Pochissime interne, lontane dalle difese e mai durante preparazione |
| T33 | Gruppo verso un campo | Ci arriva camminando, sosta e riparte |
| T34 | Fante ferito sotto il 30% vicino a campo | Rientra, recupera, riparte; niente rientri infiniti |
| T35 | Eliminare tutte le difese e ricostruire | Nessun game over bloccante; i nemici tornano ad attaccare |
| T36 | Animali vicino a nemico/esplosione | Paura e fuga; anatre in acqua, animali terrestri su terreno |
| T37 | Neve per 20–40 s, poi sereno | Accumulo visibile che torna gradualmente a zero |
| T38 | Albero tondeggiante innevato | Bordi chiari sottili, nessuna bolla bianca |
| T39 | Notte + Sereno | Scena più scura, bordi e unità leggibili, niente velo lattiginoso |
| T40 | Fuochi notturni | Fiamme e pochi pixel caldi; niente grandi flare |
| T41 | Pioggia | Scie variate e piccoli impatti, non righe geometriche fisse |
| T42 | Gocce sul vetro | Poche, minuscole, non sfocano o coprono la mappa |
| T43 | Spegnere automatismi meteo e giorno/notte | Stato fermo nel tempo; comandi manuali ancora funzionanti |
| T44 | Zoom e pan col mouse/touch | Punto sotto il mouse stabile; nessuna posa durante pan/pinch |
| T45 | Clic sulla riga trasparente sopra il dock | Inizia subito la posa sul terreno sottostante |
| T46 | Clic su un pulsante del dock | Cambia strumento senza costruire sotto la UI |
| T47 | Cursore Bombarda a ogni zoom | Nome nitido e costante, piccolo mirino, anteprima riconoscibile |
| T48 | Cursore presso i bordi dello schermo | Testo dentro la finestra, nessuna area morta di input |
| T49 | Destro su mappa, dock, testo e input | Nessun menu browser; il pan destro resta operativo |
| T50 | Musica ON / SFX OFF e viceversa | Canali realmente indipendenti e udibili quando attivi |
| T51 | Molte esplosioni e frecce | Audio contenuto; non un suono e una scossa per ogni evento |
| T52 | Vibrazione disattivata | Zero scosse, anche durante catene di bombe |
| T53 | Annullare Reset e poi confermarlo | Annullamento innocuo; conferma ricrea il livello centrale con preparazione |
| T54 | Salvataggio con neve, costruzioni e slider modificati | Ripristino coerente delle parti persistenti |
| T55 | Salvataggio senza edifici o JSON corrotto | Regno vuoto valido rispettato; file corrotto non blocca il gioco |
| T56 | Caricare vecchio castello fuori centro | Nessuna cancellazione; camera sul mastio salvato |
| T57 | Cambio scheda, blur, pointer cancel | Nessun pennello bloccato, niente recupero esplosivo di orde, audio gestito |
| T58 | Finestra 390 × 844 e desktop largo | Dock utilizzabile, opzioni raggiungibili, testo non schiacciato |
| T59 | 500–640 nemici con notte e pioggia | Misurare simulazione/rendering e picchi, senza blocchi ripetuti |
| T60 | Dipingere/rimuovere fossato di notte | Acqua aggiornata correttamente senza ricolorare l'intera mappa |

| T61 | Posare molti draghi | Sei colori casuali predefiniti, stabili nel tempo |
| T62 | Tenere premuto Drago e rilasciare | Posa ogni 0,65 s; rilascio interrompe subito |
| T63 | Drago senza nemici | Si muove vicino alla posa; può incendiare alberi vicini |
| T64 | Nemici davanti, dietro e ai lati | Il soffio colpisce soltanto dentro il cono |
| T65 | Nemico in fiamme per 4 s | Continua a muoversi e perde circa 36 HP, senza morte istantanea |
| T66 | Incendiare alberi posati e naturali | Bruciano lentamente, si scuriscono e riaprono il passaggio alla morte |
| T67 | Molti draghi sullo stesso bersaglio | Rinnovo della durata, nessuna moltiplicazione del danno continuo |
| T68 | Ombra o soldato al campo in fiamme | Ombra visibile, sosta interrotta, niente guarigione durante il fuoco |
| T69 | Riparare/rimuovere albero o rimuovere drago | Riparazione spegne; gomma non lascia incendi orfani |
| T70 | Salvataggio, caricamento, reset | Colori e incendi degli alberi persistono; reset li elimina |
| T71 | Barriera di rocce davanti a un castello chiuso | Deviazione attorno alle rocce, poi attacco alle mura |
| T72 | Colpire rocce o chiuderle ad anello | Nessun danno nemico, nessun attraversamento; gomma ancora utilizzabile |
### Test numerici della palette

Disegnare campioni di nero, ombra, prato, foglia, pietra, bordo, acqua, tetto, coccodrillo, neve e fiamma. Riferimenti per i colori iniziali: `000000`, `243b2c`, `7e9b61`, `345938`, `aaac8d`, `d6cfaa`, `448c84`, `914837`, `8cb060`, `eef0dc`, `fff0b0`.

La trasformazione deve:

- Lasciare il nero a nero.
- Tenere le ombre scure, invece di alzarle in un grigio velato.
- Separare chiaramente pietra/prato, bordo/corpo del muro e acqua/coccodrillo.
- Mantenere l'acqua più blu e il tetto più rosso del prato.
- Conservare caldi i pixel della fiamma; il campione `fff0b0` non viene attenuato.
- Non introdurre blur fra due colonne adiacenti di colore.
- Lasciare il giorno invariato e produrre una transizione graduale.

Le soglie numeriche devono aiutare il giudizio, non sostituirlo: una notte può superare un test sul contrasto e restare percettivamente troppo chiara o opaca. Confrontare screenshot della stessa scena con meteo sereno.

### Test prestazionali specifici

Nel normale rendering, intercettare temporaneamente `getImageData` **del canvas principale** e far fallire il test se viene chiamato. La lettura di un nuovo piccolo tile o sprite per costruirne la cache è ammessa; un readback completo a ogni frame no.

Dopo il preriscaldamento, ripetere decine di frame e piccoli pan su tile già presenti: i contatori di pixel ricolorati devono restare fermi. Scavando un piccolo tratto d'acqua, il numero di pixel ricolorati deve essere proporzionato alla zona modificata, non a 2600 × 1900.

La pioggia, a parità di tempo, deve disegnarsi uguale e non cambiare lo stato casuale della simulazione. Questo aiuta sia la riproducibilità sia a evitare sfarfallio casuale per frame.

### Procedura manuale breve nel browser

1. Aprire una copia nuova, cancellando soltanto il salvataggio di test o usando Reset con conferma.
2. Verificare il castello centrale sulla minimappa, non solo nella camera.
3. Prima delle orde, costruire iniziando dalla scritta sopra il dock; poi selezionare Bombarda e leggere il nome a vari zoom.
4. Provare destro sulla mappa, su un pulsante e su uno slider.
5. Passare a Notte + Sereno, poi Pioggia, e infine Neve; osservare colori, gocce e accumulo.
6. Lasciare arrivare le orde; seguire un gruppo dal bordo, un aggiramento e uno sfondamento.
7. Riparare mentre i nemici attaccano; provare una catena di micce e un coccodrillo nel fossato.
8. Separare musica/SFX, disabilitare le scosse e verificare ascoltando e guardando.
9. Provare lo scenario più affollato mentre si fa pan e si dipinge acqua.
10. Salvare, ricaricare e controllare dati, preferenze e assenza di errori console.

### Definizione di completamento

La ricostruzione è pronta quando le meccaniche critiche funzionano senza workaround manuali, i controlli restano comodi, il gioco conserva il suo aspetto anche di notte e sotto la pioggia, e le prove dichiarate sono state realmente eseguite. Un mockup, una sola schermata bella, un video precalcolato o pulsanti senza comportamento non equivalgono al gioco richiesto.

## 24. Stato delle verifiche della versione di riferimento

La versione HTML associata a questo documento ha superato **104 controlli automatici su 104** nella suite completa. Sono stati eseguiti il JavaScript reale in una VM Node, il disegno su canvas nativo e input/DOM simulati. Sono stati ispezionati visivamente anche rendering di giorno, notte, pioggia e molti nemici.

Nella simulazione di assedio a 140 s: 53 costruzioni distrutte, 178 colpi difensivi, 296 ingressi rallentati in acqua e interazioni dei coccodrilli. Questi numeri sono un campione deterministico di collaudo, **non obiettivi di bilanciamento** da riprodurre a forza.

Misure indicative del solo rendering, canvas 720 × 450 mostrato a 1440 × 900, senza simulazione nel ciclo cronometrato:

| Scena | Mediana | 95° percentile |
|---|---:|---:|
| 6 draghi e alberi in fiamme, giorno | circa 5,5 ms | circa 6,9 ms |
| Stessa scena, notte | circa 6,5 ms | circa 7,2 ms |
| Stessa scena, notte e pioggia | circa 6,9 ms | circa 7,6 ms |
| 24 draghi, 364 nemici, oltre 90 incendi, notte e pioggia | circa 19,7 ms | circa 27,1 ms |

Zero letture dei pixel del canvas principale durante queste misure. Sono misure locali su canvas nativo: **non sono un benchmark GPU del browser né una garanzia di FPS sul dispositivo dell'utente**. La verifica interattiva DOM e dell'audio percepito nel browser resta distinta; non è stata dichiarata come effettuata da questi test simulati.

Se è disponibile anche la cartella di lavoro originale, esistono uno script `work/verify.cjs` e controlli mirati con `--features`, `--revision` e `--dragon`. Il nuovo gruppo draghi/rocce aggiunge 20 test; `--dragon` esegue questi e 3 controlli audio, per un totale di 23. La suite aggiornata comprende 104 casi; `--features` ne esegue 78 e `--revision` 24. Le dipendenze del test sono del contesto di sviluppo, **non del gioco HTML**. Non presumere che quei percorsi o quel runtime esistano su un altro computer: la matrice di questa sezione permette di ricreare i test.

Quando un'altra versione cambia valori o architettura, aggiornare il documento e riportare quali test sono stati rieseguiti. Non riutilizzare automaticamente il risultato «104/104» di questa versione per un nuovo gioco.

## 25. Prompt pronto per un altro agente

Copiare il testo seguente in una nuova conversazione e allegare questo Markdown. L'HTML esistente è facoltativo come riferimento visivo o per confronto; per una ricostruzione indipendente è sufficiente partire dalla specifica.

```text
Voglio ricostruire da zero il gioco Rocca Selvatica descritto nel Markdown allegato.
Leggi tutto il documento: contiene vincoli, valori di riferimento, problemi già
corretti e criteri di collaudo. Non limitarti a un mockup o a una demo statica.

Consegna un solo file HTML autonomo e offline, con CSS, JavaScript, grafica e
audio incorporati o generati dal codice. Non richiedere server, account, CDN,
asset esterni o una procedura di installazione al giocatore.

Priorità: costruzione continua e libera senza griglia o risorse; castello già
pronto al centro di una mappa grande; 60 secondi iniziali senza alcun nemico;
orde numerose dai bordi; marcia dritta prima dell'ostacolo e pathfinding solo
quando serve; muri realmente distruttibili e riparabili durante l'attacco;
fossati rallentanti; coccodrilli autonomi locali; draghi colorati con incendi lenti su alberi e nemici; rocce indistruttibili per i nemici e deviazioni reali; difese automatiche; arcieri
soltanto su alcuni muri; otto tipi di nemici, campi e fauna.

Mantieni una pixel art curata. La notte deve essere scura ma leggibile grazie
a palette distinte dei materiali, senza un velo opaco su tutto e senza flare.
Pioggia variata, piccoli impatti, pochissime gocce discrete sul vetro; neve
grande abbastanza da vedersi, che si posa e poi si scioglie al sereno.

Interfaccia compatta con dock grafico. L'area trasparente sopra il dock lascia
passare i clic. Mirino piccolo rosso, anteprima chiara e nome dell'oggetto
nitido a risoluzione piena. Nessun menu browser al tasto destro sulla pagina.
Zoom con rotella, pan col destro, supporto touch. Musica ed effetti separati,
vibrazione completamente disattivabile, automatismi meteo e giorno/notte
indipendenti, slider per frequenza e numerosità delle orde, reset confermato.

Lavora per versioni giocabili e verifica ogni sistema. Evita lettura e
riscrittura dell'intero canvas a ogni frame per la notte: usa cache/tile,
palette per asset o una soluzione equivalente misurata. Il gioco deve restare
utilizzabile con centinaia di nemici mentre si costruisce e si sposta la camera.

Usa i valori del documento come baseline. Puoi migliorare l'organizzazione
del codice, ma segnala ogni modifica intenzionale delle regole. Non aggiungere
economia, turni, multiplayer, campagne o sistemi non richiesti a scapito del
nucleo del gioco. Non cancellare o alterare vecchi salvataggi senza necessità.

Alla fine consegna il file HTML, un riepilogo breve e i risultati reali dei
test. Distingui simulazione, rendering, browser interattivo e ascolto audio:
non dichiarare verificato ciò che non hai potuto provare. Segnala limiti e
problemi ancora presenti. La matrice di collaudo nel Markdown è il contratto
di accettazione per la ricostruzione.
```

### Materiale da portare al nuovo agente

Questo Markdown è il documento principale. Se si vuole confrontare la resa esistente, allegare anche `rocca-selvatica.html`. Per una prova indipendente, chiedere al nuovo agente di scrivere un nuovo file, senza sovrascrivere il riferimento, e confrontare i risultati con la stessa matrice di test.

Non servono la cronologia di questa conversazione, i percorsi del computer originale o un collegamento a servizi esterni per capire e ricreare il gioco.
