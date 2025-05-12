
import streamlit as st
import pandas as pd
import random

# --- Carrega dados simulados ---
data = pd.read_csv("data/jogos.csv")

st.set_page_config(page_title="IA de Apostas em Futebol", layout="wide")
st.title("🌟 IA de Apostas em Futebol - Previsões e Múltiplas")

# --- Sidebar para escolher jogo ---
st.sidebar.header("Escolha um jogo")
jogos = data[["time_casa", "time_fora"]].drop_duplicates()
jogos["partida"] = jogos["time_casa"] + " x " + jogos["time_fora"]
escolhido = st.sidebar.selectbox("Selecione: ", jogos["partida"])

# --- Dados do jogo escolhido ---
selecionado = data[(data["time_casa"] + " x " + data["time_fora"] == escolhido)].iloc[0]

# --- Previsões simuladas ---
def prever_resultado():
    return random.choices(["Casa", "Empate", "Fora"], weights=[0.45, 0.30, 0.25])[0]

def prever_over25():
    return round(random.uniform(0.4, 0.8), 2)

def prever_btts():
    return round(random.uniform(0.3, 0.7), 2)

def prever_escanteios():
    return round(random.uniform(0.5, 0.9), 2)

# --- Odds e Previsões ---
odds = {
    "1X2": {"Casa": selecionado["odd_casa"], "Empate": selecionado["odd_empate"], "Fora": selecionado["odd_fora"]},
    "Over 2.5": selecionado["odd_over25"],
    "BTTS": selecionado["odd_btts"],
    "Escanteios Over 9.5": selecionado["odd_corners"]
}

st.subheader(f"Jogo: {escolhido}")

col1, col2 = st.columns(2)

with col1:
    st.markdown("### Previsões da IA")
    previsao_1x2 = prever_resultado()
    prob_over25 = prever_over25()
    prob_btts = prever_btts()
    prob_corners = prever_escanteios()

    st.write(f"**Resultado Provável**: {previsao_1x2}")
    st.write(f"**Chance de Over 2.5 Gols**: {prob_over25 * 100:.1f}%")
    st.write(f"**Chance de Ambos Marcam (BTTS)**: {prob_btts * 100:.1f}%")
    st.write(f"**Chance de Over 9.5 Escanteios**: {prob_corners * 100:.1f}%")

with col2:
    st.markdown("### Odds e Valor Esperado")
    ev_over25 = (prob_over25 * odds["Over 2.5"]) - 1
    ev_btts = (prob_btts * odds["BTTS"]) - 1
    ev_corners = (prob_corners * odds["Escanteios Over 9.5"]) - 1

    st.write(f"Over 2.5 Gols - Odd: {odds['Over 2.5']} | EV: {ev_over25:.2f}")
    st.write(f"BTTS - Odd: {odds['BTTS']} | EV: {ev_btts:.2f}")
    st.write(f"Over Escanteios - Odd: {odds['Escanteios Over 9.5']} | EV: {ev_corners:.2f}")

# --- Sugestão de Aposta Combinada ---
st.markdown("---")
st.header("🎮 Sugestão de Aposta Múltipla")

apostas = []
if ev_over25 > 0:
    apostas.append({"mercado": "Over 2.5 Gols", "odd": odds["Over 2.5"], "prob": prob_over25})
if ev_btts > 0:
    apostas.append({"mercado": "Ambos Marcam (BTTS)", "odd": odds["BTTS"], "prob": prob_btts})
if ev_corners > 0:
    apostas.append({"mercado": "Over 9.5 Escanteios", "odd": odds["Escanteios Over 9.5"], "prob": prob_corners})

if apostas:
    odd_total = 1
    prob_total = 1
    for aposta in apostas:
        odd_total *= aposta["odd"]
        prob_total *= aposta["prob"]
        st.write(f"- {aposta['mercado']} (Odd {aposta['odd']})")

    ev_comb = (prob_total * odd_total) - 1
    st.write(f"
**Odd Combinada**: {odd_total:.2f} | **Prob Estimada**: {prob_total*100:.1f}%")
    st.write(f"**Valor Esperado da Múltipla**: {ev_comb:.2f}")
    st.success("Aposta recomendada com valor positivo!") if ev_comb > 0 else st.warning("Valor esperado não positivo.")
else:
    st.info("Nenhuma aposta com valor positivo para esta partida.")
