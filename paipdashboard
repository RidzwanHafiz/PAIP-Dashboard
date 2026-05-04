import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from pathlib import Path

# =====================================
# PAGE CONFIG
# =====================================
st.set_page_config(
    page_title="Dashboard Pengurusan",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

# =====================================
# CUSTOM CSS
# =====================================
st.markdown(
    """
    <style>
    .main {
        background-color: #F5F7FB;
    }

    .stMetric {
        background-color: white;
        padding: 15px;
        border-radius: 12px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    }

    .dashboard-title {
        font-size: 40px;
        font-weight: 700;
        color: #123A7D;
    }

    .dashboard-subtitle {
        font-size: 18px;
        color: #555;
        margin-bottom: 20px;
    }

    .card {
        background-color: white;
        padding: 20px;
        border-radius: 15px;
        box-shadow: 0 2px 10px rgba(0,0,0,0.05);
    }

    </style>
    """,
    unsafe_allow_html=True
)

# =====================================
# LOAD DATA
# =====================================
@st.cache_data
def load_data():

    file_path = "Sale By District Tariff 2.xlsx"

    if not Path(file_path).exists():
        st.error("Excel file tidak dijumpai.")
        st.stop()

    df = pd.read_excel(file_path)

    df.columns = [str(col).strip() for col in df.columns]

    return df


df = load_data()

# =====================================
# AUTO DETECT COLUMNS
# =====================================
possible_region_cols = [
    c for c in df.columns
    if any(k in c.lower() for k in ["district", "daerah", "region"])
]

possible_tariff_cols = [
    c for c in df.columns
    if any(k in c.lower() for k in ["tariff", "kategori", "product"])
]

numeric_cols = df.select_dtypes(include='number').columns.tolist()

region_col = possible_region_cols[0] if possible_region_cols else df.columns[0]
tariff_col = possible_tariff_cols[0] if possible_tariff_cols else df.columns[1]
value_col = numeric_cols[0]

# =====================================
# SIDEBAR
# =====================================
with st.sidebar:

    st.image("https://cdn-icons-png.flaticon.com/512/3135/3135715.png", width=80)

    st.title("LAKU Management")

    st.markdown("---")

    years = ["All"]

    selected_year = st.selectbox("Tahun", years)

    selected_region = st.multiselect(
        "Daerah",
        sorted(df[region_col].dropna().unique()),
        default=sorted(df[region_col].dropna().unique())
    )

    selected_tariff = st.multiselect(
        "Tarif",
        sorted(df[tariff_col].dropna().unique()),
        default=sorted(df[tariff_col].dropna().unique())
    )

# =====================================
# FILTER DATA
# =====================================
filtered_df = df[
    (df[region_col].isin(selected_region)) &
    (df[tariff_col].isin(selected_tariff))
]

# =====================================
# TITLE
# =====================================
st.markdown(
    '<div class="dashboard-title">DASHBOARD PENGURUSAN</div>',
    unsafe_allow_html=True
)

st.markdown(
    '<div class="dashboard-subtitle">Sale By District Tariff</div>',
    unsafe_allow_html=True
)

# =====================================
# KPI SECTION
# =====================================

sales_total = pd.to_numeric(
    filtered_df[value_col],
    errors='coerce'
).sum()

avg_sales = pd.to_numeric(
    filtered_df[value_col],
    errors='coerce'
).mean()

max_region = (
    filtered_df.groupby(region_col)[value_col]
    .sum()
    .idxmax()
)

col1, col2, col3, col4 = st.columns(4)

col1.metric("Jumlah Jualan", f"RM {sales_total:,.2f}")
col2.metric("Purata Jualan", f"RM {avg_sales:,.2f}")
col3.metric("Jumlah Daerah", filtered_df[region_col].nunique())
col4.metric("Daerah Tertinggi", str(max_region))

st.markdown("---")

# =====================================
# CHARTS ROW 1
# =====================================

c1, c2 = st.columns((1.2, 1))

# =====================================
# BAR CHART
# =====================================
with c1:

    st.subheader("Jumlah Jualan Mengikut Daerah")

    region_summary = (
        filtered_df
        .groupby(region_col)[value_col]
        .sum()
        .reset_index()
        .sort_values(by=value_col, ascending=False)
    )

    fig_bar = px.bar(
        region_summary,
        x=value_col,
        y=region_col,
        orientation='h',
        color=value_col,
        text_auto='.2s'
    )

    fig_bar.update_layout(
        height=500,
        showlegend=False,
        plot_bgcolor='white',
        paper_bgcolor='white'
    )

    st.plotly_chart(fig_bar, use_container_width=True)

# =====================================
# DONUT CHART
# =====================================
with c2:

    st.subheader("Jumlah Jualan Mengikut Tarif")

    tariff_summary = (
        filtered_df
        .groupby(tariff_col)[value_col]
        .sum()
        .reset_index()
    )

    fig_pie = px.pie(
        tariff_summary,
        names=tariff_col,
        values=value_col,
        hole=0.55
    )

    fig_pie.update_layout(
        height=500,
        paper_bgcolor='white'
    )

    st.plotly_chart(fig_pie, use_container_width=True)

# =====================================
# TREND ANALYSIS
# =====================================
st.subheader("Trend Prestasi")

trend_df = region_summary.copy()
trend_df["Index"] = range(1, len(trend_df)+1)

fig_line = px.line(
    trend_df,
    x="Index",
    y=value_col,
    markers=True
)

fig_line.update_layout(
    height=400,
    xaxis_title="Trend",
    yaxis_title="Sales (RM)",
    plot_bgcolor='white',
    paper_bgcolor='white'
)

st.plotly_chart(fig_line, use_container_width=True)

# =====================================
# TABLE
# =====================================
st.subheader("Ringkasan Prestasi")

summary_table = (
    filtered_df
    .groupby([region_col, tariff_col])[value_col]
    .sum()
    .reset_index()
    .sort_values(by=value_col, ascending=False)
)

st.dataframe(
    summary_table,
    use_container_width=True,
    height=350
)

# =====================================
# INSIGHTS
# =====================================
st.subheader("Insight / Penemuan")

highest_sales = region_summary.iloc[0][region_col]
lowest_sales = region_summary.iloc[-1][region_col]

st.success(
    f"Daerah tertinggi ialah {highest_sales}"
)

st.warning(
    f"Daerah terendah ialah {lowest_sales}"
)

# =====================================
# DOWNLOAD
# =====================================
csv = summary_table.to_csv(index=False).encode('utf-8')

st.download_button(
    label="⬇ Download Data",
    data=csv,
    file_name="dashboard_export.csv",
    mime="text/csv"
)

# =====================================
# FOOTER
# =====================================
st.markdown("---")
st.caption("Dashboard Pengurusan - Streamlit + Plotly")
