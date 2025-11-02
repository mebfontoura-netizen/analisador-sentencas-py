import streamlit as st
import pandas as pd
import numpy as np
import re
import matplotlib.pyplot as plt

# --- 1. Módulo de Extração de Dados (Simulado) ---
@st.cache_data
def carregar_dados_simulados():
    """
    Simula a extração de dados reais de jurisprudência.
    Retorna um DataFrame do Pandas com a estrutura definida.
    """
    st.info("Simulando a extração de dados de jurisprudência (Web Scraping/API Conceitual)...")

    # Geração de dados simulados para 100 decisões
    num_decisoes = 100
    
    # Lista de termos jurídicos para compor as ementas
    termos_juridicos = [
        "dano moral", "repercussão geral", "inconstitucionalidade", 
        "habeas corpus", "recurso extraordinário", "competência", 
        "direito do consumidor", "prescrição", "coisa julgada", 
        "liberdade de expressão", "imposto de renda", "tributário"
    ]
    
    # Geração de ementas aleatórias
    ementas = []
    for i in range(num_decisoes):
        # Cada ementa terá entre 3 e 6 termos jurídicos aleatórios
        num_termos = np.random.randint(3, 7)
        ementa_termos = np.random.choice(termos_juridicos, num_termos, replace=False)
        ementa = " ".join(ementa_termos)
        # Adiciona texto genérico para simular o corpo da ementa
        ementa += f". Decisão {i+1} analisada em profundidade."
        ementas.append(ementa)

    data = {
        'ID_Decisao': range(1, num_decisoes + 1),
        'Tribunal': np.random.choice(['STF', 'STJ'], num_decisoes),
        'Ementa': ementas,
        'Resultado': np.random.choice(['Procedente', 'Improcedente', 'Parcialmente Procedente'], num_decisoes)
    }
    
    df = pd.DataFrame(data)
    return df

# --- 2. Funções de Análise ---

def filtrar_dados(df, tribunal_filtro):
    """Filtra o DataFrame com base na seleção do Tribunal."""
    if tribunal_filtro == "AMBOS":
        return df
    return df[df['Tribunal'] == tribunal_filtro]

def analisar_frequencia_termos(df_filtrado, termos_chave):
    """Calcula a frequência de cada termo-chave nas ementas filtradas."""
    frequencias = {}
    # Garante que a busca seja case-insensitive e trate a pontuação
    termos_list = [t.strip().lower() for t in termos_chave.split(',') if t.strip()]
    
    for termo in termos_list:
        # Usa regex para buscar o termo como palavra inteira (word boundary \b)
        # O re.escape é importante caso o termo contenha caracteres especiais de regex
        padrao = r'\b' + re.escape(termo) + r'\b'
        
        # Conta as ocorrências do termo em todas as ementas
        contagem = df_filtrado['Ementa'].str.lower().str.count(padrao).sum()
        frequencias[termo] = contagem
        
    # Cria um DataFrame para o relatório
    df_frequencia = pd.DataFrame(list(frequencias.items()), columns=['Termo-Chave', 'Frequência'])
    df_frequencia = df_frequencia.sort_values(by='Frequência', ascending=False).reset_index(drop=True)
    return df_frequencia

# --- 3. Interface Streamlit ---

def main():
    st.set_page_config(
        page_title="Analisador de Sentenças STF/STJ",
        layout="wide",
        initial_sidebar_state="expanded"
    )
    
    st.title("⚖️ Analisador de Sentenças do STF/STJ")
    st.markdown("""
        **Ferramenta interativa para análise quantitativa de jurisprudência**, 
        focada na identificação de tendências e frequência de termos jurídicos em ementas de acórdãos.
    """)

    # Carrega os dados simulados
    df_dados = carregar_dados_simulados()

    # --- Sidebar para Entradas do Usuário ---
    st.sidebar.header("Configurações de Análise")

    # 1. Filtro de Tribunal
    tribunal_filtro = st.sidebar.radio(
        "Filtro de Tribunal:",
        ('AMBOS', 'STF', 'STJ'),
        index=0,
        help="Selecione o tribunal cujas decisões serão analisadas."
    )

    # 2. Termos-Chave para Busca
    termos_chave = st.sidebar.text_area(
        "Termos-Chave para Busca (separados por vírgula):",
        "dano moral, inconstitucionalidade, repercussão geral",
        height=100,
        help="Insira os termos jurídicos de interesse. A busca é case-insensitive."
    )

    # 3. Botão de Análise
    if st.sidebar.button("Analisar Decisões", type="primary"):
        if not termos_chave.strip():
            st.error("Por favor, insira pelo menos um termo-chave para iniciar a análise.")
            return

        # Processamento
        with st.spinner("Processando a análise..."):
            # 3.1. Filtragem
            df_filtrado = filtrar_dados(df_dados, tribunal_filtro)
            
            # 3.2. Análise de Frequência
            df_frequencia = analisar_frequencia_termos(df_filtrado, termos_chave)
            
            # 3.3. Filtrar ementas que contêm pelo menos um dos termos para a Amostra
            termos_list = [t.strip().lower() for t in termos_chave.split(',') if t.strip()]
            
            # Cria uma expressão regex para buscar qualquer um dos termos
            # O '(?=.*termo1)(?=.*termo2)' é para AND, mas o requisito é apenas que a ementa
            # contenha o termo, então vamos usar um OR simples.
            padrao_or = '|'.join([r'\b' + re.escape(t) + r'\b' for t in termos_list])
            
            # Filtra as ementas que contêm pelo menos um dos termos
            df_amostra = df_filtrado[
                df_filtrado['Ementa'].str.lower().str.contains(padrao_or, regex=True, na=False)
            ].copy()
            
            st.success(f"Análise concluída! {len(df_amostra)} decisões encontradas com os termos-chave.")

        # --- 4. Exibição dos Resultados ---
        
        st.header("Resultados da Análise")
        
        col1, col2 = st.columns(2)

        # 4.1. Relatório de Frequência de Termos
        with col1:
            st.subheader("1. Frequência de Termos-Chave")
            st.dataframe(df_frequencia, use_container_width=True)
            
        # 4.2. Gráfico de Distribuição de Resultados
        with col2:
            st.subheader("2. Distribuição de Resultados (Procedente/Improcedente)")
            
            # Cria o gráfico de barras
            fig_resultado, ax_resultado = plt.subplots()
            contagem_resultado = df_amostra['Resultado'].value_counts()
            
            # Cores personalizadas para melhor visualização
            cores = {'Procedente': 'green', 'Improcedente': 'red', 'Parcialmente Procedente': 'orange'}
            cores_plot = [cores.get(r, 'gray') for r in contagem_resultado.index]
            
            contagem_resultado.plot(kind='bar', ax=ax_resultado, color=cores_plot)
            ax_resultado.set_title('Distribuição dos Resultados de Julgamento')
            ax_resultado.set_ylabel('Número de Decisões')
            ax_resultado.tick_params(axis='x', rotation=0)
            
            st.pyplot(fig_resultado)

        # 4.3. Gráfico de Distribuição por Tribunal
        st.subheader("3. Distribuição de Decisões por Tribunal")
        
        # O gráfico de pizza só faz sentido se o filtro for "AMBOS"
        if tribunal_filtro == "AMBOS":
            fig_tribunal, ax_tribunal = plt.subplots()
            contagem_tribunal = df_amostra['Tribunal'].value_counts()
            
            ax_tribunal.pie(
                contagem_tribunal, 
                labels=contagem_tribunal.index, 
                autopct='%1.1f%%', 
                startangle=90,
                colors=['#1f77b4', '#ff7f0e'] # Cores padrão para STF/STJ
            )
            ax_tribunal.axis('equal') # Garante que o gráfico seja um círculo
            ax_tribunal.set_title('Percentual de Decisões Analisadas por Tribunal')
            st.pyplot(fig_tribunal)
        else:
            st.info(f"O gráfico de distribuição por tribunal não é exibido, pois o filtro atual é '{tribunal_filtro}'.")

        # 4.4. Amostra de Decisões Filtradas
        st.header("4. Amostra de Decisões Filtradas")
        st.caption(f"Exibindo as primeiras 10 ementas das {len(df_amostra)} decisões que contêm os termos-chave.")
        
        # Seleciona as colunas relevantes para a amostra
        df_amostra_display = df_amostra[['ID_Decisao', 'Tribunal', 'Resultado', 'Ementa']].head(10)
        st.dataframe(df_amostra_display, use_container_width=True)
        
    else:
        st.info("Aguardando a inserção dos termos-chave e o clique no botão 'Analisar Decisões' na barra lateral.")

if __name__ == "__main__":
    main()
