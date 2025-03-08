# Simulatore di Strategia di Investimento a Soglie

Questo progetto implementa una simulazione stocastica di una strategia di investimento basata su soglie di drawdown. La strategia prevede acquisti progressivi a livelli predefiniti di calo del prezzo (es. -10%, -20%, ...) e vendite automatiche al recupero parziale (es. +5% dalla soglia di acquisto).

## Caratteristiche Principali

- **Generazione Prezzi Stocastici**: Modello geometric Brownian motion con volatilità e drift configurabili.
- **Gestione Posizioni**: Acquisti a soglie di drawdown e vendite al recupero parziale.
- **Tracciamento Dettagliato**: Registrazione di capitale, quantità detenuta, guadagni e valore portafoglio.
- **Visualizzazione Dati**: Grafici interattivi per analizzare prezzo, operazioni, capitale e performance.

## Utilizzo

1. Configura i parametri iniziali (capitale, soglie, volatilità, ecc.).
2. Esegui la simulazione per generare il percorso dei prezzi e le operazioni.
3. Analizza i risultati attraverso i grafici e il riepilogo finale.

## Metriche Calcolate

- Valore finale del portafoglio
- Guadagni/perdite cumulativi
- Numero di acquisti e vendite
- Rendimento percentuale
