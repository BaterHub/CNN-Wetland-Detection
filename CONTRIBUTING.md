# Contribuire al progetto di utilizzo del modello HydraNet per riconoscere le aree umide da immagini Sentinel-2

## 📋 Indice

- [Segnalazioni](#segnalazioni)
- [Come Contribuire](#come-contribuire)
  - [Segnalazione di Bug](#segnalazione-di-bug)
  - [Suggerimento di Funzionalità](#suggerimento-di-funzionalità)
  - [Pull Request](#pull-request)
- [Stile del Codice](#stile-del-codice)
- [Versioning](#versioning)
- [Licenza](#licenza)

## Segnalazioni

Si prega di segnalare osservazioni a [patrizio.petricca@yahoo.it](mailto:patrizio.petricca@yahoo.it).

## Come Contribuire

### Segnalazione di Bug

I bug possono essere segnalati aprendo una nuova issue su GitHub. Prima di creare una nuova issue, controlla se il problema è già stato segnalato.

**Includi nella tua segnalazione**:
- Un titolo chiaro e descrittivo
- Passaggi precisi per riprodurre il problema
- Comportamento atteso vs comportamento osservato
- Screenshot se applicabili
- Informazioni sul tuo ambiente (OS, versione Python, dipendenze)

### Suggerimento di Funzionalità

I suggerimenti per nuove funzionalità sono sempre benvenuti:

1. Verifica che la funzionalità non sia già in sviluppo o pianificata
2. Apri una nuova issue etichettata come "enhancement"
3. Fornisci una descrizione dettagliata della funzionalità proposta
4. Spiega perché questa funzionalità sarebbe utile per il progetto

### Pull Request

1. Forka il repository
2. Crea un nuovo branch dal branch `main`
   ```bash
   git checkout -b feature/nome-funzionalita
   ```
3. Implementa le tue modifiche
4. Assicurati che il codice passi tutti i test
   ```bash
   pytest
   ```
5. Assicurati che il tuo codice segua le convenzioni di stile
   ```bash
   flake8
   ```
6. Commita le tue modifiche
   ```bash
   git commit -m "Aggiungi: breve descrizione della modifica"
   ```
7. Pusha il branch
   ```bash
   git push origin feature/nome-funzionalita
   ```
8. Apri una Pull Request su GitHub

**La tua Pull Request dovrebbe**:
- Avere un titolo chiaro e descrittivo
- Includere una descrizione dettagliata delle modifiche
- Fare riferimento a qualsiasi issue correlata
- Aggiornare la documentazione se necessario

## Stile del Codice

### Python

Seguiamo le convenzioni PEP 8 per il codice Python:

- Usa 4 spazi per l'indentazione (non tab)
- Limita le linee a 88 caratteri
- Usa docstring in stile Google per le funzioni e le classi
- Usa nomi significativi per variabili e funzioni
- Commenta il codice complesso

### Jupyter Notebook

Per i notebook:

- Organizza il notebook in sezioni logiche con markdown
- Includi output rappresentativi per celle chiave
- Pulisci l'output delle celle prima di committare (usa `nbstripout`)
- Fornisci descrizioni chiare per ogni blocco di codice

## Versioning

Utilizziamo [SemVer](http://semver.org/) per il versioning:

- MAJOR.MINOR.PATCH
- Incremento MAJOR per modifiche incompatibili con le versioni precedenti
- Incremento MINOR per aggiunte di funzionalità retrocompatibili
- Incremento PATCH per correzioni di bug retrocompatibili

## Documentazione

La documentazione è fondamentale per la manutenibilità del progetto:

- Aggiorna il README.md quando aggiungi funzionalità significative
- Mantieni aggiornati i docstring nelle funzioni e nelle classi
- Aggiungi commenti esplicativi per algoritmi complessi
- Includi esempi d'uso quando appropriato

## Licenza

Contribuendo a questo progetto, accetti che il tuo lavoro sarà distribuito sotto la stessa licenza del progetto (MIT License).