**O projeto foi desenvolvido por Ariadne Evangelista; Arthur Queiroz; Luisa Mathias e Viviane Pinheiro**

**DCA3702 – Algoritmos e Estruturas de Dados II**

# Trabalho Final: Análise de Redes Complexas (Wikipedia)
**Construção, Análise Topológica e Visualização de Grafos de Conhecimento**

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/63/Wikipedia-logo.png" alt="Wikipedia Logo" width="20%">
</p>

---

## 1. Introdução

Este trabalho aplica conceitos de **Análise de Redes Complexas (CNA)** para compreender a topologia do conhecimento na Wikipedia. O objetivo central é construir e analisar um grafo em que os **nós representam páginas (artigos)** e as **arestas os hiperlinks** entre elas, a partir de um conjunto de tópicos iniciais (*sementes*).

O estudo aborda o desafio do crescimento exponencial das redes de informação. Para isso, foi implementado um *crawler* (robô de busca) capaz de explorar a rede até **nível 2 de profundidade**, utilizando uma **heurística de amostragem** para garantir viabilidade computacional.

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
  Limite máximo de 50 links (`MAX_LINKS`) por página para evitar explosão combinatória.

- **Filtragem de Ruído:**  
  Exclusão de páginas irrelevantes (ISBN, DOI, PMID, páginas administrativas). Evitando o "efeito estrela", onde páginas administrativas aparecem falsamente como as mais importantes da rede apenas por serem citadas em todos os rodapés.

- **Fusão e Limpeza:**  
  Remoção de *self-loops* e fusão de nós duplicados (singular/plural).

- **Cálculo de Métricas:**
   Degree, Betweenness, Closeness, Eigenvector e decomposição K-Core.
  
- **Diversidade Temática:**
   O uso de sementes variadas garante que a rede não fique enviesada em um único assunto, permitindo a detecção de comunidades distintas.
---

## 3. Configuração e Pré-processamento

Definimos parâmetros iniciais para limitar a coleta de dados. Criamos uma lista de palavras de parada (*Stop Words*) e estabelecemos um limite máximo de links a serem seguidos por página (`MAX_LINKS_PER_PAGE = 50`). Escolhemos 5 sementes de domínios distintos (Ciência, Arte, Política, História, Tecnologia).

O ambiente de desenvolvimento utilizou **Python**, com as bibliotecas:

- `networkx` – modelagem de grafos  
- `wikipedia` – extração de dados  
- `matplotlib` – visualizações básicas  

```python
import networkx as nx
import wikipedia
import random

# Configuração da API para inglês
wikipedia.set_lang("en")

# PARÂMETROS DA HEURÍSTICA
# Limite rígido para evitar travamento por falta de memória
MAX_LINKS_PER_PAGE = 50 

# Sementes Selecionadas para maximizar diversidade
SEEDS = [
    "Quantum Physics", "Renaissance Art", "Climate Change", 
    "World War II", "Artificial intelligence"
]

# Stop Words: Páginas estruturais que não representam conhecimento
STOPS = set([
    "International Standard Serial Number", "International Standard Book Number",
    "National Diet Library", "International Standard Name Identifier",
    "Pubmed Identifier", "Digital Object Identifier", "Arxiv", "Bibcode"
])
````
---

## 4. Coleta e Construção da Rede

Implementamos um algoritmo de Busca em Largura (BFS - Breadth-First Search) que navega da camada 0 (sementes) até a camada 2 (vizinhos dos vizinhos). Dentro do loop, aplicamos uma lógica de seleção aleatória (random.sample) quando o número de links excede o limite estipulado.

 - A BFS é ideal para Snowball Sampling, garantindo que exploramos completamente a vizinhança imediata antes de aprofundar e para manter o escopo gerenciável (nível < 3).

 - Ao invés de pegar apenas os primeiros 50 links (que geralmente são alfabéticos ou de introdução), a amostragem aleatória preserva melhor a topologia global da rede, capturando conexões com tópicos variados dentro do artigo.

```python
# Inicialização do Grafo Direcionado
g = nx.DiGraph()
todo_lst = [(0, seed) for seed in SEEDS] # Fila de processamento
done_set = set() # Controle de visitados

while todo_lst:
    layer, page = todo_lst.pop(0) # Remove do início (Fila/BFS)
    
    # Critério de Parada: Não explorar além do nível 2
    if layer >= 2:
        if page not in done_set:
            done_set.add(page)
        continue

    try:
        wiki = wikipedia.page(page)
        raw_links = wiki.links
        
        # Filtragem inicial com Stop Words
        valid_links = [link for link in raw_links if link not in STOPS]
        
        # APLICAÇÃO DA HEURÍSTICA: Amostragem Aleatória
        if len(valid_links) > MAX_LINKS_PER_PAGE:
            sampled_links = random.sample(valid_links, MAX_LINKS_PER_PAGE)
        else:
            sampled_links = valid_links
            
        # Adição ao grafo e à fila
        for link in sampled_links:
            g.add_edge(page, link.title())
            if link not in todo_set and link not in done_set:
                todo_lst.append((layer + 1, link))
                todo_set.add(link)
    except:
        continue # Ignora erros de página não encontrada/ambígua
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

* **Notebook Colab:** [Acesse o código aqui!](https://colab.research.google.com/drive/19uU41FSju0OfrUMH-m1co90nEPOsd1aT?usp=sharing)

* **Vídeo de Apresentação:** [Acesse o vídeo aqui!]()
---
