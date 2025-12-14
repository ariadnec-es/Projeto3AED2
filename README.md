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
   Degree, Betweenness, Closeness e decomposição K-Core.
  
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
Realizamos o pós-processamento para remover self-loops (arestas de A para A) e fundir nós que representam o mesmo conceito semântico, mas com grafias ligeiramente diferentes (ex.: singular vs. plural).

 - Correção Semântica: O NetworkX trata "Network" e "Networks" como dois nós diferentes. Isso dilui a importância do conceito, dividindo suas conexões. A fusão (nx.contracted_nodes) soma as arestas, consolidando a importância no nó correto .

 - Integridade das Métricas: Self-loops não contribuem para o fluxo de informação entre tópicos e podem inflar artificialmente certas métricas de centralidade. Sua remoção resulta em um grafo mais limpo e focado na interconectividade.
   
```python
# 1. Remover Self-Loops
g.remove_edges_from(nx.selfloop_edges(g))

# 2. Identificar e Fundir Duplicatas (Plurais)
duplicates = []
nodes_to_check = list(g.nodes())

for node in nodes_to_check:
    # Lógica simples para detectar plurais em inglês ('s' no final)
    if node.endswith('s') and len(node) > 1:
        singular = node[:-1]
        if g.has_node(singular):
            duplicates.append((singular, node))

# Aplicar a contração dos nós
for u, v in duplicates:
    if g.has_node(u) and g.has_node(v):
        # Funde 'v' em 'u', somando suas arestas
        nx.contracted_nodes(g, u, v, self_loops=False)
```
---

## 5. Análise de Métricas e Resultados
Calculamos métricas fundamentais de redes complexas e as salvamos como atributos dos nós. Utilizamos aproximações (k=100) para algoritmos de alto custo computacional como o Betweenness.

Foram calculadas métricas de centralidade para identificar a relevância estrutural dos nós:

* **Degree Centrality** - Indica popularidade imediata
* **Betweenness Centrality** -  Revela "pontes" ou gargalos de informação. O parâmetro k=100 é usado porque calcular caminhos mínimos entre todos os pares de 8.000 nós é extremamente lento ($O(N^3)$ ou O(NM)); a amostragem acelera o processo com perda mínima de precisão.
* **Closeness Centrality** - Mede a eficiência de difusão. Nós com alto valor conseguem alcançar (ou serem alcançados por) toda a rede com o menor número de passos (cliques)
* **K-Core Decomposition** - Remove as "camadas de cebola" externas da rede para encontrar o núcleo denso e auto-sustentável de conhecimento .

```python
# Cálculo e Atribuição de Métricas

# Degree (Grau): Popularidade simples
degree_dict = dict(g.degree())
nx.set_node_attributes(g, degree_dict, 'degree')

# Betweenness (Intermediação): Importância como ponte
# k=100 usa apenas 100 nós pivô para estimar, acelerando o cálculo
bet_cent = nx.betweenness_centrality(g, k=100) 
nx.set_node_attributes(g, bet_cent, 'betweenness_centrality')

# Closeness (Proximidade): Velocidade de difusão
closeness_cent = nx.closeness_centrality(g, wf_improved=False)
nx.set_node_attributes(g, closeness_cent, 'closeness_centrality')

# Exportação final para visualização externa
nx.write_graphml(g, "wikipedia_network_updated_metrics.graphml")
```
---

## 6. Destaques da Análise

A análise da rede gerada revelou insights importantes sobre a topologia do conhecimento na Wikipedia. Abaixo detalhamos os principais achados quantitativos e qualitativos:

Após os processos de limpeza e fusão de nós duplicados, a rede final consolidou-se com **8.858 nós** e **10.596 arestas**.
    
* **Identificação de Hubs e Comunidades:**
    * **Hubs Globais:** Tópicos como *"World War II"* e *"Artificial Intelligence"* comportaram-se como grandes hubs, apresentando um alto grau de entrada (*In-Degree*). Isso reflete a natureza desses assuntos: eventos globais e tecnologias transversais que tocam diversas outras áreas (política, geografia, ética, engenharia).
      
    * **Comunidades Coesas:** O tópico *"Renaissance Art"* formou clusters mais densos e fechados. A análise visual e métrica indicou uma forte interconexão entre artistas, obras e locais geográficos específicos (como Itália e Flandres), criando uma "comunidade" temática bem definida dentro do grafo maior.

* A análise topológica confirmou que a Wikipedia possui características de redes de mundo pequeno (*Small-World Networks*). Apesar de cobrir domínios vastamente diferentes (Física Quântica a Arte Renascentista), o diâmetro da rede é relativamente curto, significando que é possível navegar de um tópico a outro com poucos cliques através dos hubs conectores.

### 6.1 Conclusão

Através da heurística de amostragem limitada, foi possível coletar uma rede representativa da Wikipedia partindo de 5 domínios distintos. A análise preliminar das métricas indica a presença de *hubs* conectores e uma estrutura de comunidades bem definida, que será explorada visualmente nas imagens geradas pelo Gephi e apresentadas no relatório final em vídeo.

---

## 7. Resultados e Discussão Visual

As visualizações a seguir foram geradas no software Gephi, utilizando o arquivo `.graphml` produzido na etapa anterior. As análises buscam validar a estrutura topológica da rede coletada e identificar padrões de conexão entre os diferentes temas.

### 7.1 Importância e Influência (Centralidade)

**Metodologia Visual:**
* **Layout:** Force Atlas 2 (para revelar aglomerados).
* **Tamanho dos Nós:** Proporcional ao *Degree* (Grau). Os nós maiores são mais populares.
* **Cor dos Nós:** Mapa de calor baseado na *Betweenness Centrality*. Cores quentes (Vermelho/Laranja) indicam alta intermediação; cores frias (Azul) indicam baixa intermediação.

**Interpretação Teórica:**
Em redes de informação como a Wikipedia, observamos frequentemente uma distribuição de lei de potência (*scale-free*), onde poucos nós possuem muitas conexões (Hubs) e a maioria possui poucas. Além disso, a métrica de *Betweenness* revela os "controladores de fluxo" da informação.

**Análise dos Resultados:**
Observando a Figura 1, destacam-se os seguintes pontos:

1.  **Hubs (Grandes):** Os nós de maior tamanho correspondem, como esperado, às sementes iniciais e aos termos generalistas. Identificamos **[Revolução Francesa e Existencialismo]** como um dos maiores hubs da rede.
2.  **Pontes:** Nosso gráfo não apresentou nós com tamanho médio e alto valor de betweenness, que serviriam de pontes entre os assuntos. Como podemos ver também, os Hubs são os que apresentam uma coloração mais avermelhada/alaranjada, indicando uma centralidade e dependências deles, portanto, você é obrigado a passar pelas páginas principais (os Hubs). Não existem "atalhos" ou caminhos alternativos através de páginas menores.
3.  **Hubs Locais (Grandes/Azuis):** Alguns nós são grandes, mas de cor fria. Isso indica que são muito importantes dentro de seu próprio tema, mas irrelevantes para a conectividade global da rede.

<p align="center">
  <img src="https://github.com/ariadnec-es/AnaliseRedesComplexas/blob/main/images/btweencentrality.png" alt="Centralidade" width="90%">
</p

### 7.2 Robustez e Hierarquia (K-Core Decomposition)

**Metodologia Visual:**
Utilizamos o filtro de *K-Core* para decompor a rede em camadas, similar a uma cebola.
* **K-Shells (Periferia/Azul):** Camadas externas com poucos links recíprocos.
* **K-Core Máximo (Núcleo/Vermelho):** O subgrafo mais denso da rede, onde todos os nós se conectam a pelo menos $k$ outros nós do mesmo grupo.

**Análise dos Resultados:**
A Figura 2 destaca o "núcleo duro" da rede (em destaque/vermelho).
Ao filtrar a rede, observamos que o núcleo é composto majoritariamente pelo tópico de **[Buraco Negro]**.

Isso demonstra que esse assunto possui uma base de conhecimento mais interconectada e recursiva. Em contraste, tópicos como **[Renascença e Existencialismo]** tendem a ficar na periferia (K-Shells baixos), pois citam muitos fatos, mas são pouco citados reciprocamente fora de seu contexto imediato.

<p align="center">
  <img src="https://github.com/ariadnec-es/AnaliseRedesComplexas/blob/main/images/kcoredecomp.png" alt="K-core" width="90%">
</p

## 8.3 Detecção de Comunidades (Modularidade)

**Metodologia Visual:**
Aplicamos o algoritmo de *Louvain* para cálculo de Modularidade.
* **Cores:** Cada cor distinta (Partition) representa uma classe de modularidade (Comunidade), ou seja, um grupo de páginas que se linkam mais entre si do que com o resto da rede.

**Análise dos Resultados:**
A Figura 3 revela uma formação com mais de 40 comunidades, portanto relvemos mostrar só as **[10]** maiores comunidades bem definidas. A segmentação visual valida a escolha das sementes iniciais, pois observamos clusters temáticos claros:

* **Comunidade [Amarelo]:** Agrupa tópicos relacionados a **[Revolução Francesa]**, mostrando ligações entre militares franceses como *Jean-de-Dieu Soult*, *Emmanuel de Grouchy*, entre outros. Assuntos políticos também estão no meio, como exemplo *Termidorianos* e *Descristianização da França*.
* **Comunidade [Lilás]:** Agrupa tópicos relacionados a **[Arte Renascentista]**, destacando movimentos artisticos como *Expressionismo Abstrato* e *Hyperrealism*. Além de escolas de arte, entre elas a *Hudson River School* e *Film Noir*.
* **Comunidade [Laranja]:** Agrupa tópicos relacionados a **[Inteligencia Artificial]**, conseguimos encontrar topicos relacionados a tecnincas de aprendizagem de máquina, por exemplo *Funções de Perda* e *Aprendizado por Reforço*, tópicos tambem sobre segurança e ética, indicando que a rede também mapeia as discussões sobre segurança e o futuro da tecnologia, como *Statement On AI Risk Of Extinction* e *Existential Risk From Artificial General Intelligence*.

As comunidades não possuem apenas conexões entre si, existem também nós que fazem parte de de conexões entre um ou mais grupos:

*   O nó **[Buraco Negro]** serve como ponte para os comunidades **[Rosa e Verde]**, que abordam assuntos de *Física teórica e Astronomia / Leis fundamentais da gravidade*, respectivamente.
*   O nó **[Existencialismo / Azul escuro], é um nó em comum para quatro comunidades diferentes, sendo as que mais se destacam [Azul/Verde Escuro]**, relacionados a assuntos de Filosofia da Mente, conectando movimentos e escolas filosóficas / Filosofia Aplicada, Ética e História das Ideias conectando conceitos morais, escolas históricas e campos interdisciplinares (como biologia e feminismo), respectivamente.

<p align="center">
  <img src="https://github.com/ariadnec-es/AnaliseRedesComplexas/blob/main/images/modularityclass.png" alt="Comunidades" width="90%">
</p

---

## 8. Materiais Adicionais

* **Notebook Colab:** [Acesse o código aqui!](https://colab.research.google.com/drive/19uU41FSju0OfrUMH-m1co90nEPOsd1aT?usp=sharing)

* **Vídeo de Apresentação:** [Acesse o vídeo aqui!]()

* As análises e visualizações interativas foram geradas pelo plugin **Sigma.js**, que permite explorar clusters individualmente. Disponível atravéz do link: [Wikipedia Network](https://arthurqueirozufrn.github.io/Teste-AED2/network/#)

---
