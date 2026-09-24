# Esercizi svolti — Dal modello E/R al modello relazionale

> **Corso:** Informatica (Programmazione) · Anno 5 · Modulo 1 · Lezione 03 — Modello relazionale
> **File:** `pr-a5-m1-l03-es-traduzione-er-relazionale.md`
> **Percorso repository suggerito:** `programmazione-labs/a5-m1-l03/`
>
> I diagrammi sono in sintassi Mermaid. Obiettivo: applicare le 3 regole di traduzione: Entità → Tabella, 1:1 / 1:N → Chiave esterna, N:M → Tabella associativa.

## Come leggere gli esercizi

Per ogni esercizio seguiamo 6 passi:

1.  **Individuazione di entità e associazione**
2.  **Analisi in un senso** — cardinalità (min,max) da A a B
3.  **Analisi nell'altro senso** — cardinalità (min,max) da B ad A
4.  **Schema E/R** in Mermaid
5.  **Derivazione delle tabelle** — dove mettiamo la FK? Serve tabella ponte?
6.  **Schema relazionale** — notazione `NOME(<u>PK</u>, attributi, FK)` con PK sottolineata

> Convenzione: `<u>attributo</u>` = Chiave Primaria (PK). `FK → TABELLA` = Chiave Esterna. Se la PK è composta si sottolineano insieme gli attributi: `<u>id1, id2</u>`.

---

## Esercizio 1 — Membro e Credenziali [Obiettivo: 1:1]

**Testo.** In un team di sviluppo di siti web, ad alcuni membri (gli amministratori) sono associate delle credenziali di accesso al pannello (username e password). Ad altri membri (non amministratori) non è associata nessuna credenziale. Ogni credenziale appartiene a un solo membro.

### Analisi

**Entità individuate:** `MEMBRO`, `CREDENZIALE`
**Associazione:** `POSSIEDE`
**Attributi:**
* `MEMBRO(id_membro, nome, cognome, ruolo)`
* `CREDENZIALE(id_credenziale, username, password, data_creazione)`

**Analisi in un senso — da MEMBRO a CREDENZIALE.**
Preso un MEMBRO, quante CREDENZIALI possiede? Un membro non amministratore ne ha 0, un amministratore ne ha 1.
- Min: **0**, Max: **1** → Cardinalità **(0,1)** - partecipazione facoltativa

**Analisi nell'altro senso — da CREDENZIALE a MEMBRO.**
Presa una CREDENZIALE, a quanti MEMBRO appartiene? Una credenziale non può esistere da sola, deve sempre appartenere a uno e un solo membro.
- Min: **1**, Max: **1** → Cardinalità **(1,1)** - partecipazione obbligatoria

Riassumendo: è una **associazione 1:1** con opzionalità su un solo lato: `(0,1)` lato MEMBRO, `(1,1)` lato CREDENZIALE.

### Schema E/R

```mermaid
erDiagram
    MEMBRO ||--o| CREDENZIALE : possiede