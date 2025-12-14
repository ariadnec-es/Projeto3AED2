**O projeto foi desenvolvido por Ariadne Evangelista; Arthur Queiroz; Luisa Mathias e Viviane Pinheiro**

**DCA3702 – Algoritmos e Estruturas de Dados II**

# Trabalho Final: Análise de Redes Complexas (Wikipedia)
**Construção, Análise Topológica e Visualização de Grafos de Conhecimento**

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/63/Wikipedia-logo.png" alt="Wikipedia Logo" width="20%">
</p>

---

## 1. Introdução

Este trabalho aplica conceitos de **Análise de Redes Complexas (CNA)** para compreender a topologia do conhecimento na Wikipedia. O objetivo central é construir e analisar um grafo em que os **nós representam páginas (artigos)** e as **arestas representam os hiperlinks** entre elas, partindo de um conjunto de tópicos iniciais (*sementes*).

O estudo aborda o desafio do crescimento exponencial das redes de informação. Para isso, foi implementado um *crawler* capaz de explorar a rede até **nível 2 de profundidade**, utilizando uma **heurística de amostragem** para garantir viabilidade computacional.

A partir da rede construída, foram calculadas métricas fundamentais de centralidade (**Grau, Intermediação, Proximidade e Autovetor**) e realizada a **decomposição estrutural K-Core**.

A análise final integra visualização avançada com o software **Gephi**, permitindo identificar *hubs* de informação, pontes entre disciplinas e comunidades temáticas. O projeto consolida o uso das bibliotecas Python **NetworkX** e **Wikipedia-API**, revelando a estrutura subjacente da maior enciclopédia livre do mundo.

---

## 2. Metodologia

A base de dados foi construída dinamicamente por meio da **API da Wikipedia**. Para garantir diversidade topológica e temática, foram selecionadas cinco sementes de domínios distintos:

- **Quantum Physics** – Ciência  
- **Renaissance Art** – História/Arte  
- **Climate Change** – Meio Ambiente/Política  
- **World War II** – História Global  
- **Artificial Intelligence** – Tecnologia  

A metodologia de coleta e análise consistiu em quatro etapas principais:

- **Amostragem Aleatória Limitada (Bounded Random Sampling):**  
  Limite máximo de 50 links por página para evitar explosão combinatória.

- **Filtragem de Ruído:**  
  Exclusão de páginas irrelevantes (ISBN, DOI, PMID, páginas administrativas).

- **Fusão e Limpeza:**  
  Remoção de *self-loops* e fusão de nós duplicados (singular/plural).

- **Cálculo de Métricas:**  
  Degree, Betweenness, Closeness, Eigenvector e decomposição K-Core.

---

## 3. Configuração e Pré-processamento

O ambiente de desenvolvimento utilizou **Python**, com as bibliotecas:

- `networkx` – modelagem de grafos  
- `wikipedia` – extração de dados  
- `matplotlib` – visualizações básicas  

```python
import networkx as nx
import wikipedia
import random

# Configuração da API
wikipedia.set_lang("en")

# Parâmetros da heurística
MAX_LINKS_PER_PAGE = 50 

# Sementes
SEEDS = [
    "Quantum Physics",
    "Renaissance Art",
    "Climate Change",
    "World War II",
    "Artificial intelligence"
]

# Stop Words
STOPS = set([
    "International Standard Serial Number",
    "International Standard Book Number",
    "National Diet Library",
    "International Standard Name Identifier",
    "Pubmed Identifier",
    "Digital Object Identifier",
    "Arxiv",
    "Bibcode"
])
````

---

## 4. Coleta e Construção da Rede

A construção da rede seguiu um algoritmo de **Busca em Largura (BFS)**, respeitando o limite máximo de profundidade (nível < 3).

```python
while todo_lst:
    layer, page = todo_lst.pop(0)

    if layer >= 2:
        done_set.add(page)
        continue

    try:
        wiki = wikipedia.page(page)
        raw_links = wiki.links

        valid_links = [link for link in raw_links if link not in STOPS]

        if len(valid_links) > MAX_LINKS_PER_PAGE:
            sampled_links = random.sample(valid_links, MAX_LINKS_PER_PAGE)
        else:
            sampled_links = valid_links

        for link in sampled_links:
            g.add_edge(page, link)
            if link not in todo_set and link not in done_set:
                todo_lst.append((layer + 1, link))
                todo_set.add(link)
    except:
        continue
```

### Limpeza e Fusão de Duplicatas

```python
g.remove_edges_from(nx.selfloop_edges(g))

duplicates = []
nodes_to_check = list(g.nodes())

for node in nodes_to_check:
    if node.endswith('s') and len(node) > 1:
        singular = node[:-1]
        if g.has_node(singular):
            duplicates.append((singular, node))

for u, v in duplicates:
    if g.has_node(u) and g.has_node(v):
        nx.contracted_nodes(g, u, v, self_loops=False)
```

---

## 5. Análise de Métricas e Resultados

Foram calculadas métricas de centralidade para identificar a relevância estrutural dos nós:

* **Degree Centrality**
* **Betweenness Centrality**
* **Closeness Centrality**
* **Eigenvector Centrality**
* **K-Core Decomposition**

```python
degree_dict = dict(g.degree())
nx.set_node_attributes(g, degree_dict, 'degree')

bet_cent = nx.betweenness_centrality(g, k=100)
nx.set_node_attributes(g, bet_cent, 'betweenness_centrality')

closeness_cent = nx.closeness_centrality(g)
nx.set_node_attributes(g, closeness_cent, 'closeness_centrality')

eigen_cent = nx.eigenvector_centrality(g)
nx.set_node_attributes(g, eigen_cent, 'eigenvector_centrality')

nx.write_graphml(g, "wikipedia_network_updated_metrics.graphml")
```

### 5.1 Destaques da Análise

O grafo final contém aproximadamente:

* **8.858 nós**
* **10.596 arestas**

Os resultados confirmam a estrutura de **Mundo Pequeno** da Wikipedia, com identificação clara de *hubs* globais e de comunidades temáticas.

### 5.2 Visualização (Gephi)

Recomenda-se o uso dos layouts:

* **Force Atlas 2**
* **Fruchterman-Reingold**

Os atributos exportados permitem análise visual de centralidade, modularidade, e K-Core.

---

## 6. Conclusão

A heurística de amostragem limitada mostrou-se eficaz para controlar o crescimento exponencial da rede, permitindo a construção de um grafo representativo e computacionalmente viável.

As métricas de **Betweenness** e **Eigenvector Centrality** foram essenciais para identificar nós estruturalmente relevantes. A integração com o **Gephi** validou visualmente a formação de comunidades e interconexões entre áreas distintas do conhecimento.

---

## 7. Materiais Adicionais

* **Notebook Colab:** [Acesse o código aqui!](https://colab.research.google.com/drive/19uU41FSju0OfrUMH-m1co90nEPOsd1aT)

* **Vídeo de Apresentação:**  *(link a ser)*

---
