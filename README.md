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

- **Black hole** – Ciência Exata/Física  
- **Renaissance Art** – História/Arte  
- **Artificial intelligence** – Tecnologia/Computação  
- **French Revolution** – Ciências Humanas/História 
- **Existentialism** – Filosofia/Literatura  

A metodologia de coleta e análise consistiu em quatro etapas principais:

- **Amostragem Aleatória Limitada (Bounded Random Sampling):**  
  Limite máximo de 50 links (`MAX_LINKS`) por página para evitar explosão combinatória.

- **Filtragem de Ruído:**  
  Exclusão de páginas irrelevantes (ISBN, DOI, PMID, páginas administrativas). Evitando o "efeito estrela", onde páginas administrativas aparecem falsamente como as mais importantes da rede apenas por serem citadas em todos os rodapés.

- **Fusão e Limpeza:**  
  Remoção de *self-loops* e fusão de nós duplicados (singular/plural).

- **Cálculo de Métricas:**
   Degree, Betweenness e decomposição K-Core.
  
- **Diversidade Temática:**
   O uso de sementes variadas garante que a rede não fique enviesada em um único assunto, permitindo a detecção de comunidades distintas.
---

## 3. Configuração e Pré-processamento

Definimos parâmetros iniciais para limitar a coleta de dados. Criamos uma lista de palavras de parada (*Stop Words*) e estabelecemos um limite máximo de links a serem seguidos por página (`MAX_LINKS_PER_PAGE = 50`). Escolhemos 5 sementes de domínios distintos (Ciência, Arte, Filosofia, História e Tecnologia).

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
    "Black hole",               # Ciência Exata/Física (Vermelho) 0
    "Renaissance Art",          # Arte/História/Cultura (Lilás) 2
    "Artificial intelligence",  # Tecnologia/Computação (Laranja) 3
    "French Revolution",        # Ciências Humanas/História (Amarelo) 5
    "Existentialism"            # Filosofia/Literatura (Azul escuro) 12
]

# Stop Words: Páginas estruturais que não representam conhecimento
STOPS = set([
    "International Standard Serial Number", "International Standard Book Number",
    "National Diet Library", "International Standard Name Identifier",
    "Pubmed Identifier", "Pubmed Central", "Digital Object Identifier",
    "Arxiv", "Proc Natl Acad Sci Usa", "Bibcode", "Jstor", "Doi (Identifier)",
    "Isbn (Identifier)", "Pmid (Identifier)", "Wayback Machine", "Help:Authority Control"
])
````
---

## 4. Coleta e Construção da Rede

Implementamos um algoritmo de Busca em Largura (BFS - Breadth-First Search) que navega da camada 0 (sementes) até a camada 2 (vizinhos dos vizinhos). Dentro do loop, aplicamos uma lógica de seleção aleatória (random.sample) quando o número de links excede o limite estipulado.

 - A BFS é ideal para Snowball Sampling, garantindo que exploramos completamente a vizinhança imediata antes de aprofundar e para manter o escopo gerenciável (nível < 3).

 - Ao invés de pegar apenas os primeiros 50 links (que geralmente são alfabéticos ou de introdução), a amostragem aleatória preserva melhor a topologia global da rede, capturando conexões com tópicos variados dentro do artigo.

```python
# Coleta de Dados
g = nx.DiGraph()
todo_lst = []
todo_set = set()
done_set = set()

# Inicializa a lista de tarefas com as 5 sementes (Layer 0)
for seed in SEEDS:
    try:
        # Tenta validar se a página existe antes de começar
        p = wikipedia.page(seed)
        title = p.title
        todo_lst.append((0, title))
        todo_set.add(title)
        print(f"Semente adicionada: {title}")
    except:
        print(f"Erro ao adicionar semente: {seed}")

print("--- Iniciando Coleta (Isso pode demorar alguns minutos) ---")

# Loop principal
while todo_lst:
    layer, page = todo_lst.pop(0) # Remove do início (BFS)

    # Critério de Parada: Explorar até nível 2 (altura < 3)
    # Se chegarmos no layer 2, não buscamos mais vizinhos, apenas adicionamos o nó.
    if layer >= 2:
        if page not in done_set:
            done_set.add(page)
        continue

    # Feedback visual para o usuário não achar que travou
    print(f"Processando [Nível {layer}]: {page}")

    try:
        wiki = wikipedia.page(page)
    except:
        # Se der erro (página ambígua ou inexistente), pula
        continue

    done_set.add(page)

    # Obtém links da página
    raw_links = wiki.links

    # --- APLICAÇÃO DA HEURÍSTICA ---
    # Filtra links indesejados primeiro
    valid_links = [link for link in raw_links if link not in STOPS and not link.startswith("List of")]

    # Seleciona uma amostra aleatória se houver muitos links
    if len(valid_links) > MAX_LINKS_PER_PAGE:
        sampled_links = random.sample(valid_links, MAX_LINKS_PER_PAGE)
    else:
        sampled_links = valid_links

    # Adiciona arestas ao grafo
    for link in sampled_links:
        link = link.title()

        # Adiciona aresta (Página Atual -> Link Encontrado)
        g.add_edge(page, link)

        # Adiciona à fila de processamento se ainda não foi visto
        if link not in todo_set and link not in done_set:
            todo_lst.append((layer + 1, link))
            todo_set.add(link)

print(f"\nColeta Finalizada!")
print(f"Total de Nós: {len(g.nodes())}")
print(f"Total de Arestas: {len(g.edges())}")
```

### Limpeza e Fusão de Duplicatas
Realizamos o pós-processamento para remover self-loops (arestas de A para A) e fundir nós que representam o mesmo conceito semântico, mas com grafias ligeiramente diferentes (ex.: singular vs. plural).

 - Correção Semântica: O NetworkX trata "Network" e "Networks" como dois nós diferentes. Isso dilui a importância do conceito, dividindo suas conexões. A fusão (nx.contracted_nodes) soma as arestas, consolidando a importância no nó correto .

 - Integridade das Métricas: Self-loops não contribuem para o fluxo de informação entre tópicos e podem inflar artificialmente certas métricas de centralidade. Sua remoção resulta em um grafo mais limpo e focado na interconectividade.
   
```python
# Limpeza
print("Limpando grafo...")

# 1. Remover Self-Loops (página apontando para ela mesma)
g.remove_edges_from(nx.selfloop_edges(g))

# 2. Fundir plurais (Ex: 'Network' e 'Networks')
duplicates = []
nodes_to_check = list(g.nodes())

for node in nodes_to_check:
    # Heurística simples para encontrar plurais (ex: 'Networks' -> 'Network')
    if node.endswith('s') and len(node) > 1:
        singular_form = node[:-1]
        if g.has_node(singular_form) and singular_form != node: # Evita adicionar o próprio nó
            duplicates.append((singular_form, node))
    # Verificar para terminações 'es' como em 'Heroes' -> 'Hero'
    elif node.endswith('es') and len(node) > 2 and node[-3] not in 'aeiou':
        singular_form = node[:-2]
        if g.has_node(singular_form) and singular_form != node:
            duplicates.append((singular_form, node))

# Adicionar uma verificação para garantir que os nós existam antes de tentar a contração
for u_node, v_node in duplicates:
    if g.has_node(u_node) and g.has_node(v_node):
        # Funde v_node em u_node
        g = nx.contracted_nodes(g, u_node, v_node, self_loops=False)
    else:
        print(f"Pulando contração: Um ou ambos os nós ('{u_node}', '{v_node}') não encontrados no grafo.")

# Remover atributo de contração que o networkx cria
    if 'contraction' in g.nodes[n]:
        del g.nodes[n]['contraction']

edges_to_clean = []
for u, v, data in g.edges(data=True):
    attrs_to_remove = []
    for key, value in data.items():
        if isinstance(value, dict):
            
            attrs_to_remove.append(key)
    if attrs_to_remove:
        edges_to_clean.append((u, v, attrs_to_remove))

for u, v, attrs_to_remove in edges_to_clean:
    for attr_key in attrs_to_remove:
        # Excluir o atributo problemático dos dados da aresta
        if attr_key in g[u][v]:
            del g[u][v][attr_key]

print(f"Grafo Limpo. Nós: {len(g.nodes())}, Arestas: {len(g.edges())}")
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
# Análise de Métricas
print("Calculando métricas...")

# Degree
degree_dict = dict(g.degree())
nx.set_node_attributes(g, degree_dict, 'degree')

print("Calculando Degree Centrality...")
deg_cent = nx.degree_centrality(g)
nx.set_node_attributes(g, deg_cent, 'degree_centrality')

# Betweenness
print("Calculando Betweenness Centrality (pode demorar)...")
bet_cent = nx.betweenness_centrality(g, k=100) # k=100 aproxima o cálculo para ser mais rápido
nx.set_node_attributes(g, bet_cent, 'betweenness_centrality')

# K-Core e K-Shell
print("Calculando Core Number...")
g_undirected = g.to_undirected()
g_undirected.remove_edges_from(nx.selfloop_edges(g_undirected)) # garante limpeza
core_numbers = nx.core_number(g_undirected)
nx.set_node_attributes(g, core_numbers, 'k_core')

print("Cálculos finalizados e atributos salvos nos nós.")
```
---

## 6. Destaques da Análise

A análise da rede gerada revelou insights importantes sobre a topologia do conhecimento na Wikipedia. Abaixo detalhamos os principais achados quantitativos e qualitativos:

Após os processos de limpeza e fusão de nós duplicados, a rede final consolidou-se com **8.876 nós** e **10.634 arestas**.
    
* **Identificação de Hubs e Comunidades:**
    * **Hubs Globais:** Tópicos como *"French Revolution"* e *"Existentialisme"* comportaram-se como grandes hubs, apresentando um alto grau de entrada (*In-Degree*). Isso reflete a natureza desses assuntos: eventos globais e tecnologias transversais que tocam diversas outras áreas (filosofia, geografia, ética, engenharia).
      
    * **Comunidades Coesas:** O tópico *"Renaissance Art"* formou clusters mais densos e fechados. A análise visual e métrica indicou uma forte interconexão entre artistas, obras e locais geográficos específicos (como Itália e Flandres), criando uma "comunidade" temática bem definida dentro do grafo maior.

* A análise topológica confirmou que a Wikipedia possui características de redes de mundo pequeno (*Small-World Networks*). Apesar de cobrir domínios vastamente diferentes (Física a Arte Renascentista), o diâmetro da rede é relativamente curto, significando que é possível navegar de um tópico a outro com poucos cliques através dos hubs conectores.

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

Em redes de informação como a Wikipedia, observamos frequentemente uma distribuição de lei de potência (*scale-free*), onde poucos nós possuem muitas conexões (Hubs) e a maioria possui poucas. Além disso, a métrica de *Betweenness* revela os "controladores de fluxo" da informação.

Observando a Figura 1, destacam-se os seguintes pontos:

1.  **Hubs (Grandes):** Os nós de maior tamanho correspondem, como esperado, às sementes iniciais e aos termos generalistas. Identificamos **Revolução Francesa e Existencialismo** como um dos maiores hubs da rede.
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
Ao filtrar a rede, observamos que o núcleo é composto majoritariamente pelo tópico de **Buraco Negro**.

Isso demonstra que esse assunto possui uma base de conhecimento mais interconectada e recursiva. Em contraste, tópicos como **Renascença e Existencialismo** tendem a ficar na periferia (K-Shells baixos), pois citam muitos fatos, mas são pouco citados reciprocamente fora de seu contexto imediato.

<p align="center">
  <img src="https://github.com/ariadnec-es/AnaliseRedesComplexas/blob/main/images/kcoredecomp.png" alt="K-core" width="90%">
</p

### 7.3 Detecção de Comunidades (Modularidade)

**Metodologia Visual:**
Aplicamos o algoritmo de *Louvain* para cálculo de modularidade.
* **Cores:** Cada cor distinta (Partition) representa uma classe de modularidade (Comunidade), ou seja, um grupo de páginas que se linkam mais entre si do que com o resto da rede.

**Análise dos Resultados:**
A Figura 3 revela uma formação com mais de 40 comunidades, portanto, resolvemos mostrar só as **10** maiores comunidades bem definidas. A segmentação visual valida a escolha das sementes iniciais, pois observamos clusters temáticos claros:

* **Comunidade Amarelo:** Agrupa tópicos relacionados à **Revolução Francesa**, mostrando ligações entre militares franceses como *Jean-de-Dieu Soult*, *Emmanuel de Grouchy*, entre outros. Assuntos políticos também estão no meio, como exemplo *Termidorianos* e *Descristianização da França*.
* **Comunidade Lilás:** Agrupa tópicos relacionados a **Arte Renascentista**, destacando movimentos artísticos como *Expressionismo Abstrato* e *Hyperrealism*. Além de escolas de arte, entre elas a *Hudson River School* e *Film Noir*.
* **Comunidade Laranja:** Agrupa tópicos relacionados a **Inteligencia Artificial**, conseguimos encontrar topicos relacionados a tecnincas de aprendizagem de máquina, por exemplo *Funções de Perda* e *Aprendizado por Reforço*, tópicos tambem sobre segurança e ética, indicando que a rede também mapeia as discussões sobre segurança e o futuro da tecnologia, como *Statement On AI Risk Of Extinction* e *Existential Risk From Artificial General Intelligence*.

As comunidades não possuem apenas conexões entre si, existem também nós que fazem parte de de conexões entre um ou mais grupos:

*   O nó **Buraco Negro** serve como ponte para as comunidades **Rosa e Verde**, que abordam assuntos de *Física teórica e Astronomia / Leis fundamentais da gravidade*, respectivamente.
*   O nó **Existencialismo / Azul escuro** é um nó em comum para quatro comunidades diferentes, sendo as que mais se destacam **Azul/Verde Escuro**, relacionadas a assuntos de Filosofia da Mente, conectando movimentos e escolas filosóficas / Filosofia Aplicada, Ética e História das Ideias conectando conceitos morais, escolas históricas e campos interdisciplinares (como biologia e feminismo), respectivamente.

<p align="center">
  <img src="https://github.com/ariadnec-es/AnaliseRedesComplexas/blob/main/images/modularityclass.png" alt="Comunidades" width="90%">
</p

---

## 8. Materiais Adicionais

* **Notebook Colab:** [Acesse o código aqui!](https://colab.research.google.com/drive/19uU41FSju0OfrUMH-m1co90nEPOsd1aT?usp=sharing)

* **Vídeo de Apresentação:** [Acesse o vídeo aqui!]()

* As análises e visualizações interativas foram geradas pelo plugin **Sigma.js**, que permite explorar clusters individualmente. Disponível atravéz do link: [Wikipedia Network](https://ariadnec-es.github.io/AnaliseRedesComplexas/network/)

---
