# Esercizi Guidati sull'Algebra Relazionale – Dal Linguaggio Naturale alle Query Formali

**Materia:** Informatica / Basi di Dati – Anno 5  
**Argomento:** Algebra relazionale – selezione, proiezione, join, unione, differenza, ridenominazione, divisione  
**Livello:** Intermedio-Avanzato – con soluzione guidata completa  
**Tempo stimato:** 60-90 minuti  
**File:** Unico file pronto per GitHub

> **Metodo del docente:** L'algebra relazionale non è un esercizio di simboli. È il modo di pensare prima di scrivere SQL. Quando vi chiedo una query in algebra, vi sto chiedendo: “quali tabelle ti servono davvero, quali righe vuoi tenere, quali colonne vuoi vedere?” Se rispondete a queste tre domande in ordine, la formula viene da sola. Io in classe la faccio sempre così: prima a parole, poi con gli operatori.

---

## Indice

1. [Richiamo veloce degli operatori](#richiamo-veloce-degli-operatori)
2. [Esercizio 1 – E-commerce: selezione, proiezione e join](#esercizio-1--e-commerce-selezione-proiezione-e-join)
3. [Esercizio 2 – Biblioteca: join multipli, differenza e divisione](#esercizio-2--biblioteca-join-multipli-differenza-e-divisione)
4. [Esercizio 3 – Università: casi limite, outer join logico e ridenominazione](#esercizio-3--università-casi-limite-outer-join-logico-e-ridenominazione)

---

## Richiamo veloce degli operatori

| Operatore | Simbolo | Cosa fa | Esempio lettura |
|---|---|---|---|
| Selezione | σ condizione(R) | Filtra righe | “tieni solo le righe dove…” |
| Proiezione | π attributi(R) | Filtra colonne, elimina duplicati | “mostrami solo…” |
| Prodotto cartesiano | R × S | Tutte le combinazioni | Base per join |
| Join naturale | R ⋈ S | Unisce su attributi con stesso nome | Join implicito su chiavi |
| Theta-join | R ⋈ condizione S | Unisce con condizione | Join con condizione esplicita |
| Unione | R ∪ S | Righe di R o S (compatibili) | |
| Intersezione | R ∩ S | Righe in entrambe | |
| Differenza | R - S | Righe in R non in S | “quelli che non…” |
| Ridenominazione | ρ nuovo(R) o ρ nuovo(attr)(R) | Rinomina relazione o attributi | Utile per self-join |
| Divisione | R ÷ S | “tutti quelli che hanno tutti” | La più difficile, vediamo esercizio 2 |

> **Nota personale:** Non memorizzate i simboli a memoria come se fossero geroglifici. Scrivete prima a parole: “Voglio i clienti di Milano che hanno comprato un monitor”. Poi traducete: selezione città=Milano, join con ordini e prodotti, proiezione nome. I simboli sono solo abbreviazioni.

---

## Esercizio 1 – E-commerce: selezione, proiezione e join

### Schema (già in 3NF – quello dell'esercitazione precedente)

```
CITTÀ(id_città PK, nome, provincia, cap)
CLIENTE(id_cliente PK, nome, cognome, id_città FK)
PRODOTTO(id_prodotto PK, nome, prezzo)
ORDINE(id_ordine PK, data, id_cliente FK)
DETTAGLIO_ORDINE(id_ordine FK, id_prodotto FK, quantità)
```

Dati di esempio minimali (per seguire i passaggi):

**CLIENTE**
| id_cliente | nome | cognome | id_città |
|---|---|---|---|
| C1 | Mario | Rossi | CT1 |
| C2 | Anna | Bianchi | CT2 |

**CITTÀ**
| id_città | nome | provincia |
|---|---|---|
| CT1 | Milano | MI |
| CT2 | Roma | RM |

**PRODOTTO**
| id_prodotto | nome | prezzo |
|---|---|---|
| P1 | Penna | 0.50 |
| P2 | Monitor | 250 |

**ORDINE**
| id_ordine | data | id_cliente |
|---|---|---|
| O1 | 2024-01-10 | C1 |
| O2 | 2024-01-11 | C2 |

**DETTAGLIO_ORDINE**
| id_ordine | id_prodotto | quantità |
|---|---|---|
| O1 | P1 | 2 |
| O1 | P2 | 1 |
| O2 | P2 | 1 |

### Query da risolvere

**Q1.1:** Tutti i clienti di Milano.

**Q1.2:** Nome e cognome di tutti i clienti (senza duplicati).

**Q1.3:** Nome dei prodotti ordinati da Mario Rossi.

**Q1.4:** Clienti che NON hanno mai ordinato una Penna.

### Svolgimento Guidato

#### Q1.1 – Clienti di Milano

**A parole:** Mi serve CLIENTE + CITTÀ per filtrare per nome città.

**Passi:**
1. Unisco CLIENTE e CITTÀ su id_città
2. Seleziono solo righe dove CITTÀ.nome = 'Milano'
3. Proietto solo gli attributi di CLIENTE (o tutti, a seconda di cosa chiedono)

**Formula:**

```
CLIENTE_MILANO = σ nome='Milano' (CITTÀ)

RISULTATO = π CLIENTE.* (CLIENTE ⋈ id_città=id_città CLIENTE_MILANO)
```

Versione compatta con natural join se rinomino:

```
RISULTATO = π id_cliente, nome, cognome ( σ nome_città='Milano' (CLIENTE ⋈ CITTÀ) )
```

> **Nota:** Se usate il natural join, attenzione ai nomi attributi uguali. Qui id_città è uguale in entrambe, quindi ⋈ naturale funziona. Se avete chiamato l'attributo in modo diverso, usate theta-join.

**SQL equivalente:**

```sql
SELECT CLIENTE.* FROM CLIENTE JOIN CITTÀ ON CLIENTE.id_città = CITTÀ.id_città
WHERE CITTÀ.nome = 'Milano';
```

#### Q1.2 – Nome e cognome (proiezione pura)

```
RISULTATO = π nome, cognome (CLIENTE)
```

Sembra banale, ma ricordate: la proiezione elimina i duplicati. Se avete due Mario Rossi, ne vedete uno solo. In SQL sarebbe `SELECT DISTINCT`.

> **Errore tipico che vedo sempre:** Confondere selezione e proiezione. Selezione = filtro su righe (orizzontale). Proiezione = filtro su colonne (verticale). Io lo ripeto così: “σ sta in piedi come una riga, π sta in piedi come una colonna”.

#### Q1.3 – Prodotti ordinati da Mario Rossi

**A parole:** Mario Rossi → suoi ordini → dettagli → prodotti.

**Catena di join:**

```
1. Seleziono Mario: σ nome='Mario' ∧ cognome='Rossi' (CLIENTE)
2. Join con ORDINE
3. Join con DETTAGLIO_ORDINE
4. Join con PRODOTTO
5. Proietto nome prodotto
```

**Formula completa:**

```
MARIO = σ nome='Mario' ∧ cognome='Rossi' (CLIENTE)

RISULTATO = π PRODOTTO.nome (
  PRODOTTO ⋈ DETTAGLIO_ORDINE ⋈ ORDINE ⋈ MARIO
)
```

Se volete essere espliciti:

```
RISULTATO = π nome_prodotto (
  σ nome_cliente='Mario' ∧ cognome='Rossi' (
    CLIENTE ⋈ ORDINE ⋈ DETTAGLIO_ORDINE ⋈ PRODOTTO
  )
)
```

#### Q1.4 – Clienti che NON hanno mai ordinato una Penna

Qui serve la **differenza**. È il pattern “tutti meno quelli che…”.

**Passi:**
1. Trova id_cliente di chi HA ordinato Penna
2. Sottrai da tutti i clienti

```
PENNA = σ nome='Penna' (PRODOTTO)

CLIENTI_PENNA = π id_cliente (
  CLIENTE ⋈ ORDINE ⋈ DETTAGLIO_ORDINE ⋈ PENNA
)

TUTTI_CLIENTI = π id_cliente (CLIENTE)

RISULTATO = TUTTI_CLIENTI - CLIENTI_PENNA
```

Poi se vuoi nome e cognome:

```
RISULTATO_FINALE = π nome, cognome (CLIENTE ⋈ RISULTATO)
```

> **Consiglio del docente:** La differenza è l'operatore più sottovalutato. Quando la domanda contiene “non”, “mai”, “nessun”, pensate subito a differenza. Prima trovate chi HA fatto qualcosa, poi sottraete.

---

## Esercizio 2 – Biblioteca: join multipli, differenza e divisione

### Schema

```
LIBRO(isbn PK, titolo, id_editore FK)
AUTORE(cf PK, nome, cognome)
LIBRO_AUTORE(isbn FK, cf_autore FK) PK(isbn, cf_autore)
COPIA(id_copia PK, isbn FK)
UTENTE(tessera PK, nome, cognome)
PRESTITO(id_prestito PK, id_copia FK, tessera FK, data_inizio)
```

### Query

**Q2.1:** Titoli dei libri prestati a Roma? No, scusate, più precisa: Titoli dei libri mai prestati.

**Q2.2:** Utenti che hanno preso in prestito TUTTI i libri di Calvino (divisione – la più difficile).

**Q2.3:** Autori che hanno scritto almeno un libro mai prestato.

### Svolgimento Guidato

#### Q2.1 – Libri mai prestati

Stesso pattern differenza dell'esercizio 1.

```
LIBRI_PRESTATI = π isbn (COPIA ⋈ PRESTITO)

TUTTI_LIBRI = π isbn (LIBRO)

MAI_PRESTATI = TUTTI_LIBRI - LIBRI_PRESTATI

RISULTATO = π titolo (LIBRO ⋈ MAI_PRESTATI)
```

#### Q2.2 – Utenti che hanno preso in prestito TUTTI i libri di Calvino (Divisione)

Questa è la query che mette in crisi tutti la prima volta. Leggiamola bene.

**A parole:** Voglio gli utenti tali che per OGNI libro di Calvino, esiste un prestito di quell'utente per quel libro.

È il classico caso per **divisione**.

**Passi preparatori:**

```
LIBRI_CALVINO = π isbn ( σ cognome='Calvino' (AUTORE ⋈ LIBRO_AUTORE ⋈ LIBRO) )

PRESTITI_UTENTE_LIBRO = π tessera, isbn ( PRESTITO ⋈ COPIA ⋈ LIBRO )
-- ottengo coppie (utente, libro) effettivamente prestate
```

**Divisione:**

```
RISULTATO_TESSERE = PRESTITI_UTENTE_LIBRO ÷ LIBRI_CALVINO
```

Cosa significa? `R ÷ S` restituisce le tessere tali che per ogni isbn in S, la coppia (tessera, isbn) esiste in R.

Poi:

```
RISULTATO = π nome, cognome (UTENTE ⋈ RISULTATO_TESSERE)
```

> **Nota personale – come spiego la divisione in classe:** La divisione è come dire “fammi vedere chi ha collezionato tutta la collezione”. Se Calvino ha 3 libri, chi li ha presi tutti e tre? Non basta averne preso uno. Serve averli presi tutti. È l'operatore più astratto, ma quando vi serve, non c'è alternativa più pulita. Se non vi chiedono esplicitamente “tutti”, evitatela. Se vi chiedono “tutti”, è lei.

**Alternativa senza divisione (con differenza, più lunga ma didattica):**

```
UTENTI_CHE_MANCANO_QUALCOSA = π tessera (
  (π tessera × LIBRI_CALVINO) - PRESTITI_UTENTE_LIBRO
)
RISULTATO = TUTTI_UTENTI - UTENTI_CHE_MANCANO_QUALCOSA
```

Leggetela così: prendo tutti gli utenti × tutti i libri di Calvino (tutte le combinazioni possibili), tolgo quelle che esistono davvero, mi restano le coppie mancanti. Chi ha almeno una mancante, non ha preso tutto.

#### Q2.3 – Autori con almeno un libro mai prestato

```
MAI_PRESTATI come prima

LIBRI_MAI_PRESTATI_COMPLETO = LIBRO ⋈ MAI_PRESTATI

RISULTATO = π nome, cognome ( AUTORE ⋈ LIBRO_AUTORE ⋈ LIBRI_MAI_PRESTATI_COMPLETO )
```

---

## Esercizio 3 – Università: casi limite, outer join logico e ridenominazione

### Schema

```
STUDENTE(matricola PK, nome, cognome, corso_laurea)
CORSO(id_corso PK, nome, docente)
ESAME(matricola FK, id_corso FK, voto) PK(matricola, id_corso)
```

### Query

**Q3.1:** Studenti che non hanno sostenuto nessun esame (outer join logico con differenza).

**Q3.2:** Coppie di studenti dello stesso corso di laurea (self-join con ridenominazione).

**Q3.3:** Corso con media voti più alta? (qui serve estensione con aggregazione – mostro come ragionare)

### Svolgimento Guidato

#### Q3.1 – Studenti senza esami

```
TUTTI_STUDENTI = π matricola (STUDENTE)

STUDENTI_CON_ESAMI = π matricola (ESAME)

SENZA_ESAMI = TUTTI_STUDENTI - STUDENTI_CON_ESAMI

RISULTATO = STUDENTE ⋈ SENZA_ESAMI
```

> **Nota:** In algebra relazionale pura non esiste LEFT OUTER JOIN. Si simula con differenza + unione. In SQL usereste `LEFT JOIN WHERE ESAME.matricola IS NULL`. È importante saper fare entrambe le strade.

#### Q3.2 – Coppie di studenti dello stesso corso (self-join)

Qui serve **ridenominazione**, altrimenti non potete joinare la stessa tabella con se stessa.

**A parole:** Voglio coppie (S1, S2) tali che S1.corso = S2.corso e S1.matricola < S2.matricola (per evitare duplicati e coppie con se stesso).

```
ρ S1(STUDENTE) -- rinomino prima copia
ρ S2(STUDENTE) -- rinomino seconda copia

COPPIE = σ S1.corso_laurea = S2.corso_laurea ∧ S1.matricola < S2.matricola (
  S1 × S2
)

RISULTATO = π S1.nome, S1.cognome, S2.nome, S2.cognome, S1.corso_laurea (COPPIE)
```

La condizione `matricola < matricola` evita sia (Rossi, Verdi) e (Verdi, Rossi) sia (Rossi, Rossi).

> **Errore tipico:** Dimenticare la ridenominazione e scrivere `STUDENTE ⋈ STUDENTE`. Non ha senso, state joinando una tabella con se stessa senza distinguerle. Servono due alias.

#### Q3.3 – Media voti per corso (estensione)

Algebra relazionale classica non ha aggregazione, ma nelle versioni estese si usa:

```
γ id_corso; AVG(voto) → media (ESAME)
```

Formula:

```
MEDIE = γ id_corso; AVG(voto) → media (ESAME)

RISULTATO = CORSO ⋈ MEDIE
```

Se volete solo il corso con media massima, in algebra pura è complesso (serve nidificazione), in SQL fareste:

```sql
SELECT id_corso, AVG(voto) as media FROM ESAME GROUP BY id_corso ORDER BY media DESC LIMIT 1;
```

> **Consiglio finale:** Quando vi chiedo “media, somma, conteggio”, sappiate che state uscendo dall'algebra pura ed entrando nell'estesa. È corretto menzionarlo. Nei compiti di quinta spesso vi chiedo solo selezione/proiezione/join/differenza, non aggregazione, proprio per restare nel puro.

---

## Schemi di Soluzione Rapida da Copiare per lo Studio

### Pattern 1 – “Tutti quelli che hanno fatto X”
```
RIS = π attributi ( σ condizione ( R ⋈ S ⋈ T ) )
```

### Pattern 2 – “Quelli che NON hanno fatto X”
```
HA_FATTO = π id ( ... )
TUTTI = π id (R)
RIS = TUTTI - HA_FATTO
```

### Pattern 3 – “Quelli che hanno fatto TUTTI gli X”
```
R = coppie (utente, oggetto)
S = tutti gli oggetti richiesti
RIS = R ÷ S
```

### Pattern 4 – “Coppie nella stessa tabella”
```
ρ A(R)
ρ B(R)
RIS = σ A.attr = B.attr ∧ A.id < B.id (A × B)
```

---

## Checklist di Autovalutazione

- [ ] So distinguere σ (righe) da π (colonne)?
- [ ] So quando serve un join? (quando devo collegare informazioni di tabelle diverse)
- [ ] So quando serve differenza? (quando c'è “non”, “mai”, “nessuno”)
- [ ] So quando serve divisione? (quando c'è “tutti”)
- [ ] So usare ρ per self-join?
- [ ] So scrivere prima a parole e poi tradurre in simboli?
- [ ] So controllare che le relazioni di unione/differenza siano compatibili (stessi attributi)?

---

**Autore:** Prof. Giagnotti – Informatica Anno 5  
**Licenza:** CC BY-SA 4.0 – Uso didattico  
**Nota personale finale:** L'algebra relazionale sembra astratta finché non scrivete la prima query SQL complessa con 4 join e una NOT EXISTS e vi accorgete che è esattamente la stessa cosa che avete fatto qui, solo con parole diverse. Se riuscite a scrivere queste 8-9 query a mano su carta, senza provare a caso in MySQL, siete già un passo avanti a molti. E se la divisione vi sembra ostica, è normale: è l'ultima che si capisce davvero, ma quando la capite, avete capito tutto il resto.
