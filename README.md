**O projeto foi desenvolvido por Ariadne Evangelista; Arthur Queiroz; Luisa Mathias e Viviane Pinheiro** **DCA3702 – Algoritmos e Estruturas de Dados II**

# Trabalho Final: Análise de Redes Complexas (Wikipedia)
**Construção, Análise Topológica e Visualização de Grafos de Conhecimento**

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/63/Wikipedia-logo.png" alt="Capa" width="20%"><br>
</p>

---

## 1. Introdução

<p align = "justify">Este trabalho aplica conceitos de Análise de Redes Complexas (CNA) para compreender a topologia do conhecimento na Wikipedia. [cite_start]O objetivo central é construir e analisar um grafo onde os nós representam páginas (artigos) e as arestas representam os hiperlinks entre elas, partindo de um conjunto de tópicos iniciais ("Sementes") [cite: 978-981].</p>

<p align = "justify">O estudo aborda o desafio do crescimento exponencial de redes de informação. Para isso, foi implementado um <i>crawler</i> (robô de busca) capaz de explorar a rede até o nível 2 de profundidade, utilizando uma heurística de amostragem para garantir a viabilidade computacional. [cite_start]A partir da rede construída, foram calculadas métricas fundamentais de centralidade (Grau, Intermediação, Proximidade e Autovetor) e realizada a decomposição estrutural (K-Core) [cite: 983-987].</p>

<p align = "justify">A análise final integra a visualização avançada utilizando o software Gephi, permitindo a identificação de "hubs" de informação, pontes entre disciplinas e a formação de comunidades temáticas. [cite_start]Este projeto consolida o uso de bibliotecas Python (NetworkX, Wikipedia-API) com ferramentas de visualização para revelar a estrutura subjacente da maior enciclopédia livre do mundo [cite: 988-991].</p>

## 2. Metodologia

<p align = "justify">A base de dados foi construída dinamicamente através da API da Wikipedia. [cite_start]Para garantir a diversidade topológica e temática, foram selecionadas cinco "sementes" (seeds) de domínios distintos: <i>Quantum Physics</i> (Ciência), <i>Renaissance Art</i> (História/Arte), <i>Climate Change</i> (Meio Ambiente/Política), <i>World War II</i> (História Global) e <i>Artificial Intelligence</i> (Tecnologia) [cite: 1037-1051, 1421-1432].</p>

<p align = "justify">A metodologia de coleta e análise consistiu em quatro etapas principais:</p>

* **Amostragem Aleatória Limitada (Bounded Random Sampling):** Para evitar a explosão combinatória (onde 1 semente pode levar a 1 milhão de links em poucos níveis), definiu-se um limite rígido de 50 arestas por página. [cite_start]Se uma página possui mais links que o limite, uma amostra aleatória é selecionada [cite: 997-1002, 1033].
* [cite_start]**Filtragem de Ruído:** Implementação de uma lista de <i>Stop Words</i> para excluir páginas irrelevantes à estrutura de conhecimento, como datas, identificadores (ISBN, DOI, PMID) e páginas administrativas da Wikipedia [cite: 1000, 1058-1068].
* [cite_start]**Fusão e Limpeza:** Pré-processamento para remover <i>self-loops</i> e fundir nós duplicados (singular/plural, ex: "Network" e "Networks") utilizando heurísticas de string [cite: 1253-1255].
* [cite_start]**Cálculo de Métricas:** Aplicação de algoritmos para determinar a importância dos nós (Degree, Betweenness, Closeness, Eigenvector Centrality) e a hierarquia da rede (K-Core Decomposition) [cite: 1350-1361, 1669].

## 3. Configuração e Pré-processamento

<p align = "justify">O ambiente de desenvolvimento utilizou Python com as bibliotecas `networkx` para modelagem de grafos, `wikipedia` para extração de dados e `matplotlib` para plotagens básicas. A configuração inicial definiu os parâmetros da heurística e as sementes de busca.</p>

```python
import networkx as nx
import wikipedia
import random

# Configuração da API
wikipedia.set_lang("en")

# PARÂMETROS DA HEURÍSTICA
MAX_LINKS_PER_PAGE = 50 

# Sementes (Seeds) de assuntos diferentes
SEEDS = [
    "Quantum Physics",      # Ciência (Física)
    "Renaissance Art",      # Arte e História
    "Climate Change",       # Meio Ambiente e Política
    "World War II",         # História Global
    "Artificial intelligence" # Tecnologia
]

# Stop Words (Filtro de Ruído)
STOPS = set([
    "International Standard Serial Number", "International Standard Book Number",
    "National Diet Library", "International Standard Name Identifier",
    "Pubmed Identifier", "Digital Object Identifier", "Arxiv", "Bibcode"
])
