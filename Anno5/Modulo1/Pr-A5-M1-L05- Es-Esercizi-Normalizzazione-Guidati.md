# Esercizi Guidati sulla Normalizzazione – Dal Caos alla 3NF e BCNF

**Materia:** Informatica / Basi di Dati – Anno 5  
**Argomento:** 1NF, 2NF, 3NF, BCNF, dipendenze funzionali  
**Livello:** Intermedio – con soluzione guidata completa  
**Tempo stimato:** 60-90 minuti totali  
**File:** Unico file pronto per GitHub – copia e incolla così com'è

> **Metodo del docente:** Non partire mai dalla tabella finale. Parti sempre dalle domande: *“Cosa devo rappresentare? Quali dipendenze funzionali vedo nel mondo reale? Dove si ripetono i dati?”* Scrivi le dipendenze su carta prima di aprire MySQL Workbench. È l'abitudine che separa un database che dura anni da uno che riscrivi dopo 3 mesi.

---

## Indice

1. [Esercizio 1 – E-commerce: da tabella piatta a 3NF](#esercizio-1--e-commerce-da-tabella-piatta-a-3nf-pulita)
2. [Esercizio 2 – Università: quando la 3NF non basta (BCNF)](#esercizio-2--università-quando-la-3nf-non-basta-caso-bcnf)
3. [Esercizio 3 – Biblioteca: progetto completo scenario → E/R → 3NF](#esercizio-3--biblioteca-progetto-completo-scenario--er--3nf)

---

## Esercizio 1 – E-commerce: da tabella piatta a 3NF pulita

### Scenario
Ti viene passato un file Excel del reparto vendite, una sola tabella:

| id_ordine | data | cliente | città_cliente | provincia | prodotti | quantità | prezzi | venditore | zona_venditore |
|---|---|---|---|---|---|---|---|---|---|
| 1001 | 2024-01-10 | Mario Rossi | Milano | MI | Penna, Quaderno | 2, 1 | 0.50, 2.00 | Giulia Verdi | Nord |
| 1002 | 2024-01-11 | Anna Bianchi | Roma | RM | Monitor | 1 | 250.00 | Marco Neri | Centro |

### Obiettivo
Portare lo schema in 3NF, eliminando tutte le anomalie.

### Svolgimento Guidato

#### Passo 0 – Trova le dipendenze funzionali (fondamentale)
Prima di toccare una tabella, scrivi le regole del mondo reale:

```
id_ordine → data, cliente, venditore
cliente → città_cliente
città_cliente → provincia  (dipendenza transitiva!)
id_prodotto → nome_prodotto, prezzo_listino
venditore → zona_venditore (transitiva!)
(id_ordine, id_prodotto) → quantità, prezzo_unitario
```

> **Consiglio del docente:** Se non scrivi le dipendenze, stai improvvisando. Scrivile. Tutte. Anche quelle che ti sembrano ovvie.

#### Passo 1 – Violazione 1NF
**Problema:** `prodotti`, `quantità`, `prezzi` contengono liste separate da virgola. Non sono atomici.

**Regola 1NF:** Ogni attributo deve contenere un valore singolo. Una tabella deve avere una chiave primaria.

**Correzione → 1NF:**
Dividi ogni prodotto in una riga. La PK diventa composta `(id_ordine, id_prodotto)`.

| id_ordine | cliente | id_prodotto | nome_prodotto | quantità |
|---|---|---|---|---|
| 1001 | Mario Rossi | P01 | Penna | 2 |
| 1001 | Mario Rossi | P02 | Quaderno | 1 |
| 1002 | Anna Bianchi | P04 | Monitor | 1 |

**Errore tipico:** Fermarsi qui. Ora il nome cliente è ripetuto per ogni prodotto dello stesso ordine. Abbiamo risolto 1NF ma creato ridondanza.

#### Passo 2 – Violazione 2NF (dipendenza parziale)
PK composta = `(id_ordine, id_prodotto)`

- `(id_ordine, id_prodotto) → quantità` OK, serve tutta la PK
- `id_ordine → cliente` NO, dipende solo da metà PK → **parziale**
- `id_prodotto → nome_prodotto` NO, dipende solo da metà PK → **parziale**

**Regola 2NF:** Tutti gli attributi non-chiave devono dipendere dall'intera PK.

**Correzione → 2NF:** Estrai le dipendenze parziali.

```
ORDINE(id_ordine PK, data, id_cliente FK)
CLIENTE(id_cliente PK, nome, cognome, città)
PRODOTTO(id_prodotto PK, nome, prezzo_listino)
DETTAGLIO_ORDINE(id_ordine FK, id_prodotto FK, quantità, prezzo_unitario) PK: (id_ordine, id_prodotto)
```

#### Passo 3 – Violazione 3NF (dipendenza transitiva)
Guarda CLIENTE:

```
id_cliente → città_cliente
città_cliente → provincia
Quindi: id_cliente → città_cliente → provincia (TRANSITIVA)
```

Stessa cosa per venditore: `venditore → zona`

**Regola 3NF:** Nessun attributo non-chiave deve dipendere da un altro attributo non-chiave.

**Correzione → 3NF:**

```
CITTÀ(id_città PK, nome, provincia)
ZONA_VENDITA(id_zona PK, nome)
VENDITORE(id_venditore PK, nome, id_zona FK)
CLIENTE(id_cliente PK, nome, cognome, id_città FK)
PRODOTTO(id_prodotto PK, nome, prezzo)
ORDINE(id_ordine PK, data, id_cliente FK, id_venditore FK)
DETTAGLIO_ORDINE(id_ordine FK, id_prodotto FK, quantità, prezzo_unitario)
```

**Risultato:** Da 1 tabella con 10 colonne e 3 anomalie → 6 tabelle pulite. Ogni dato in un solo posto. Aggiorni il CAP di Milano una volta sola.

> **Caso d'uso reale:** Con la tabella piatta, se cambi zona a un venditore devi aggiornare 50 ordini. Con la 3NF, un UPDATE in ZONA_VENDITA.

---

## Esercizio 2 – Università: quando la 3NF non basta (caso BCNF)

### Scenario
Relazione universitaria:

```
ISCRIZIONE(Studente, Materia, Docente)
```

Regole del mondo reale (vincoli):
1. Per ogni coppia (Studente, Materia) c'è un solo Docente → `(Studente, Materia) → Docente`
2. Ogni Docente insegna una sola Materia → `Docente → Materia`

Da queste, due chiavi candidate: `{Studente, Materia}` e `{Studente, Docente}`

Dati di esempio:

| Studente | Materia | Docente |
|---|---|---|
| Rossi | Basi di Dati | Bianchi |
| Verdi | Basi di Dati | Bianchi |
| Neri | Reti | Gialli |

### Domanda
È in 3NF? È in BCNF?

### Svolgimento Guidato

#### Passo 1 – Verifica 3NF
Per ogni dipendenza X → Y, o X è superchiave, o Y è attributo primo (fa parte di una chiave candidata).

- `(Studente, Materia) → Docente`: X è chiave → OK per 3NF
- `Docente → Materia`: X=Docente non è superchiave, MA Y=Materia è primo (fa parte della chiave {Studente, Materia}) → **Tollerato in 3NF**

→ La relazione **è in 3NF**.

#### Passo 2 – Verifica BCNF
Regola BCNF (più severa): Per ogni dipendenza X → Y non banale, X DEVE essere superchiave. Nessuna eccezione per attributi primi.

- `Docente → Materia`: Docente da solo determina Materia, ma non determina Studente. Quindi Docente NON è superchiave.

→ **Violazione BCNF.** C'è ridondanza: ogni volta che vedo Bianchi, so già che insegna Basi di Dati, ripetuto per ogni studente.

#### Passo 3 – Decomposizione BCNF
Separa la dipendenza problematica:

```
DOCENTE_MATERIA(Docente PK, Materia)  // Docente → Materia, Docente è chiave
STUDENTE_DOCENTE(Studente, Docente) PK: (Studente, Docente)
```

Ora ogni tabella rispetta BCNF.

> **Nota del docente – perché è difficile:** La BCNF si vede solo quando hai più chiavi candidate sovrapposte. È il caso più subdolo. Se alla prima lettura non vi torna, è normale. Il trucco che uso io: chiediti “questo determinante, da solo, identifica tutta la riga?”. Se no, sei fuori BCNF. Qui Docente da solo non identifica Studente → fuori.

---

## Esercizio 3 – Biblioteca: progetto completo scenario → E/R → 3NF

### Scenario (traccia da esame)
La biblioteca comunale vuole gestire prestiti.

Requisiti:
- Un libro ha ISBN (PK), titolo, anno, editore. Un editore ha nome e città. Un editore pubblica molti libri.
- Un libro può avere più autori. Un autore ha CF, nome, cognome.
- Una copia fisica ha codice copia (PK) + ISBN del libro + stato (disponibile/prestito).
- Un utente ha tessera (PK), nome, cognome, città, provincia, CAP.
- Un prestito ha id_prestito, data_inizio, data_fine, utente, copia. Un utente può prendere più copie in prestiti diversi.

### Svolgimento Guidato

#### Passo 0 – Dipendenze funzionali (su carta)
```
ISBN → titolo, anno, id_editore
id_editore → nome_editore, città_editore
id_autore → nome, cognome
id_copia → ISBN, stato
tessera → nome, cognome, città
città → provincia, CAP (transitiva!)
id_prestito → data_inizio, data_fine, tessera, id_copia
```

#### Passo 1 – Concettuale E/R (descrivi a parole, poi disegna)
Entità: LIBRO, EDITORE, AUTORE, COPIA, UTENTE, CITTÀ, PRESTITO
Relazioni: PUBBLICA (Editore 1-N Libro), SCRITTO_DA (Libro N-M Autore), HA_COPIA (Libro 1-N Copia), EFFETTUA (Utente 1-N Prestito), RIGUARDA (Copia 1-N Prestito)

#### Passo 2 – Logico non normalizzato (se parti da Excel)
Se metti tutto in PRESTITO:

```
PRESTITO_ALL_IN_ONE(id_prestito, data, tessera, nome_utente, città, provincia, CAP, id_copia, ISBN, titolo, autori_lista, editore, città_editore)
```

Violazioni evidenti: autori_lista è lista → no 1NF, città→provincia transitiva → no 3NF, ISBN→titolo parziale se PK composta.

#### Passo 3 – Normalizzazione completa fino a 3NF

**1NF:** Separa autori multipli → tabella AUTORE e tabella ponte LIBRO_AUTORE.

**2NF:** Elimina dipendenze parziali (se avevi PK composta id_prestito + id_copia).

**3NF:** Estrai CITTÀ e EDITORE.

Schema finale in 3NF (pronto per SQL):

```sql
CITTÀ(id_città PK, nome, provincia, cap)

EDITORE(id_editore PK, nome, id_città FK)

AUTORE(cf PK, nome, cognome)

LIBRO(isbn PK, titolo, anno, id_editore FK)

LIBRO_AUTORE(isbn FK, cf_autore FK) PK(isbn, cf_autore)

UTENTE(tessera PK, nome, cognome, id_città FK)

COPIA(id_copia PK, isbn FK, stato)

PRESTITO(id_prestito PK, data_inizio, data_fine, tessera FK, id_copia FK)
```

#### Passo 4 – Verifica anomalie risolte
- Inserimento: posso inserire una città senza utenti? Sì (tabella CITTÀ indipendente)
- Cancellazione: se cancello l'ultimo prestito di un utente, perdo l'utente? No, UTENTE è separato
- Aggiornamento: cambio CAP di una città? Un UPDATE in CITTÀ

> **Consiglio finale per GitHub:** Consegna sempre 3 file: 1) dipendenze.txt 2) schema ER (foto o draw.io) 3) schema.sql con CREATE TABLE. Chi corregge vuole vedere il ragionamento, non solo le tabelle finali.

---

## Checklist di Autovalutazione per Tutti gli Esercizi

- [ ] Ho scritto tutte le dipendenze funzionali prima di disegnare?
- [ ] Ogni tabella ha PK definita?
- [ ] In 1NF ho eliminato liste e gruppi ripetuti?
- [ ] In 2NF ho controllato se la PK è composta e se qualche attributo dipende solo da metà?
- [ ] In 3NF ho cercato dipendenze transitive non-chiave → non-chiave?
- [ ] Ho verificato BCNF chiedendo “ogni determinante è superchiave?”
- [ ] Posso rispondere alle 3 anomalie (inserimento, cancellazione, aggiornamento) per ogni schema?

## Consegna GitHub Suggerita

```
/normalizzazione-esercizi/
  README.md (questo file)
  esercizio1/
    dipendenze.md
    schema_3NF.sql
  esercizio2/
    analisi_BCNF.md
  esercizio3/
    scenario.md
    schema_ER.png
    schema_finale.sql
```

---

**Autore:** Prof. Giagnotti – Informatica Anno 5 Modulo 1  
**Licenza:** CC BY-SA 4.0 – Uso didattico libero  
**Nota Personale:** Questi esercizi nascono dagli errori che vedo ogni anno in classe: la paura di fare “troppe tabelle” e la tentazione di mettere tutto in una tabella unica perché “così è più semplice”. Non lo è. Una tabella enorme è come un cassetto dove butti chiavi, calzini e documenti: all'inizio sembra comodo, dopo due settimane non trovi più nulla. Normalizzare è mettere ordine nei cassetti.
