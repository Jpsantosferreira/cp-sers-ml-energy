Ana Julia Yumi Inoue - RM: 569430
João Pedro Santos Ferreira - RM: 569202
Maria Fernanda Dias Ribeiro - RM: 569999
Ulysses Gomes Soares de Souza - RM: 573826
Yasmin Cristina Carvalho Mayer - RM: 573964


O projeto possui duas tarefas independentes de Machine Learning:

1. **Classificação da fonte de geração de empreendimentos da ANEEL**
2. **Regressão da radiação solar em Petrolina (PE)**

Em cada tarefa foram treinados e comparados **três algoritmos diferentes**, totalizando seis modelos.

---

## Objetivo

O objetivo do projeto é aplicar técnicas de aprendizado de máquina em dados relacionados à geração de energia renovável.

Na primeira tarefa, o objetivo é verificar se é possível classificar um empreendimento como **Solar, Eólica ou Hidráulica** utilizando apenas sua potência outorgada e sua localização geográfica.

Na segunda tarefa, o objetivo é estimar a **radiação solar horizontal em W/m²** a partir de variáveis meteorológicas e da hora do dia.

O projeto também busca comparar diferentes algoritmos de Machine Learning utilizando métricas adequadas para classificação e regressão.

---

# 1. Classificação da fonte renovável

## Fonte dos dados

Os dados foram obtidos através do **SIGA — Sistema de Informações de Geração da ANEEL**, utilizando sua API pública.

Fonte:

* ANEEL — https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel

A API utilizada é pública e não exige token ou chave de acesso.

Foram consideradas as seguintes categorias:

| Código ANEEL | Classe utilizada |
| ------------ | ---------------- |
| UFV          | Solar            |
| EOL          | Eólica           |
| UHE          | Hidráulica       |
| PCH          | Hidráulica       |
| CGH          | Hidráulica       |

As categorias UHE, PCH e CGH foram agrupadas em uma única classe chamada **Hidráulica**.

Cada linha representa um empreendimento de geração.

## Variáveis utilizadas

As entradas utilizadas no modelo foram:

* `potencia_kw` — potência outorgada em kW;
* `latitude` — latitude aproximada do empreendimento;
* `longitude` — longitude aproximada do empreendimento.

A variável alvo foi:

* `fonte` — Solar, Eólica ou Hidráulica.

Não foram utilizadas como entrada informações como nome do empreendimento, código CEG, sigla do tipo de geração ou descrições relacionadas à fonte, pois essas informações poderiam entregar diretamente a resposta do modelo.

## Pré-processamento

Foi realizada uma análise dos tipos de dados, valores ausentes, distribuição das classes e características das variáveis.

Os dados foram divididos em:

* **80% para treinamento**
* **20% para teste**

A divisão foi feita de forma **estratificada**, mantendo a proporção das classes entre treinamento e teste.

Foi utilizada uma semente fixa para garantir a reprodutibilidade dos resultados.

Quando necessário, a padronização das variáveis foi realizada utilizando somente os dados de treinamento.

## Algoritmos utilizados

Foram comparados três classificadores:

1. **K-Nearest Neighbors (KNN)**
2. **Regressão Logística**
3. **Random Forest Classifier**

Todos os modelos utilizaram a mesma divisão entre treinamento e teste.

## Métricas

Os modelos foram avaliados utilizando:

* Accuracy
* Precision
* Recall
* F1-Score
* Matriz de confusão

Para Precision, Recall e F1-Score foi utilizada a média **macro**, considerando igualmente as três classes.

### Interpretação

Segundo os resultados do colab classificação apresenta limitações porque a fonte de geração não depende somente da potência e da localização geográfica.

Embora a localização possa apresentar padrões relacionados à distribuição das fontes renováveis e a potência possa apresentar características diferentes entre empreendimentos, existem empreendimentos de fontes distintas com características semelhantes.

Por isso, erros de classificação podem ocorrer principalmente entre classes que possuem regiões geográficas ou faixas de potência semelhantes.

A matriz de confusão apresentada no notebook permite identificar quais classes apresentaram maior quantidade de confusões.

---

# 2. Regressão da radiação solar

## Fonte dos dados

Os dados meteorológicos foram obtidos através da **API histórica da Open-Meteo**.

Fonte:

* Open-Meteo — Historical Weather API: https://open-meteo.com/en/docs/historical-weather-api

A consulta foi realizada para **Petrolina, Pernambuco**, utilizando aproximadamente:

* Latitude: `-9.39`
* Longitude: `-40.50`
* Fuso horário: `America/Recife`

## Período

O período analisado foi:

**01/04/2025 a 30/06/2025**

Foram utilizadas as horas locais entre **07:00 e 17:00**.

Os dados são históricos estimados por modelos/reanálise. Portanto, não representam necessariamente medições realizadas por um sensor específico ou por um painel fotovoltaico.

## Variáveis utilizadas

As entradas utilizadas no modelo foram:

* `temperatura_c` — temperatura do ar em °C;
* `umidade_pct` — umidade relativa em %;
* `nuvens_pct` — cobertura de nuvens em %;
* `vento_kmh` — velocidade do vento em km/h;
* `hora` — hora local do registro.

A variável alvo foi:

* `radiacao_w_m2` — radiação solar global horizontal em W/m².

A coluna `data_hora` foi utilizada para ordenar os dados e preservar a sequência temporal, mas não foi utilizada diretamente como variável de entrada.

Também não foi utilizada a própria `radiacao_w_m2`, nem qualquer transformação direta dela, como entrada do modelo.

## Divisão temporal

Como os dados possuem ordem temporal, não foi realizado embaralhamento.

A divisão utilizada foi:

* **Primeiras 80% das horas:** treinamento
* **Últimas 20% das horas:** teste

Dessa forma, o modelo é treinado com os registros anteriores e avaliado sobre registros posteriores.

## Algoritmos utilizados

Foram comparados três regressores:

1. **Regressão Linear**
2. **Decision Tree Regressor**
3. **Random Forest Regressor**

Todos os modelos utilizaram a mesma divisão temporal entre treinamento e teste.

## Métricas

Os modelos foram avaliados utilizando:

* **MAE** — Erro Absoluto Médio, em W/m²;
* **MSE** — Erro Quadrático Médio, em (W/m²)²;
* **R²** — coeficiente de determinação.

### Interpretação

Segundo o colab variável `hora` possui um papel importante porque a radiação solar apresenta uma forte relação com o período do dia. Entre 07:00 e 17:00, a disponibilidade de radiação varia conforme a posição do Sol, atingindo normalmente valores maiores durante o período próximo ao meio do dia.

Além da hora, variáveis meteorológicas como cobertura de nuvens, temperatura e umidade podem ajudar a explicar as variações da radiação.

Entretanto, a previsão de radiação solar **não é equivalente à previsão da geração elétrica de um sistema fotovoltaico**.

A geração elétrica depende de outros fatores, como:

* potência instalada;
* quantidade e características dos módulos;
* orientação e inclinação dos painéis;
* eficiência dos equipamentos;
* temperatura dos módulos;
* perdas elétricas;
* inversor;
* sombreamento;
* condições específicas da instalação.

Portanto, o modelo desenvolvido estima a radiação solar em W/m² e não a energia elétrica produzida por um sistema fotovoltaico.

---

# Resumo dos resultados

## Classificação

Foram comparados três algoritmos:

* KNN
* Regressão Logística
* Random Forest

A comparação foi realizada utilizando Accuracy, Precision, Recall e F1-Score com média macro.

De forma geral, os resultados mostram que potência e localização possuem alguma capacidade de diferenciar as fontes, mas não são suficientes para representar todos os fatores que determinam a escolha ou classificação de um empreendimento de geração.

## Regressão

Foram comparados:

* Regressão Linear
* Decision Tree Regressor
* Random Forest Regressor

A comparação foi realizada utilizando MAE, MSE e R².

Os resultados devem ser interpretados considerando que os dados meteorológicos são estimativas históricas e que a variável prevista é radiação solar, e não geração elétrica.

---

# Estrutura do projeto

```text
.
AULA07_Regresso_Linear_Dados_Energia.ipynb
Aula_APIs_Energia_Renovavel_ML.ipynb
Data_for_UCI_named.csv
README.md
aneel_classificacao_orange.csv
aula06_ml_energia.ipynb
meteo_regressao_orange.csv
```

---

# Como executar

## 1. Clonar o repositório

```bash
git https://github.com/Jpsantosferreira/cp-sers-ml-energy
cd cp-sers-ml-energy
```

## 2. Criar um ambiente virtual

No Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

No Linux/macOS:

```bash
python3 -m venv venv
source venv/bin/activate
```

## 3. Instalar as bibliotecas

```bash
pip install pandas numpy matplotlib scikit-learn jupyter
```

Caso esteja utilizando o Google Colab, as bibliotecas normalmente já estão disponíveis.

## 4. Abrir o notebook

```bash
jupyter notebook
```

Depois, abra:

```text
Aula_APIs_Energia_Renovavel_ML.ipynb
```

## 5. Executar as células

Execute as células do notebook **na ordem**, desde os imports até as conclusões finais.

O notebook realiza:

1. Consulta das APIs públicas;
2. Preparação dos dados;
3. Geração dos arquivos CSV;
4. Análise exploratória;
5. Separação entre treino e teste;
6. Treinamento dos três classificadores;
7. Avaliação da classificação;
8. Treinamento dos três regressores;
9. Avaliação da regressão;
10. Geração dos gráficos;
11. Comparação e interpretação dos resultados.

As APIs utilizadas não exigem tokens ou senhas.

---

# Reprodutibilidade

Para manter os resultados reproduzíveis:

* foi utilizada uma semente fixa na divisão da classificação;
* a classificação utiliza divisão estratificada;
* a regressão mantém a ordem temporal;
* os mesmos conjuntos de treino e teste são utilizados na comparação dos modelos;
* transformações de dados são ajustadas somente utilizando o conjunto de treinamento quando aplicável.

---

# Limitações

### Classificação

A fonte de um empreendimento não pode ser determinada perfeitamente apenas pela potência e pelas coordenadas geográficas. Outras informações técnicas, regulatórias e de projeto podem ser necessárias para uma classificação mais completa.

Além disso, os dados representam empreendimentos cadastrados no SIGA e não representam diretamente a quantidade de energia efetivamente gerada.

### Regressão

Os dados meteorológicos utilizados são provenientes de uma API histórica baseada em modelos/reanálise. Portanto, podem existir diferenças em relação às condições medidas exatamente no local.

Além disso, o modelo estima radiação solar global horizontal e não a produção de energia de um sistema fotovoltaico específico.

---

# Fontes

* **ANEEL — SIGA — Sistema de Informações de Geração da ANEEL**
  https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel

* **ANEEL — Recurso utilizado na API DataStore**
  https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel/resource/11ec447d-698d-4ab8-977f-b424d5deee6a

* **Open-Meteo — Historical Weather API**
  https://open-meteo.com/en/docs/historical-weather-api

---

# Conclusão

O projeto permitiu aplicar algoritmos de aprendizado supervisionado em dois problemas diferentes relacionados à energia renovável.

Na primeira tarefa, foi realizada uma classificação multiclasses de empreendimentos em Solar, Eólica e Hidráulica utilizando potência e localização.

Na segunda tarefa, foi realizada uma regressão para estimar a radiação solar em Petrolina a partir de variáveis meteorológicas e da hora do dia.

A comparação dos seis modelos permite observar as diferenças entre algoritmos de classificação e regressão e avaliar suas respectivas métricas de desempenho. Os resultados numéricos completos, matrizes de confusão, gráficos e análises estão apresentados no notebook.

O projeto utiliza exclusivamente dados públicos e não contém senhas, tokens ou chaves privadas.
