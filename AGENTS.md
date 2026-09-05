# AGENTS.md

## Progetto e fonte di verità

Questo repository contiene **Rocca Selvatica**, un sandbox medievale in pixel art eseguito direttamente nel browser.

Prima di modificare meccaniche, interfaccia, grafica, audio, salvataggi o prestazioni, leggere integralmente `ROCCA_SELVATICA_DESIGN_E_RICOSTRUZIONE.md`. Quel documento è la specifica funzionale e il contratto di accettazione; i valori indicati come “di riferimento” sono la baseline, mentre i “limiti attuali” non vanno presentati come funzioni implementate.

In caso di conflitto:

1. prevale la richiesta esplicita dell'utente;
2. poi prevale il documento di design;
3. infine prevale il comportamento del codice esistente.

Segnalare nel riepilogo finale ogni deviazione intenzionale dalla specifica.

## Struttura del repository

- `index.html`: versione attiva del gioco; contiene HTML, CSS e JavaScript.
- `manifest.webmanifest`, `service-worker.js`, `favicon.svg`, `icons/`: supporto PWA e installabilità.
- `ROCCA_SELVATICA_DESIGN_E_RICOSTRUZIONE.md`: specifica completa, valori di riferimento e matrice di collaudo.
- `OLD/rocca-selvatica-v1.html` e `OLD/rocca-selvatica-v2.html`: snapshot storici di riferimento. Non modificarli salvo richiesta esplicita.

## Vincoli da preservare

- Il gioco deve essere autonomo e funzionare offline, senza CDN, font remoti, immagini esterne, account, server o build obbligatoria per il giocatore.
- Conservare la natura sandbox: costruzione libera e continua, nessuna economia, nessun turno, nessuna griglia visibile e nessun limite artificiale agli oggetti.
- Il mondo misura 5200 × 3800 e il castello iniziale resta al centro dell'intera mappa.
- Una nuova partita concede 60 secondi di preparazione senza nemici; in seguito le orde arrivano dai bordi reali del mondo.
- Mura, acqua, armi, nemici, coccodrilli, draghi, incendi, rocce e fauna devono avere interazioni reali, non soltanto animazioni decorative.
- La caduta del mastio non deve bloccare la partita.
- Preservare compatibilità dei salvataggi. Non cambiare la chiave o lo schema persistente senza una migrazione sicura e non sostituire un regno vuoto valido con il livello iniziale.
- Mantenere input mouse e touch: posa immediata e continua, zoom sul puntatore, pan, pinch, rilascio/cancel/blur sicuri e menu contestuale disabilitato su tutta la pagina.
- L'area trasparente sopra il dock deve lasciare passare i clic; i controlli del dock non devono costruire sulla mappa sottostante.
- Musica, effetti sonori e vibrazione restano controlli indipendenti.
- La notte usa palette/cache/tile o una tecnica equivalente. Non leggere e riscrivere l'intero canvas principale a ogni frame.
- La resa resta pixel art leggibile: niente velo notturno uniforme, grandi flare, blur globale o effetti che coprano la scena.

## Metodo di modifica

- Preferire cambi piccoli e mirati in `index.html`; non effettuare riscritture estese senza necessità.
- Prima di cambiare una costante o una regola, cercarne tutti gli usi e verificare la sezione corrispondente della specifica.
- Conservare gli indici spaziali e invalidare navigazione/cache quando cambiano ostacoli, acqua o decorazioni interattive.
- Evitare scansioni complete per entità o per fotogramma quando esiste già un indice spaziale.
- Non introdurre dipendenze esterne per comodità di sviluppo se diventano necessarie a runtime.
- Aggiornare la documentazione quando cambia intenzionalmente il comportamento descritto; non attribuire alla versione modificata i risultati di test della versione di riferimento.

## Verifica

Per ogni modifica eseguire almeno:

1. una ricerca statica delle stringhe e dei simboli interessati;
2. un controllo di sintassi del JavaScript incorporato, se è disponibile un runtime locale;
3. una prova nel browser proporzionata all'area modificata.

Per modifiche al gameplay o all'interfaccia, usare i casi pertinenti della matrice T01–T72 nella sezione 23 del documento di design. Controllare almeno giorno/notte, meteo, input, salvataggio/caricamento e una scena affollata quando il cambiamento può influenzarli.

Distinguere sempre nel resoconto:

- test logici o simulati;
- rendering canvas;
- prova DOM/input in un browser reale;
- verifica percettiva di grafica e audio.

Non dichiarare superati test non eseguiti. I “104/104” documentati appartengono soltanto alla versione di riferimento.

## Criterio di completamento

Consegnare una versione giocabile, non un mockup. Nel riepilogo finale indicare file cambiati, comportamento ottenuto, verifiche realmente eseguite e limiti rimasti. Se una modifica altera regole, prestazioni o formato dei salvataggi, dichiararlo esplicitamente.
