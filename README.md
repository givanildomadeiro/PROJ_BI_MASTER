## Projeto Final do Curso CCE PUC-Rio - BI MASTER 2019.2

### Otimização de Custo de Transporte de Combustíveis

Aluno: Givanildo Madeiro de Souza - Matrícula: 192.190.006

Orientador: Felipe Borges

### Trabalho apresentado ao curso BI MASTER como pré-requisito para conclusão de "Curso de Pós Graduação Business Intelligence Master"



### **Resumo**

O caminho percorrido pelos combustíveis no Brasil, da origem, distribuição e consumo pelos veículos, é um tanto extenso e repleto de desafios, a iniciar pelo tamanho do país. 

Uma das empresas responsáveis por esta façanha é a Transpetro, que detém a maior e mais bem estruturada malha de transporte de combustíveis do Brasil. 
Calcular os custos logísticos consiste em um dos maiores desafios para uma empresa como a Transpetro, podendo gerar grandes prejuízos e desperdício de tempo, caso um cálculo não saia como o desejado.

Este trabalho visa gerar cenários com o objetivo de minimizar os custos logísticos com base na distribuição e localização das origens e destinos das refinarias e distribuidoras.


### **1. Introdução**

A Petrobras Transporte S.A. – Transpetro é uma empresa brasileira de transporte e logística de combustíveis, atuando com importação e exportação desde o petróleo cru até os seus derivados, gás e etanol.

Com mais de 14 mil km em oleodutos e gasodutos, navios e terminais (armazenadores de produtos antes de irem para refinarias ou exportação), a Transpetro tem o desafio de levar para o Brasil inteiro, o combustível que move a economia do país.
Para fazer com que isto aconteça, os combustíveis precisam percorrer um caminho com algumas etapas. Partindo da origem como uma refinaria, eles são transportados para as bases primárias e secundárias, ou seja, as distribuidoras. 

O primeiro destino envolve as bases primárias, conhecidas por possuírem grandes tanques verticais ou horizontais. Esta primeira etapa é realizada através de oleoduto ou gasoduto. 

A segunda etapa está representada pelas bases secundárias, que recebem os combustíveis principalmente pelo modal rodoviário, representado pelos caminhões-tanque.

A terceira etapa é concluída com os combustíveis chegando à rede de postos espalhados pelo Brasil. 

Um dos grandes desafios é otimizar o uso dessa malha de maneira a minimizar os custos com o transporte, obedecendo vários critérios estabelecidos entre a origem e o destino.
A finalidade do presente trabalho é demonstrar a aplicação da otimização desta questão de logística apresentado acima.

Visando simplificar o escopo e viabilizar o trabalho, desconsideramos algumas características do problema real: a localização real das refinarias e das distribuidoras, a não especificação dos múltiplos tipos de combustíveis existentes bem como as reais malhas modais existentes.


### **2. Modelagem**

**2.1 Objetivo:**

O objetivo do projeto é obter o menor custo da operação com o transporte entre as refinarias e as distribuidoras, considerando a capacidade de produção das refinarias e a demanda das distribuidoras.

**2.2 Premissas:**

- As Refinarias possuem localização pré-estabelecida e uma capacidade de produção (em volume m³) diária que não pode ser excedida.
- As Distribuidoras possuem localização conhecida e uma demanda diária (em volume m³) de consumo.

**2.3 Aplicação e codificação:**

Para esta aplicação, iremos utilizar o pacote Python-MIP que fornece ferramentas para modelagem e solução de Problemas de Programação Linear Inteira Mista (MIPs) em Python. A instalação padrão inclui o COIN-OR Linear Programming Solver - CLP, que é atualmente o solucionador de programação linear de código aberto mais rápido e o solver COIN-OR Branch-and-Cut - CBC, um solver MIP altamente configurável. Ele também funciona com o solver Gurobi MIP de última geração. Python-MIP foi escrito em Python moderno e digitado funcionando com o compilador Python rápido just-in-time Pypy.

Abaixo iremos apresentar o passo a passo dos comandos:

**2.3.1: Instalação do pacote Python-MIP.**

    !pip install mip
    import matplotlib.pyplot as plt
    from math import sqrt, log
    from itertools import product
    from mip import Model, xsum, minimize, OptimizationStatus

**2.3.2: Determinação da quantidade de refinarias, suas posições e suas capacidades produtivas (volume m³).**

    Refinarias = [1, 2, 3]
    pRefinarias = {1: (38, 180), 2: (95, 165), 3: (80, 60)}
    cRefinarias = {1: 700, 2: 2400, 3: 2200}

**2.3.3: Definição das distribuidoras (12 unidades), com suas localizações e demandas (volume m³).**

    Distribuidoras = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
    pDistribuidoras = {1: (45, 220), 2: (25, 130), 3: (55, 110), 4: (90, 105), 5: (65, 180), 6: (70, 80), 7: (74, 32), 8: (66, 18), 9: (69, 0), 10: (80, 165), 11: (90, 50), 12: (100, 136)}
    dDistribuidoras = {1: 302, 2: 273, 3: 275, 4: 266, 5: 87, 6: 296, 7: 297, 8: 310, 9: 302, 10: 309, 11: 420, 12: 100}

**2.3.4: Projeção das Refinarias e Distribuidoras**

    plt.figure(figsize=(7,7))
    plt.title("Mapa :: Refinarias x Distribuidoras")
    plt.xlabel("Distância X")
    plt.ylabel("Distância Y")

    for i, p in pRefinarias.items():
        plt.scatter((p[0]), (p[1]), marker="^", color="green", s=300)
        plt.text((p[0]), (p[1]), "$R_%d$" % i, size=14)

    for i, p in pDistribuidoras.items():
        plt.scatter((p[0]), (p[1]), marker="o", color="brown", s=25)
        plt.text((p[0]), (p[1]), "$D_{%d}$" % i, size=14)

![image](https://user-images.githubusercontent.com/60948599/136705028-0d539766-b8eb-4b47-a11d-90f5dc239bc0.png)

**2.3.5: Cálculo das distâncias entre as Refinarias e as Distribuidoras:**

    dist = {(f, c): round(sqrt((pRefinarias[f][0] - pDistribuidoras[c][0]) ** 2 + (pRefinarias[f][1] - pDistribuidoras[c][1]) ** 2), 1)
                for (f, c) in product(Refinarias, Distribuidoras) }

**2.3.6: Criação do modelo MIP:**

    modelo = Model()
    zRefinarias = {i: modelo.add_var(ub=cRefinarias[i]) for i in Refinarias} 

**2.3.7: Definição dos volumes que serão transferidos da Refinaria (i) para a Distribuidora (j):**

    xRefinarias = {(i, j): modelo.add_var() for (i, j) in product(Refinarias, Distribuidoras)}
    for j in Distribuidoras:
        modelo += xsum(xRefinarias[(i, j)] for i in Refinarias) == dDistribuidoras[j]

**2.3.8: Processamento da capacidade das Refinarias e o consumo das Distribuidoras:**

    for i in Refinarias:
        modelo += zRefinarias[i] >= xsum(xRefinarias[(i, j)] for j in Distribuidoras)

**2.3.9: Função Objetivo:**

    modelo.objective = minimize(
        xsum(dist[i, j] * xRefinarias[i, j] for (i, j) in product(Refinarias, Distribuidoras)) )
    modelo.optimize()

**2.3.10: Demonstração gráfica do resultado:**

    plt.figure(figsize=(7,7))
    plt.title("Mapa :: Refinarias x Distrbuidoras")
    plt.xlabel("Distância X")
    plt.ylabel("Distância Y")

    for i, p in pRefinarias.items():
        plt.scatter((p[0]), (p[1]), marker="^", color="green", s=300)
        plt.text((p[0]), (p[1]), "$R_%d$" % i, size=14)

    for i, p in pDistribuidoras.items():
        plt.scatter((p[0]), (p[1]), marker="o", color="brown", s=25)
        plt.text((p[0]), (p[1]), "$D_{%d}$" % i, size=14)

    if modelo.num_solutions:
        print("Solução com o melhor Custo: {}".format(modelo.objective_value))
        print("Capacidades R1, R2 e R3: {} ".format([zRefinarias[f].x for f in Refinarias]))

    # Plotando as alocações
    for (i, j) in [(i, j) for (i, j) in product(Refinarias, Distribuidoras) if xRefinarias[(i, j)].x >= 1e-6]:
        plt.plot(
            (pRefinarias[i][0], pDistribuidoras[j][0]), (pRefinarias[i][1], pDistribuidoras[j][1]), linestyle="--", color="darkgray"
        )

### **3. Resultados**

    Solução com o melhor Custo: 117136.0
    Capacidades R1, R2 e R3: [700.0, 2400.0, 2200.0]

![image](https://user-images.githubusercontent.com/60948599/136705239-b43c0a23-9e6f-4269-88b1-1840550c1ea7.png)


### **4. Conclusão**

O trabalho incluiu o objetivo de demonstrar como conseguir o melhor resultado em custo com transporte de combustíveis no Brasil, levando em consideração as localizações de origem e destino, bem como as capacidades das refinarias e demanda das distribuidoras.

Iniciamos apresentando o contexto do transporte de combustíveis no país, bem como os modais envolvidos nas operações e os desafios de encontrar a melhor relação de custo beneficio deste setor.

No contexto do trabalho foram desconsideradas a localização real das refinarias e das distribuidoras, a não especificação dos múltiplos tipos de combustíveis existentes bem como as reais malhas modais existentes.

Em seguida, procedemos com a escolha do pacote de tecnologia mais eficaz que nos apoiasse na solução do problema.

O pacote Python-MIP dispõe de algoritmos inteligentes combinados com uma alta performance de execução. Com ele, conseguimos chegar a um modelo de cálculo que servirá de base para projetos que utilizem dados reais de localização (longitude e latitude), bem como o mapeamento completo da malha modal do nosso país. 
