# Introdução

- Queremos classificar as cidades em termos de qualidade de serviços de telecomunicações
- Hoje é dificil fazer isso porque temos que analisar muitas variáveis, dificilmente todas elas são melhores quando comparamos os municipios. É dificil também porque não existe objetivamente uma definição de qual é uma cidade melhor ou pior, dependendo da experiencia de campo para validação.
- Uma alternativa é construir um escore que unifica todas as variáveis, agencias reguladoras costumeiramente usam combinações lineares agregando em um único valor as caracteristicas da cidade
- Um exemplo de escore utilizado é o IQS, da anatel, que pontualmente com base nos valores lidos das cidades calcula um escalar que representa a qualidade do serviço naquele municipio no periodo
- Calcular pontualmente o escore para um municipio desconsidera a relação com outros municipios, trazendo notavel ruidez quando analisamos a classificação dos municipios ao longo do tempo
- Devido aos dados e à forma como é construido aglomera maior parte do valor em uma faixa estreita, o que torna dificil a distinção e assim o uso para classificação das cidades.
- Nós propomos um modelo autosupervisionado de GNN LTR baseado em competição que compara em pares todas as cidades e regride um escore para cada entidade municipio--periodo

# Dataset 

- Para classificação das entidades municipio--periodo utilizamos dados de QoS como {lista de INDs}
- Sabendo que não apenas as variáveis de QoS representam a qualidade de telecomunicações, utilizamos informações de QoE como ISG e infraestrutura a partir do IBC
- Considerando isso aplicamos todos estes dados em uma única matriz $\mathbf{X} \in \mathbb{R}^{i \cross d}$, onde i representa a quantidade entidades municipio--periodo e d é são as features usadas no modelo
- Como base de comparação foram usados dois escores, utilizados pela ANATEL, que consolidam os indicadores de QoS, IQS_leg e IQS_rev. Suas formulações são mostradas a seguir.

// Acho que essa parte de IQS ficou meio perdida no texto dentro de dataset
## Legacy IQS

- Cálculo

## Revised IQS

- Cálculo

# Methodology

- Motivar brevemente o modelo e chamar Fig. 1

Fig. 1: 
- \mathbf{X} -> [Data preprocessing] ->_{X^{\mathrm{model}}} [Graph construction] -> [Positional Encoding] ->  [Model] -> \mathbf{s}
- [Data preprocessing] |_|>_{X^{\mathrm{pareto}} [Model]

## Preprocessamento

- Analisando os dados propostos utilizados no sistema, verificamos a que os dados são muito heterogêneos e heavy-tailed \ref{fig?}.
- Para deixar os dados adequados para otimização neural aplicamos alguns processamentos
- Como primeira etapa de preprocessamento transformamos os dados usando log1p X^\mathrm{log} = \log(1+\mathrm{X^\mathrm{raw}})
- Após usamos Robust Scaling analisando apenas os dados dentro dos percentils p5~p95
- Por fim aplicamos um Standard Scaling para centralizarmos em zero com variancia unitaria, condicionaldo para otimizadores de primeira ordem
- Adicionalmente montamos um proxy utilizado no treinamento X^\mathrm{pareto} que é E_j[x_{i,j}] a partir do MinMax(X^log)

Fig ?: Histograma dos dados utilizados no modelo, pre e pos processmento

## Graph construction and Manifold Structure

- Grafos são capazes de criar estimativas de geometrias de manifold
- Fazemos isso porque? (TODO: tirar dúvida)
- para isso usamos o knn com metrica de minkowski, gerando uma matriz de indices $\mathcal{I} \in  \{1,...,N\}^{N \cross k}$ que indica a lista de vizinhos para cada nó i
- Muitas implementações de knn traçam como vizinho mais proximo o próprio nó, ou seja $i \rightarrow i$, o que gera loops, trazendo um viés residual ao modelo, podendo ser explorado para não acontecer desaparecimento de informação

## Positional Encoding

!TODO: E se usarmos só PCA ao invés de ser fallback e deixa SpectralEmbedding pra outro artigo maior? (já ta caindo sempre no PCA---acho)

- Message passing GNN Permutation-invariance? (TODO: tirar dúvida)
- Devido a essa propriedade de permutation-invariance, precisamos de uma noção de coordenadas globais. Para isso usamos metodos de obter codificacoes posicionais
- Para obter as codificacoes posicionais usamos PCA, que encontra as direcoes ortogonais de máxima variance e projeta com base nesses componentes principais. O que gera uma aproximacao linear das coordenadas do manifold
- Este p_i computado é entao processado com StandardScaler para que seja concatenado com o dado x_i, dessa forma não há sobrevalencia devida à escala

## Model Architecture

// TODO: tirar dúvidas sobre tudo aqui
- Copiar o que ta escrito Message passing $m_i^{(\ell)} = \mathrm{AGG}(\{\phi(h_j^{(\ell)}):(j\rightarrow i) \in E\}), \quad h_i^{(\ell+1)} = \psi(h_i^{(\ell)},m_i^{(\ell)})$
- Copiar o que ta escrito ResGCN
- Falar sobre graph subsampling
- Activation function falar que ta sendo usado o GELU, e que outras 
- Falar sobre monotonicidade
- Falar sobre segunda head de reconstrucao

## Loss terms

- Compass loss
- Reconstruction loss
- Pairwise ranking loss
- Geometric smoothness

# Resultados obtidos

Fig. 2: Histograma comparativo \mathbf{s}, \mathrm{QoS}_\mathrm{leg}, e \mathrm{QoS}_\mathrm{rev}
Fig. 3: Mapas comparativos \mathbf{s}, \mathrm{QoS}_\mathrm{leg}, e \mathrm{QoS}_\mathrm{rev}

# Conclusão

# Trabalhos futuros

- Comparar com baselines de LTR na literatura
- Análise em outros domínios
- Usar outros positional encodings
- Testar outras funções de ativação
