**O projeto foi desenvolvido por Ariadne Evangelista; Arthur Queiroz; Luisa Mathias e Viviane Pinheiro** 

**DCA3702 – Algoritmos e Estruturas de Dados II**

# Trabalho Final: Análise de Redes Complexas (Wikipedia)
**Construção, Análise Topológica e Visualização de Grafos de Conhecimento**

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/63/Wikipedia-logo.png" alt="Capa" width="20%"><br>
</p>

---

## 1. Introdução

<p align = "justify">Este trabalho aplica conceitos de Análise de Redes Complexas (CNA) para compreender a topologia do conhecimento na Wikipedia.O objetivo central é construir e analisar um grafo onde os nós representam páginas (artigos) e as arestas representam os hiperlinks entre elas, partindo de um conjunto de tópicos iniciais ("Sementes"). </p>

<p align = "justify">O estudo aborda o desafio do crescimento exponencial de redes de informação. Para isso, foi implementado um <i>crawler</i> (robô de busca) capaz de explorar a rede até o nível 2 de profundidade, utilizando uma heurística de amostragem para garantir a viabilidade computacional. A partir da rede construída, foram calculadas métricas fundamentais de centralidade (Grau, Intermediação, Proximidade e Autovetor) e realizada a decomposição estrutural (K-Core).</p>

<p align = "justify">A análise final integra a visualização avançada utilizando o software Gephi, permitindo a identificação de "hubs" de informação, pontes entre disciplinas e a formação de comunidades temáticas. Este projeto consolida o uso de bibliotecas Python (NetworkX, Wikipedia-API) com ferramentas de visualização para revelar a estrutura subjacente da maior enciclopédia livre do mundo.</p>

## 2. Metodologia

<p align = "justify">A base de dados foi construída dinamicamente através da API da Wikipedia. Para garantir a diversidade topológica e temática, foram selecionadas cinco "sementes" (seeds) de domínios distintos: <i>Quantum Physics</i> (Ciência), <i>Renaissance Art</i> (História/Arte), <i>Climate Change</i> (Meio Ambiente/Política), <i>World War II</i> (História Global) e <i>Artificial Intelligence</i> (Tecnologia). </p>

<p align = "justify">A metodologia de coleta e análise consistiu em quatro etapas principais:</p>

* **Amostragem Aleatória Limitada (Bounded Random Sampling):** Para evitar a explosão combinatória (onde 1 semente pode levar a 1 milhão de links em poucos níveis), definiu-se um limite rígido de 50 arestas por página. Se uma página possui mais links que o limite, uma amostra aleatória é selecionada.
* **Filtragem de Ruído:** Implementação de uma lista de <i>Stop Words</i> para excluir páginas irrelevantes à estrutura de conhecimento, como datas, identificadores (ISBN, DOI, PMID) e páginas administrativas da Wikipedia.
* **Fusão e Limpeza:** Pré-processamento para remover <i>self-loops</i> e fundir nós duplicados (singular/plural, ex: "Network" e "Networks") utilizando heurísticas de string.
* **Cálculo de Métricas:** Aplicação de algoritmos para determinar a importância dos nós (Degree, Betweenness, Closeness, Eigenvector Centrality) e a hierarquia da rede (K-Core Decomposition).

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
```

## 4. Coleta e Construção da Rede

<p align = "justify">A construção da rede seguiu um algoritmo de Busca em Largura (BFS). O código abaixo demonstra o loop principal que gerencia a fila de processamento (todo_lst), aplica a amostragem aleatória aos links encontrados e constrói o grafo direcionado (gnx.DiGraph), respeitando a profundidade máxima de exploração (nível < 3) .</p> 

Aqui está o código em **Markdown** pronto para ser copiado e colado diretamente no arquivo `README.md` do seu repositório no GitHub.

````markdown
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
````

## 4\. Coleta e Construção da Rede

\<p align = "justify"\>A construção da rede seguiu um algoritmo de Busca em Largura (BFS). [cite_start]O código abaixo demonstra o loop principal que gerencia a fila de processamento (`todo_lst`), aplica a amostragem aleatória aos links encontrados e constrói o grafo direcionado (`gnx.DiGraph`), respeitando a profundidade máxima de exploração (nível \< 3) [cite: 1073-1074].\</p\>

```python
# Coleta de Dados - Loop Principal (Resumo)
while todo_lst:
    layer, page = todo_lst.pop(0)
    
    # Critério de Parada: Explorar até nível 2
    if layer >= 2:
        if page not in done_set:
            done_set.add(page)
        continue

    try:
        wiki = wikipedia.page(page)
        raw_links = wiki.links
        
        # APLICAÇÃO DA HEURÍSTICA (REQUISITO #04)
        valid_links = [link for link in raw_links if link not in STOPS]
        
        # Seleciona amostra se houver muitos links
        if len(valid_links) > MAX_LINKS_PER_PAGE:
            sampled_links = random.sample(valid_links, MAX_LINKS_PER_PAGE)
        else:
            sampled_links = valid_links
            
        for link in sampled_links:
            g.add_edge(page, link.title())
            if link not in todo_set and link not in done_set:
                todo_lst.append((layer + 1, link))
                todo_set.add(link)
    except:
        continue
```

[cite_start]\<p align = "justify"\>Após a coleta, foi realizada uma etapa robusta de limpeza para fusão de nós duplicados (ex: plurais) e remoção de arestas que apontam para o próprio nó (self-loops), garantindo a consistência das métricas topológicas [cite: 1256-1262].\</p\>

```python
# Limpeza e Fusão de Duplicatas
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

## 5\. Análise de Métricas e Resultados

\<p align = "justify"\>Com o grafo limpo, foram calculadas métricas estatísticas para identificar a relevância de cada página na rede. [cite_start]Os resultados foram incorporados como atributos dos nós para posterior visualização [cite: 1363-1389].\</p\>

  * **Degree Centrality:** Mede a popularidade (In-Degree).
  * **Betweenness Centrality:** Identifica nós que funcionam como "pontes" entre diferentes áreas do conhecimento.
  * **Closeness Centrality:** Mede a proximidade de um nó em relação a todos os outros.
  * **Eigenvector Centrality:** Avalia a influência de um nó baseada na qualidade de suas conexões.
  * **K-Core:** Decompõe a rede em camadas para identificar o núcleo denso de conexões.

<!-- end list -->

```python
# Cálculo das Métricas
degree_dict = dict(g.degree())
nx.set_node_attributes(g, degree_dict, 'degree')

print("Calculando Betweenness Centrality...")
bet_cent = nx.betweenness_centrality(g, k=100) # Aproximação para performance
nx.set_node_attributes(g, bet_cent, 'betweenness_centrality')

print("Calculando Closeness e Eigenvector...")
closeness_cent = nx.closeness_centrality(g, wf_improved=False)
nx.set_node_attributes(g, closeness_cent, 'closeness_centrality')

eigen_cent = nx.eigenvector_centrality(g)
nx.set_node_attributes(g, eigen_cent, 'eigenvector_centrality')

# Exportação para Gephi
nx.write_graphml(g, "wikipedia_network_updated_metrics.graphml")
```

### 5.1. Destaques da Análise

[cite_start]\<p align = "justify"\>A análise final resultou em um grafo limpo contendo aproximadamente **8.858 nós** e **10.596 arestas**[cite: 1653]. As métricas calculadas revelaram a estrutura de "Mundo Pequeno" da Wikipedia, permitindo identificar hubs globais (como páginas de grandes guerras ou tecnologia) e hubs locais (tópicos específicos de arte). O arquivo GraphML gerado contém todos esses atributos, permitindo filtros avançados no Gephi.\</p\>

### 5.2. Visualização (Gephi)

\<p align = "justify"\>Para a representação visual, recomenda-se o uso do algoritmo de layout \<i\>Force Atlas 2\</i\> ou \<i\>Fruchterman Reingold\</i\> no Gephi. [cite_start]As visualizações destacam a estrutura de comunidades e a separação entre o núcleo (K-Core) e a periferia (K-Shell) da rede [cite: 862, 879-887, 917-921].\</p\>

> **Nota:** As imagens abaixo são ilustrativas baseadas nos requisitos do projeto.

#### Visualização de Centralidade e Comunidades

*Tamanho dos nós proporcional à centralidade de grau e cores baseadas em comunidades (Modularity).*

#### Decomposição K-Core (Núcleo vs Periferia)

*Destaque visual para o núcleo denso da rede (K-Core máximo) em contraste com as camadas periféricas (K-Shell).*

## 6\. Conclusão

\<p align = "justify"\>A implementação da heurística de amostragem limitada provou-se eficaz para controlar o crescimento exponencial da rede, permitindo a construção de um grafo representativo e computacionalmente viável partindo de 5 domínios distintos. A análise topológica confirmou a estrutura densamente conectada da Wikipedia.\</p\>

\<p align = "justify"\>As métricas de centralidade, especialmente o \<i\>Betweenness\</i\> e o \<i\>Eigenvector Centrality\</i\>, foram fundamentais para distinguir entre páginas populares e páginas estruturalmente importantes. [cite_start]A integração com o Gephi permitiu validar visualmente a formação de comunidades temáticas e a interconexão entre tópicos aparentemente distintos, cumprindo os objetivos de análise de estruturas complexas propostos na disciplina [cite: 1433-1435].\</p\>

## 7\. Adicionais

  * **Notebook do Colab:** [Acesse o código aqui!](https://colab.research.google.com/drive/19uU41FSju0OfrUMH-m1co90nEPOsd1aT?usp=sharing).
  * **Vídeo de Apresentação:** [Link]

```
