# Geoestatística — Atividade Final (Helber Soares)

<a href="https://colab.research.google.com/github/helberjf/Helber/blob/main/Geoestastica_Atividade_Final_Helber_Soares.ipynb" >
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Abrir no Google Colab"/>
</a>

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)](https://jupyter.org/)
[![Status](https://img.shields.io/badge/Status-Em%20desenvolvimento-yellow)](#)
[![License](https://img.shields.io/badge/Licença-[INSERIR]-lightgrey)](#licenca)

## Visão geral

Este repositório reúne a atividade final de Geoestatística desenvolvida em Python por Helber Soares, com foco em:
- Análise exploratória de dados espaciais (EDA)
- Construção e ajuste de semivariogramas
- Modelagem variográfica (esférico, exponencial, gaussiano, etc.)
- Interpolação por krigagem (ex.: krigagem ordinária)
- Validação (ex.: validação cruzada) e avaliação de desempenho
- Geração de mapas temáticos e superfícies interpoladas

O projeto foi estruturado para ser facilmente executado tanto no Google Colab (sem instalação local) quanto em ambiente local (conda/pip). O notebook principal contém toda a lógica, código e visualizações.

## Conteúdo principal

- Notebook principal:
  - Geoestastica_Atividade_Final_Helber_Soares.ipynb

- Principais etapas no notebook:
  1. Importação e limpeza dos dados
  2. Análise exploratória (estatísticas descritivas e visualizações)
  3. Cálculo do semivariograma experimental
  4. Ajuste de modelos variográficos
  5. Krigagem e geração de superfícies
  6. Validação e métricas
  7. Exportação de resultados (mapas, figuras e/ou camadas geoespaciais)

> Observação: Se esta atividade depende de um conjunto de dados específico, consulte a seção “Dados” para organizar e apontar corretamente os caminhos.

## Abertura rápida no Google Colab

A forma mais simples de executar é abrir o notebook diretamente no Colab (nenhuma instalação local é necessária):

<a href="https://colab.research.google.com/github/helberjf/Helber/blob/main/Geoestastica_Atividade_Final_Helber_Soares.ipynb" >
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Abrir no Google Colab"/>
</a>

No Colab:
1. Conecte o ambiente (Runtime > Connect).
2. Se o notebook pedir bibliotecas específicas, rode as células de instalação (pip) no topo do notebook.
3. Caso utilize dados locais, suba os arquivos pelo painel lateral ou monte o Google Drive.

## Requisitos

Dependências típicas para fluxos de geoestatística e geoprocessamento (ajuste conforme seu notebook):

- Python 3.9+
- NumPy, Pandas, SciPy
- Matplotlib, Seaborn
- GeoPandas, Shapely
- Rasterio
- scikit-gstat (skgstat) ou GSTools
- PyKrige
- scikit-learn (para métricas/validação)
- Contextily (mapas base) [opcional]
- Plotly [opcional, para visualizações interativas]
- Jupyter

Exemplos de instalação (escolha UMA das abordagens abaixo):

### Opção A: Ambiente conda (recomendado para geoprocessamento)
```bash
conda create -n geoestatistica python=3.10 -y
conda activate geoestatistica
conda install -c conda-forge numpy pandas scipy matplotlib seaborn geopandas shapely rasterio contextily -y
pip install scikit-gstat pykrige scikit-learn plotly
