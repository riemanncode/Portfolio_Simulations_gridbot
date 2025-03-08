import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from collections import defaultdict

# ==============================
# 1. CONFIGURAZIONE INIZIALE
# ==============================
np.random.seed(42)

# Parametri personalizzabili
initial_capital = 1_000_000
initial_invested_pct = 0.08  # 8% inizialmente investito
initial_pmc = 1050  # Prezzo medio di carico iniziale
grid_pct = 0.05  # es: 0.05 => operazioni ogni -5% (se impostato a 0.01 saranno -1%, -2%, etc.)
increment_qty = 0.005  # incremento del capitale investito: pct_capital = 0.015 + i * increment_qty

# Creazione della griglia delle soglie: da 0 a -1.0 a step di -grid_pct
# Es. se grid_pct = 0.05 => thresholds = [0, -0.05, -0.10, -0.15, ..., -0.95]
thresholds = np.arange(1*grid_pct, -1.0, -grid_pct)

# ==============================
# 2. PARAMETRI SIMULAZIONE PREZZI
# ==============================
days = 365
volatility = 0.02  # 2% volatilità giornaliera
drift = 0.0  # drift = 0 per oscillazioni casuali
starting_price = 1218

# ==============================
# 3. GENERAZIONE PREZZI STOCASTICI
# ==============================
daily_returns = np.exp(drift + volatility * np.random.randn(days))
price_path = starting_price * np.cumprod(daily_returns)

# ==============================
# 4. INIZIALIZZAZIONE VARIABILI
# ==============================
# a) Posizione iniziale (8% del capitale investita a 1050)
invested_amount = initial_capital * initial_invested_pct
initial_qty_held = invested_amount / initial_pmc
capital = initial_capital - invested_amount  # capitale liquido residuo
qty_held = initial_qty_held

# b) Picco dinamico (ndq_peak) iniziale
ndq_peak = max(starting_price, price_path[0])

# c) Dizionario delle posizioni aperte (strategie BUY a griglia)
#    Chiave = soglia (es. -0.15), Valore = dict con info {'entry_price', 'qty', 'invested'}
open_positions = dict()

# d) Variabili per tracciare la storia e per il DataFrame di debug
history = []  # per grafici e analisi finali
debug_data = []  # per costruire il DataFrame con TUTTE le variabili

buy_signals = []
sell_signals = []

print("=== SITUAZIONE INIZIALE ===")
print(f"Capitale totale: {initial_capital:,.2f} €")
print(f"Quota inizialmente investita: {invested_amount:,.2f} € ({initial_invested_pct * 100:.1f}%)")
print(f"Prezzo medio di carico (PMC): {initial_pmc:.2f} €")
print(f"Quantità iniziale detenuta: {initial_qty_held:.4f}")
print(f"Capitale liquido rimanente: {capital:,.2f} €")
print("=" * 50, "\n")

# ==============================
# 5. CICLO DI SIMULAZIONE
# ==============================
for day, current_price in enumerate(price_path):
    # Salva i valori iniziali del giorno (prima delle operazioni)
    capital_before = capital
    qty_held_before = qty_held

    # ----------------------------
    # PRINT INIZIALE DEL GIORNO
    # ----------------------------
    print(f"\n=== GIORNO {day} ===")
    print(f"Prezzo NDQ: {current_price:.2f}")

    # 5a) Aggiornamento del picco dinamico
    if current_price > ndq_peak:
        ndq_peak = current_price
    print(f"Picco attuale: {ndq_peak:.2f}")

    # 5b) Calcolo del drop corrente rispetto al picco
    current_drop = (current_price - ndq_peak) / ndq_peak  # es: -0.15 per un drop del 15%
    print(f"Drop attuale: {current_drop:.2%}")
    print(f"Capitale disponibile prima delle operazioni: {capital_before:,.2f} €")
    print(
        f"Quantità detenuta prima delle operazioni: {qty_held_before:.4f} (valore: {qty_held_before * current_price:,.2f} €)")

    # Inizializza variabili per tracciare le azioni del giorno
    action_list = []
    qty_change_total = 0.0
    pct_invested_list = []  # per salvare i valori di pct_capital usati per ogni BUY

    # ----------------------------
    # FASE DI VENDITA
    # ----------------------------
    closed_gains = 0.0
    thresholds_to_close = []
    for T, pos in open_positions.items():
        sell_trigger = T + grid_pct  # es: se T = -0.15 e grid_pct = 0.05 => sell_trigger = -0.10
        if current_drop >= sell_trigger:
            gain = (current_price - pos['entry_price']) * pos['qty']
            closed_gains += gain
            capital += pos['qty'] * current_price
            qty_held -= pos['qty']
            thresholds_to_close.append(T)
            sell_signals.append((day, current_price))
            action_list.append(f"SELL at {T:.2%}")
            qty_change_total -= pos['qty']
            print(f"*** VENDITA *** Soglia: {T:.2%}")
            print(f"    - entry_price: {pos['entry_price']:.2f}")
            print(f"    - current_price: {current_price:.2f}")
            print(f"    - qty vendute: {pos['qty']:.4f}")
            print(f"    - gain: {gain:,.2f} €")
    for T in thresholds_to_close:
        del open_positions[T]

    # ----------------------------
    # FASE DI ACQUISTO
    # ----------------------------
    for i, T in enumerate(thresholds):
        if T not in open_positions and current_drop <= T:
            pct_capital = 0.015 + i * increment_qty  # es: 0.015, 0.015+increment_qty, etc.
            invest_amount = pct_capital * capital
            if invest_amount > 100:  # evitiamo acquisti troppo piccoli
                qty = invest_amount / current_price
                open_positions[T] = {
                    'entry_price': current_price,
                    'qty': qty,
                    'invested': invest_amount
                }
                capital -= invest_amount
                qty_held += qty
                buy_signals.append((day, current_price))
                action_list.append(f"BUY at {T:.2%}")
                qty_change_total += qty
                pct_invested_list.append(pct_capital)
                print(f"+++ ACQUISTO +++ Soglia: {T:.2%}")
                print(f"    - current_price: {current_price:.2f}")
                print(f"    - invest_amount: {invest_amount:,.2f} €")
                print(f"    - qty comprate: {qty:.4f}")

    # ----------------------------
    # CALCOLO VALORI FINALE GIORNALIERO
    # ----------------------------
    portfolio_value = capital + qty_held * current_price
    print(f"--> Guadagno/Perdita realizzato OGGI: {closed_gains:,.2f} €")
    print(f"--> Capitale finale giornata: {capital:,.2f} €")
    print(f"--> Valore totale portafoglio: {portfolio_value:,.2f} €")
    print("--------------------------------------------------------")

    # Salvataggio dei dati per la storia (per grafici/analisi finali)
    history.append({
        'day': day,
        'price': current_price,
        'ndq_peak': ndq_peak,
        'drop': current_drop,
        'capital': capital,
        'qty_held': qty_held,
        'portfolio_value': portfolio_value,
        'closed_gains': closed_gains
    })

    # ----------------------------
    # COSTRUZIONE DELLA RIGA DI DEBUG
    # ----------------------------
    debug_row = {
        'day': day,
        'current_price': current_price,
        'ndq_peak': ndq_peak,
        'current_drop': current_drop,
        'capital_before': capital_before,
        'qty_held_before': qty_held_before,
        'capital_after': capital,
        'qty_held_after': qty_held,
        'closed_gains_today': closed_gains,
        'portfolio_value': portfolio_value,
        'open_positions': str(open_positions),
        'action': "; ".join(action_list) if action_list else "NONE",
        'qty_change': qty_change_total,
        'pct_invested': "; ".join([f"{x:.4f}" for x in pct_invested_list]) if pct_invested_list else "NA"
    }
    debug_data.append(debug_row)

# ==============================
# 6. CREAZIONE DEL DATAFRAME DI DEBUG
# ==============================
df_debug = pd.DataFrame(debug_data)

# ==============================
# 7. ANALISI RISULTATI FINALI
# ==============================
cumulative_gains = np.cumsum([h['closed_gains'] for h in history])
portfolio_values = [h['portfolio_value'] for h in history]
cap_list = [h['capital'] for h in history]

final_value = portfolio_values[-1]
total_gain = final_value - initial_capital
days_range = np.arange(days)

print("\n=== DATAFRAME DI DEBUG (prime 10 righe) ===")
print(df_debug.head(10))

# ==============================
# 8. GRAFICI
# ==============================
plt.figure(figsize=(15, 10))

# 8a) Prezzo con segnali di acquisto e vendita
plt.subplot(2, 2, 1)
plt.plot(price_path, label='Prezzo NDQ', alpha=0.7)
if buy_signals:
    bdays, bprices = zip(*buy_signals)
    plt.scatter(bdays, bprices, color='green', label='Acquisti', zorder=5)
if sell_signals:
    sdays, sprices = zip(*sell_signals)
    plt.scatter(sdays, sprices, color='red', label='Vendite', zorder=5)
plt.title('Andamento Prezzo con Operazioni')
plt.xlabel("Giorni")
plt.ylabel("Prezzo")
plt.legend()

# 8b) Evoluzione del capitale liquido e valore totale del portafoglio
plt.subplot(2, 2, 2)
plt.plot(cap_list, label='Capitale Liquido')
plt.plot(portfolio_values, label='Valore Totale Portafoglio')

# Aggiunta della linea nera con (Valore di Portafoglio + Gains Cumulati + Capitale Liquido)
# ATTENZIONE: qui i gains cumulati sono già inclusi in 'capital',
# quindi stiamo "doppiamente" sommando i realized gains.
portfolio_value_plus_cg = [pv + cg for pv, cg in zip(portfolio_values, cumulative_gains)]
plt.plot(portfolio_value_plus_cg, color='black', label='Valore + Gains Realizzati')

plt.title('Evoluzione del Capitale')
plt.xlabel("Giorni")
plt.ylabel("Euro")
plt.legend()

# 8c) Guadagni/Perdite Cumulativi
plt.subplot(2, 2, 3)
plt.plot(cumulative_gains, 'g-' if cumulative_gains[-1] >= 0 else 'r-')
plt.title('Guadagni/Perdite Cumulativi')
plt.axhline(0, color='black', linestyle='--')
plt.xlabel("Giorni")
plt.ylabel("Euro")

# 8d) Quantità Asset Detenuta
plt.subplot(2, 2, 4)
plt.plot([h['qty_held'] for h in history], color='purple')
plt.title('Quantità Asset Detenuta')
plt.xlabel("Giorni")
plt.ylabel("Quote")

# plt.tight_layout()
# plt.show()

# ==============================
# 9. RIEPILOGO FINALE
# ==============================
print(f"""
=== RISULTATO FINALE ===
Capitale Iniziale: €{initial_capital:,.2f}
Valore Finale Portafoglio: €{final_value:,.2f}
Guadagno/Perdita Totale: €{total_gain:+,.2f}
Rendimento Percentuale: {(final_value / initial_capital - 1):.2%}

Numero Acquisti: {len(buy_signals)}
Numero Vendite: {len(sell_signals)}
""")
