## Geoestatística de Roubos em São Paulo (DBSCAN, Mapas e Visualização)

Análise geoestatística de ocorrências de roubo na cidade de São Paulo usando clusterização DBSCAN, GeoPandas e visualização em mapas base. Inclui pipeline de limpeza, projeção adequada (CRS), descoberta de clusters, visualizações estáticas (Matplotlib/Contextily) e opções interativas (Folium).

[Abra no Google Colab]([(https://colab.research.google.com/drive/1RQpXc7spykVnBInO0egm5h9r64StSAIt)]) • [Demonstrações](#resultados-expectados) • [Como executar](#como-executar)

### Sumário
- Visão geral
- Principais recursos
- Estrutura do projeto
- Dados
- Metodologia
- Como executar
  - Google Colab
  - Ambiente local
- Parâmetros do DBSCAN e dicas de tuning
- Resultados esperados
- Limitações e próximos passos
- Dependências
- Contribuição
- Licença
- Agradecimentos

## Visão geral
Este projeto realiza uma análise espacial de ocorrências de roubo em São Paulo, buscando:
- Identificar áreas de concentração (hotspots) via clusterização baseada em densidade (DBSCAN);
- Visualizar os clusters sobre mapas base (CartoDB Positron, entre outros);
- Fornecer um pipeline transparente e reproduzível em Python com Jupyter Notebook.

Use casos:
- Apoio a estudos acadêmicos de geoestatística;
- Diagnóstico preliminar de padrões espaciais de criminalidade;
- Base para relatórios, dashboards ou análises mais aprofundadas (ex.: integração com séries temporais).

## Principais recursos
- Pré-processamento geoespacial com GeoPandas e Shapely;
- Projeção de coordenadas para uma métrica adequada à distância (CRS projetado, ex.: EPSG:3857);
- Clusterização com DBSCAN (scikit-learn), com parâmetros ajustáveis (eps, min_samples);
- Visualização estática com Matplotlib + Contextily (basemap CartoDB.PositronNoLabels);
- Visualização interativa opcional com Folium (marcadores e camadas por cluster);
- Exportação de resultados (CSV/GeoJSON) com o cluster de cada ponto.


## Dados
- Cobertura: Cidade de São Paulo (SP), Brasil.
- Conteúdo: pontos de ocorrência de roubo (latitude/longitude), datas e atributos auxiliares (quando disponíveis).
- Fonte: ajuste aqui para a sua fonte específica (ex.: SSP-SP, dados abertos da Prefeitura, dataset acadêmico).
- Formatos suportados: CSV, GeoJSON, Shapefile, Parquet (GeoParquet).

Coloque o arquivo de dados em data/raw e aponte o caminho no notebook.

Exemplo de layout mínimo esperado:
- latitude
- longitude
- data_ocorrencia (opcional)
- bairro/distrito (opcional)
- outros atributos (opcional)

Validação rápida:
- Remover linhas com coordenadas ausentes ou inválidas;
- Garantir que os pontos estão dentro da extensão geográfica de São Paulo;
- Deduplicar registros óbvios (se necessário).

## Metodologia

### 1) Pré-processamento
- Carregamento do CSV/GeoJSON em GeoDataFrame (GeoPandas);
- Conversão para geometria de pontos (Point) a partir de latitude/longitude;
- Remoção de outliers geográficos e coordenadas nulas;
- Padronização de colunas e tipos.

Exemplo (trecho genérico):
```python
import geopandas as gpd
from shapely.geometry import Point

df = ...  # carregue seu CSV com pandas
gdf = gpd.GeoDataFrame(
    df.dropna(subset=["latitude", "longitude"]),
    geometry=[Point(xy) for xy in zip(df["longitude"], df["latitude"])],
    crs="EPSG:4326"  # WGS84
)
```

### 2) Projeção e CRS
Para usar DBSCAN com eps em metros, projete para um CRS em metros (ex.: Web Mercator EPSG:3857, ou um CRS local apropriado).

```python
gdf_3857 = gdf.to_crs(epsg=3857)
```

Se preferir usar distância geodésica (haversine), mantenha EPSG:4326 e ajuste o DBSCAN para métrica "haversine" com coordenadas em radianos.

### 3) Clusterização com DBSCAN
DBSCAN encontra regiões de alta densidade. Principais parâmetros:
- eps: raio de vizinhança (em metros, se CRS projetado; em radianos, se haversine);
- min_samples: pontos mínimos para formar um núcleo de cluster.

Exemplo (métricas em metros com EPSG:3857):
```python
import numpy as np
from sklearn.cluster import DBSCAN

X = np.vstack([gdf_3857.geometry.x, gdf_3857.geometry.y]).T

db = DBSCAN(eps=200, min_samples=10, metric="euclidean", n_jobs=-1).fit(X)
gdf_3857["cluster"] = db.labels_
```

Após a rotulagem, você pode voltar para EPSG:4326 para exportar/visualizar em webmaps:
```python
gdf = gdf_3857.to_crs(epsg=4326)
```

### 4) Visualização
Visualização estática com Matplotlib e Contextily (basemap CartoDB.PositronNoLabels):

```python
import matplotlib.pyplot as plt
import contextily as ctx

fig, ax = plt.subplots(figsize=(10, 10))
gdf_3857.plot(
    ax=ax,
    column="cluster",
    categorical=True,
    legend=True,
    markersize=5,
    cmap="tab20",
    alpha=0.9
)
ctx.add_basemap(
    ax,
    source=ctx.providers.CartoDB.PositronNoLabels
)
ax.set_axis_off()
plt.tight_layout()
plt.show()
```

Interativo opcional com Folium:
```python
import folium

m = folium.Map(location=[-23.55, -46.63], zoom_start=11, tiles="CartoDB Positron")
for _, row in gdf.iterrows():
    c = "gray" if row["cluster"] == -1 else "red"
    folium.CircleMarker(
        location=[row.geometry.y, row.geometry.x],
        radius=3,
        color=c,
        fill=True,
        fill_opacity=0.7
    ).add_to(m)

m.save("outputs/figures/mapa_clusters.html")
```

## Como executar

### Opção A) Google Colab
1. Clique em “Abrir no Colab”: [link]((https://colab.research.google.com/drive/1RQpXc7spykVnBInO0egm5h9r64StSAIt)).
2. Monte seu Google Drive (se necessário) e ajuste os caminhos dos arquivos.
3. Instale as dependências no início do notebook:
   ```python
   !pip install -q geopandas shapely contextily folium scikit-learn matplotlib seaborn pyproj
   ```
4. Execute as células na ordem indicada.

Dica: se usar dados maiores, prefira salvar em Parquet/GeoParquet para leitura mais rápida.

### Opção B) Ambiente local (recomendado para reprodutibilidade)
1. Crie e ative um ambiente virtual:
   - Windows:
     ```
     python -m venv .venv
     .venv\Scripts\activate
     ```
   - macOS/Linux:
     ```
     python -m venv .venv
     source .venv/bin/activate
     ```
2. Instale as dependências:
   ```
   pip install -r requirements.txt
   ```
3. Abra o Jupyter:
   ```
   jupyter lab
   ```
   ou
   ```
   jupyter notebook
   ```
4. Abra o notebook em notebooks/Geoestatistica-Atividade-Final-Helber-Soares.ipynb, ajuste o caminho dos dados em data/raw e execute.

## Parâmetros do DBSCAN e dicas de tuning
- eps (métrico): depende da densidade urbana e da granularidade desejada.
  - Ponto de partida (área urbana densa): 150–300 metros.
  - Em áreas mais esparsas: 300–600 metros.
- min_samples: 5–20 costuma funcionar bem; valores maiores reduzem clusters pequenos.
- Se usar haversine:
  - Converta coords p/ radianos: X = np.radians(np.c_[lat, lon]).
  - eps em radianos ~ distância_m / raio_da_terra (6371000 m). Ex.: 200 m ≈ 200/6371000 ≈ 3.14e-5.
  - metric="haversine".
- Valide visualmente os clusters para evitar interpretar ruído (-1) como padrões relevantes.
- Considere estratificar por tempo (ex.: faixas de horário) se houver coluna temporal.

## Resultados esperados
- Mapa com pontos coloridos por cluster sobre fundo CartoDB Positron (sem rótulos);
- Exportação dos pontos com a coluna cluster:
  - outputs/exports/roubos_sp_clusters.geojson
  - outputs/exports/roubos_sp_clusters.csv
- Métricas simples:
  - Número de clusters encontrados (excluindo rótulo -1);
  - Proporção de pontos marcados como ruído.

Exemplos de figuras (ajuste os caminhos conforme seu output):
- outputs/figures/clusters_sp.png
- outputs/figures/mapa_clusters.html (interativo)

## Limitações e próximos passos
- DBSCAN é sensível a eps; uma escolha ruim pode juntar áreas distintas ou criar “ilhas” artificiais.
- Dados de roubo podem ter subnotificação ou imprecisões de geocodificação.
- Próximos passos:
  - Análises temporais (sazonalidade, hora/dia da semana);
  - Outras técnicas (HDBSCAN, KDE/Kernel Density Estimation, Moran’s I/Local Indicators of Spatial Association);
  - Normalização por população/área para taxas;
  - Junção espacial com malhas de setores censitários ou distritos.

## Dependências
Crie um requirements.txt como abaixo (ajuste versões conforme seu ambiente):

```
geopandas>=0.14
shapely>=2.0
pyproj>=3.6
contextily>=1.5
folium>=0.16
scikit-learn>=1.4
matplotlib>=3.8
seaborn>=0.13
pandas>=2.2
numpy>=1.26
jupyterlab>=4.0
```

Observações:
- Em alguns sistemas, a instalação de GeoPandas/Shapely pode exigir bibliotecas do sistema (GEOS, PROJ, GDAL). Em caso de dificuldade, usar conda-forge facilita:
  ```
  conda create -n geo python=3.11
  conda activate geo
  conda install -c conda-forge geopandas contextily scikit-learn folium matplotlib seaborn
  ```

## Contribuição
Contribuições são bem-vindas:
1. Abra uma issue descrevendo a sugestão ou problema;
2. Faça um fork, crie uma branch:
   ```
   git checkout -b feat/nome-da-feature
   ```
3. Envie um pull request com uma descrição clara e prints/resultados quando aplicável.

Sugestões de melhorias:
- Pipeline modular em src/ com funções reaproveitáveis;
- Testes unitários mínimos para validação de CRS e integridade de dados;
- Notebooks separados: 01_preprocessamento, 02_dbscan, 03_visualizacao.

## Licença
MIT License.

Resumo da MIT:
- Você pode usar, copiar, modificar, mesclar, publicar, distribuir, sublicenciar e/ou vender;
- Exige incluir o aviso de copyright e a licença;
- Sem garantias.

## Agradecimentos
- Professores e colegas da disciplina de Geoestatística;
- Comunidade de dados abertos de São Paulo;
- Bibliotecas open-source: GeoPandas, Shapely, scikit-learn, Contextily, Folium, Matplotlib, Pandas, NumPy.

## Metadados (opcional para o GitHub)
- Descrição curta: Análise geoestatística de roubos em São Paulo com DBSCAN e mapas base.
- Tópicos (tags): geoestatistics, geospatial, crime-data, sao-paulo, dbscan, clustering, geopandas, contextily, shapely, spatial-analysis, python, jupyter-notebook.

---
