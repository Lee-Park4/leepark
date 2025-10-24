import streamlit as st
import yfinance as yf
import pandas as pd
import plotly.graph_objects as go
from datetime import datetime

st.set_page_config(page_title="SCHD ETF Dashboard", layout="wide", page_icon="💹")

tickers = ['SCHD','PEP','KO','MRK','AMGN','CSCO','TXN','JNJ','PFE',
           'HD','IBM','AVGO','T','XOM','CVX','PG','MCD','VZ','RTX','UPS']

st.title("💹 SCHD ETF Real-Time Portfolio Dashboard")
st.caption("실시간 시세 · 뉴스 · 실적 일정 통합 플랫폼")

# ETF 섹션
with st.container():
    schd = yf.Ticker('SCHD')
    price = schd.history(period='1d', interval='1m')['Close'][-1]
    st.metric("SCHD 실시간 가격", f"${price:.2f}")

    hist = schd.history(period='6mo')
    fig = go.Figure(data=[go.Candlestick(x=hist.index,
                open=hist['Open'], high=hist['High'], 
                low=hist['Low'], close=hist['Close'])])
    fig.update_layout(title="최근 6개월 가격 추이 (SCHD)", template="plotly_dark")
    st.plotly_chart(fig, use_container_width=True)

# 구성 종목 시세
st.header("상위 20개 구성 종목 실시간 데이터")
rows = []
for t in tickers:
    ticker = yf.Ticker(t)
    info = ticker.info
    rows.append({
        '종목': t,
        '이름': info.get('shortName', ''),
        '현재가': info.get('currentPrice', ''),
        '배당수익률': info.get('dividendYield', ''),
        'PER': info.get('trailingPE', '')
    })
df = pd.DataFrame(rows)
st.dataframe(df)

# 뉴스 및 실적 일정
col1, col2 = st.columns([2,1])
with col1:
    st.subheader("📢 최신 뉴스")
    for news in yf.Ticker('SCHD').news[:5]:
        t = datetime.utcfromtimestamp(news['providerPublishTime']).strftime('%Y-%m-%d')
        st.markdown(f"**{news['title']}** ({t})")
        st.caption(news['publisher'])
        st.write(news['link'])

with col2:
    st.subheader("📅 실적 일정")
    for t in tickers[:10]:
        ticker = yf.Ticker(t)
        cal = ticker.calendar
        if not cal.empty:
            next_date = cal.iloc[0,0].strftime('%Y-%m-%d')
            st.write(f"• {t}: {next_date}")
