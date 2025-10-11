# Desafio Quant ESG — README

Este repositório contém um **pipeline quantitativo de investimento** com ênfase em **ESG** que:
1) coleta dados de mercado de ativos brasileiros e fatores macro/setoriais,  
2) constrói _features_ (técnicos e macro) para séries temporais,  
3) treina um classificador (**XGBoost**) com validação apropriada no tempo,  
4) **otimiza o limiar de decisão** (probabilidade) para transformar as saídas do modelo em sinais de compra/venda,  
5) **backtesta** duas estratégias de portfólio baseadas nos sinais do modelo e  
6) **compara** o resultado com _benchmarks_ (Ibovespa e um ETF ESG).

> Tudo está orquestrado no notebook `notebooks/Final_Project.ipynb`, com gráficos salvos na pasta `notebooks/`.

---

## Visão Geral

O notebook executa um estudo com um pequeno conjunto de ativos brasileiros (ex.: `ITUB4.SA`, `ELET3.SA`, `RADL3.SA`, `LREN3.SA`, `VALE3.SA`) e utiliza fatores externos, como:
- **`^BVSP`** (Ibovespa),
- **`EWZ`** (ETF Brasil em NYSE),
- **`BRL=X`** (câmbio USD/BRL),
- **`ESGB11.SA`** (ETF ESG Brasil).

Os dados são baixados do **Yahoo Finance** via `yfinance`. O objetivo é verificar como variáveis técnicas e macroeconômicas ajudam a **prever alta/queda** no próximo período e usar essa previsão para **alocar capital**.

---

## Arquitetura Lógica

1. **Setup & Dependências**
   - Instala e importa: `yfinance`, `pandas`, `numpy`, `scikit-learn`, `xgboost`, `ta`, `matplotlib`.
   - Ajustes de ambiente para garantir reprodutibilidade no Jupyter.

2. **Painel de Controle (Cenários e Parâmetros)**
   - Define **cenários de mercado** (ex.: *“Mercado Desafiador (Juros Altos)”* e *“Mercado de Alta Pós‑Pandemia”*).
   - Escolhe **tickers-alvo** e **fatores** a serem usados por ativo.
   - Parâmetros de janela temporal, _split_ treino/validação/teste e grades de busca de hiperparâmetros.

3. **Funções do Sistema**
   - **Coleta e preparação**: baixa cotações, sincroniza datas, trata _NaN_ e constrói o *dataset*.
   - **Engenharia de _features_** com `ta` (ex.: RSI, MACD, médias móveis, bandas etc.) e cruzas com fatores macro (`^BVSP`, `EWZ`, `BRL=X`, `ESGB11.SA`).
   - **Validação temporal** com `TimeSeriesSplit` (evita *look-ahead bias*).
   - **Treino e _tuning_** do **XGBoost** via `GridSearchCV`.
   - **Otimização do Limiar**: dado que o classificador retorna probabilidades, encontra-se o **limiar de confiança ótimo** (ex.: 0.63) que maximizou o retorno na validação.
   - **Backtest & Portfólio**: gera sinais e simula duas carteiras:
     - **Peso Igual**: capital dividido igualmente entre ativos sinalizados.
     - **Ponderado por Confiança**: peso proporcional à probabilidade prevista pelo modelo.
   - **Benchmarks**: compara contra
     - **Média dos Ativos** (carteira ingênua),
     - **Ibovespa** (`^BVSP`),
     - **ETF ESG Brasil** (`ESGB11.SA`).

4. **Execução Principal (Orquestrador)**
   - Roda o processo por **ativo × cenário**, salva **gráficos** e **métricas** (ex.: `classification_report`).
   - Consolida os resultados em **gráficos finais** de evolução do capital das estratégias vs. benchmarks.

5. **Métricas & Relatórios**
   - Exibe **importância de fatores** (gráfico `importancia_geral_fatores.png`).
   - Salva por ativo e cenário arquivos `resultado_<ATIVO>_<CENARIO>.png` com o desempenho ao longo do tempo.

---

## Fluxo de Execução (Resumo)

1. **Coleta** (`yfinance.download`) dos *tickers* de interesse e fatores.
2. **Construção de *features*** técnicas e junção com fatores macro.
3. **Splits temporais** → treino / validação / teste.
4. **Treino do XGBoost** com _grid search_ + *scaler* para _features_.
5. **Otimização do limiar** de probabilidade na validação (transforma probabilidade em SINAL).
6. **Backtest** nas janelas de teste:
   - Marca dias com compra (probabilidade ≥ limiar).
   - Calcula retornos diário/composto por estratégia (Peso Igual e Ponderado).
7. **Comparação** com benchmarks e **plotagem** dos gráficos.

---

## Saídas e Arquivos Gerados

Na pasta `notebooks/` você encontrará:
- `importancia_geral_fatores.png`: ranking global de importância das variáveis usadas pelo modelo.
- `resultado_<TICKER>_<CENARIO>.png`: evolução do capital para cada ativo/cenário.
  - Ex.: `resultado_VALE3.SA_Mercado_de_Alta_Pós-Pandemia.png`
  - Ex.: `resultado_ITUB4.SA_Mercado_Desafiador_(Juros_Altos).png`

Também são impressas no notebook:
- **Métricas de classificação** (precisão/recall/F1) por janela.
- **Relatórios consolidados** das carteiras vs. benchmarks (Ibov e ESG).

---

## Como Reproduzir

1. **Pré-requisitos**
   - Python 3.9+
   - Jupyter Notebook ou JupyterLab

2. **Instalação rápida das dependências** (opção fora do notebook):
   ```bash
   pip install yfinance pandas numpy scikit-learn xgboost ta matplotlib
   ```

3. **Abrir o notebook**
   ```bash
   jupyter lab notebooks/Final_Project.ipynb
   # ou
   jupyter notebook notebooks/Final_Project.ipynb
   ```

4. **Executar todas as células** na ordem.  
   - A **primeira célula** já faz `pip install` das dependências, caso necessário.
   - Os **gráficos** serão salvos automaticamente em `notebooks/`.

> **Observação:** o download de dados do Yahoo (`yfinance`) requer internet e pode falhar temporariamente. Se ocorrer, execute a célula novamente.

---

## Personalização

- **Ativos & Fatores**: edite o dicionário que mapeia cada `TICKER` para sua lista de **fatores** e **nomes** correspondentes.
- **Cenários**: ajuste os cenários no “Painel de Controle” (janelas de datas e rótulos).
- **Modelo**: altere a grade do `GridSearchCV` (ex.: `n_estimators`, `max_depth`, `learning_rate`) e os parâmetros do `TimeSeriesSplit`.
- **Indicadores**: incremente/ajuste os indicadores em `ta` (RSI, MACD, MFI, bandas etc.).
- **Regras de Portfólio**: mude o cálculo de pesos, o critério de entrada/saída ou a forma de agregação de sinais.

---

## Boas Práticas Embutidas

- **Validação no tempo** com `TimeSeriesSplit` (evita vazamento de informação).  
- **Otimização de limiar** em **validação**, não no teste (evita *overfitting* ao período de avaliação).  
- **Comparação com benchmarks reais** (Ibov e **ETF ESG**).  
- **Visualizações** claras para interpretação do desempenho e da importância de variáveis.

---

## Estrutura do Repositório

```
DesafioQuantESG-main/
└─ notebooks/
   ├─ Final_Project.ipynb
   ├─ importancia_geral_fatores.png
   ├─ resultado_<TICKER>_<CENARIO>.png
   └─ ... (demais imagens de resultados)
```

---


## Desenvolvedores

- Davi Celestino. Git: https://github.com/Davicsb
- João Gabriel. Git: https://github.com/JGSEIXAS
- Marcos Mendonça. Git: https://github.com/eumarcosmendonca
