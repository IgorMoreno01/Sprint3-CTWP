# Sistema de Análise de Risco para Equipamentos Agrícolas — SOMPO Seguradora

Solução em Python para identificação de fatores ambientais e operacionais que aumentam o risco de dano ou perda de equipamentos agrícolas, desenvolvida como desafio da SOMPO Seguradora.

O sistema calcula um score de risco por equipamento, classifica o nível de exposição, gera alertas preventivos e recomendações acionáveis, e disponibiliza tudo através de um MVP funcional que recebe dados de sensores simulados.

---

## Índice

- [Contexto e objetivo](#contexto-e-objetivo)
- [Funcionalidades](#funcionalidades)
- [Arquitetura da solução](#arquitetura-da-solução)
- [Evolução por sprint](#evolução-por-sprint)
- [Metodologia](#metodologia)
- [Tecnologias](#tecnologias)
- [Como executar](#como-executar)
- [Como usar](#como-usar)
- [Base de dados](#base-de-dados)
- [Resultados obtidos](#resultados-obtidos)
- [Limitações e próximos passos](#limitações-e-próximos-passos)
- [Autor](#autor)

---

## Contexto e objetivo

Equipamentos agrícolas operam expostos a uma combinação de fatores de risco: condições climáticas adversas, terrenos difíceis, proximidade de áreas alagáveis e desgaste acumulado ao longo dos anos. Para uma seguradora, antecipar quais equipamentos têm maior probabilidade de sofrer dano ou perda é essencial tanto para precificação quanto para ação preventiva.

Este projeto constrói uma solução analítica que:

1. Estrutura e trata dados operacionais, ambientais e de histórico de incidentes
2. Quantifica o risco de cada equipamento em um score de 0 a 100
3. Segmenta a frota em perfis estratégicos de risco
4. Simula cenários de mitigação ("e se o equipamento mudasse de operação?")
5. Entrega tudo em um protótipo funcional que consome dados dinâmicos

---

## Funcionalidades

### Análise de risco
- Cálculo de score de risco (0–100) por equipamento, separado em dimensão **operacional** e **ambiental**
- Classificação em três níveis: **Baixo**, **Médio** e **Alto**
- Geração automática de alertas preventivos com base em condições críticas
- Recomendações acionáveis personalizadas por equipamento

### Inteligência analítica
- Análise de correlação entre variáveis e identificação dos principais drivers de risco
- Segmentação da frota em perfis estratégicos via **K-Means** (aprendizado não supervisionado)
- Modelo de classificação com **Random Forest** para predição de nível de risco
- Simulação de cenários de mitigação com medição de impacto no score

### MVP funcional
- Recebimento de dados via payload **JSON** (simulação de sensor/API)
- Validação de campos obrigatórios, tipos e faixas antes do processamento
- Pipeline integrado de ponta a ponta: entrada → validação → processamento → saída
- Dashboard consolidado com tabelas, resumos e visualizações
- Menu interativo com acesso a todas as funcionalidades

---

## Arquitetura da solução

O sistema foi estruturado em três camadas com responsabilidades separadas:

```
┌─────────────────────────────────────────────────────────────┐
│                         ENTRADA                             │
│  receber_dados_sensor()   →  converte payload JSON          │
│  validar_dados()          →  campos, tipos e faixas         │
│  tratar_inconsistencias() →  normalização de texto          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      PROCESSAMENTO                          │
│  calcular_scores()          →  score operacional/ambiental  │
│  classificar_nivel_risco()  →  Baixo / Médio / Alto         │
│  identificar_perfil()       →  perfil estratégico           │
│  gerar_alertas()            →  condições críticas           │
│  gerar_recomendacoes()      →  ações sugeridas              │
│                                                             │
│  processar_equipamento()    →  orquestra o pipeline         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                          SAÍDA                              │
│  exibir_resultado()  →  relatório individual legível        │
│  dashboard()         →  painel consolidado + gráfico        │
│  executar_sistema()  →  menu interativo                     │
└─────────────────────────────────────────────────────────────┘
```

O ponto central da arquitetura é a função `processar_equipamento()`, que orquestra o fluxo completo sem reimplementar regras: ela apenas coordena as funções construídas nas sprints anteriores. Isso mantém uma única fonte de verdade para as regras de negócio.

---

## Evolução por sprint

### Sprint 1 — Fundação analítica
Estruturação da base de dados, tratamento de valores nulos, padronização de categorias e conversão de tipos. Criação das variáveis analíticas de risco (operacional, ambiental, idade, histórico), cálculo do score inicial ponderado, classificação em três níveis e sistema de alertas preventivos.

### Sprint 2 — Aprofundamento e IA
Refinamento da base com detecção e tratamento de outliers, análise exploratória avançada com matriz de correlação e análise temporal. O score foi reformulado separando risco ambiental e operacional, incorporando temperatura e umidade que estavam subutilizadas. Segmentação da frota via K-Means, simulação de cenários, geração de recomendações e treinamento de um modelo Random Forest.

### Sprint 3 — MVP funcional
Integração de todos os componentes em um protótipo que funciona de ponta a ponta. Estruturação do backend em camadas, simulação de recebimento de dados de sensores via JSON, validação de entrada, dashboard consolidado e menu interativo unificado.

---

## Metodologia

### Composição do score de risco

O score final combina duas dimensões independentes, cada uma normalizada para a escala 0–100:

**Score operacional** — fatores ligados à operação e ao equipamento:

| Componente | Peso | Justificativa |
|---|---|---|
| Histórico de incidentes | 45% | Único indicador factual baseado em ocorrências reais |
| Tipo de operação | 35% | Reflete exposição direta ao risco |
| Tempo de uso | 20% | Indicador secundário de desgaste |

**Score ambiental** — fatores ligados ao contexto ambiental:

| Componente | Peso | Justificativa |
|---|---|---|
| Condição climática | 50% | Avaliação categórica consolidada |
| Temperatura | 25% | Dado contínuo de sensor |
| Umidade | 25% | Dado contínuo de sensor |

**Score final** = (Score operacional × 60%) + (Score ambiental × 40%)

O peso maior para a dimensão operacional se justifica por ser mais persistente e controlável, enquanto condições ambientais são transitórias.

### Classificação de risco

Os pontos de corte foram definidos com base nos quartis observados na distribuição do score:

| Faixa | Classificação |
|---|---|
| 0 – 39 | Baixo |
| 40 – 59 | Médio |
| 60 – 100 | Alto |

### Perfis estratégicos

O K-Means identificou quatro agrupamentos naturais ao cruzar as duas dimensões de risco:

| Perfil | Característica |
|---|---|
| **Crítico** | Risco operacional e ambiental elevados |
| **Risco Operacional** | Problema concentrado na operação e no equipamento |
| **Risco Ambiental** | Problema concentrado nas condições ambientais |
| **Baixo Risco** | Ambas as dimensões controladas |

Essa separação permite direcionar ações específicas: um equipamento de perfil ambiental precisa de ajuste na janela de operação, enquanto um de perfil operacional demanda manutenção ou mudança de rota.

---

## Tecnologias

| Biblioteca | Uso no projeto |
|---|---|
| **pandas** | Manipulação, agregação e organização dos dados |
| **numpy** | Geração de dados e operações numéricas |
| **scikit-learn** | K-Means (segmentação) e Random Forest (classificação) |
| **matplotlib** | Visualizações de apoio à análise |
| **json** | Simulação de integração com sensores e APIs |

Desenvolvido em **Python 3** no **Google Colab**.

---

## Como executar

### Google Colab (recomendado)

1. Faça o download do arquivo `.ipynb` deste repositório
2. Acesse [Google Colab](https://colab.research.google.com)
3. Vá em `Arquivo` → `Fazer upload de notebook` e selecione o arquivo
4. Execute em `Ambiente de execução` → `Reiniciar e executar tudo`

Não é necessário instalar nada: todas as bibliotecas já vêm disponíveis no Colab.

### Ambiente local

```bash
pip install pandas numpy scikit-learn matplotlib jupyter
jupyter notebook
```

### Observação sobre a execução

A última célula do notebook é o menu interativo. Ao executar todas as células, a execução pausa nela aguardando entrada do usuário — este é o comportamento esperado, não um erro. As células anteriores executam integralmente e deixam todas as análises prontas.

---

## Como usar

Ao executar a última célula, o menu interativo é exibido:

```
==========================================================
SISTEMA DE ANALISE DE RISCO - SOMPO SEGURADORA
Equipamentos Agricolas - MVP
==========================================================
--- MVP: entrada de dados e processamento ---
1 - Processar lote de sensores
2 - Dashboard dos resultados
3 - Analisar novo equipamento (entrada manual)
4 - Detalhar equipamento processado
--- Analises da base historica ---
5 - Ranking de risco por operacao
6 - Perfis estrategicos (K-Means)
7 - Principais drivers de risco
8 - Buscar equipamento na base
9 - Comparar cenarios de mitigacao
0 - Sair
==========================================================
```

### Exemplo: analisando um equipamento novo (opção 3)

O sistema solicita os dados e retorna a análise completa:

```
ANALISE DE RISCO - SENSOR-001
==========================================================
Colheitadeira | Centro-Oeste | Proximidade de água
Clima: Tempestade | 34.5C | 72.0% umidade
Uso: 12 anos | Incidentes: 4
----------------------------------------------------------
Score operacional :   95.0
Score ambiental   :  86.98
SCORE DE RISCO    :  91.79  [ALTO]
Perfil            : Crítico
----------------------------------------------------------
ALERTA: risco elevado. Acao preventiva recomendada com urgencia.

ALERTAS (4):
  [!] Risco elevado por operação próxima à água
  [!] Condição climática adversa: Tempestade
  [!] Histórico crítico: 4 incidentes
  [!] Classificação geral: ALTO RISCO

RECOMENDACOES (4):
  -> Prioridade maxima: inspecao imediata e plano de mitigacao
  -> Reduzir exposicao a areas alagaveis / priorizar campo aberto
  -> Ajustar janela de operacao para evitar clima adverso
  -> Programar manutencao preventiva (historico elevado)
==========================================================
```

### Formato do payload de entrada

O sistema consome dados no formato JSON, simulando o recebimento de telemetria:

```json
{
    "id_equipamento": "SENSOR-001",
    "tipo_equipamento": "Colheitadeira",
    "regiao": "Centro-Oeste",
    "tipo_operacao": "Proximidade de água",
    "condicao_climatica": "Tempestade",
    "temperatura": 34.5,
    "umidade": 72.0,
    "tempo_uso_anos": 12,
    "historico_incidentes": 4
}
```

**Campos obrigatórios e valores aceitos:**

| Campo | Tipo | Valores válidos |
|---|---|---|
| `id_equipamento` | texto | identificador livre |
| `tipo_equipamento` | texto | Trator, Colheitadeira, Pulverizador, Plantadeira, Semeadora |
| `regiao` | texto | Sul, Sudeste, Centro-Oeste, Nordeste, Norte |
| `tipo_operacao` | texto | Campo aberto, Transporte, Proximidade de água, Encosta, Armazenamento |
| `condicao_climatica` | texto | Seco, Chuvoso, Tempestade, Neblina, Normal |
| `temperatura` | numérico | -5 a 45 (°C) |
| `umidade` | numérico | 0 a 100 (%) |
| `tempo_uso_anos` | numérico | 0 a 50 |
| `historico_incidentes` | numérico | 0 a 20 |

Payloads que não atendam a esses critérios são rejeitados com a lista de erros encontrados.

---

## Base de dados

O projeto utiliza uma base de **900 registros gerados sinteticamente**, com uso autorizado para fins didáticos. A geração usa semente fixa (`seed=42`), garantindo reprodutibilidade: qualquer execução produz exatamente os mesmos dados.

A base foi construída deliberadamente com imperfeições para permitir demonstrar as etapas de tratamento:

- Valores nulos em campos ambientais
- Inconsistências de capitalização e espaçamento em categorias
- Outliers com valores fisicamente impossíveis (umidade acima de 100%, temperatura acima de 45°C)

### Tratamento aplicado

| Problema | Solução |
|---|---|
| Nulos em variáveis numéricas | Imputação pela mediana |
| Nulos em variáveis categóricas | Imputação pela moda |
| Inconsistências de texto | Normalização com `strip()` e `title()` |
| Outliers | Detecção por IQR e por limites físicos, seguida de re-imputação pela mediana regional |

Sobre a detecção de outliers: o método IQR isoladamente capturou apenas metade dos valores impossíveis, porque outliers numerosos inflam o próprio intervalo interquartil. A combinação com limites físicos de domínio (umidade não excede 100%, temperatura de operação não atinge 45°C) capturou a totalidade dos casos.

---

## Resultados obtidos

### Distribuição de risco da frota

| Classificação | Equipamentos | Percentual |
|---|---|---|
| Médio | 517 | 57,4% |
| Alto | 250 | 27,8% |
| Baixo | 133 | 14,8% |

### Principais drivers de risco

A importância das variáveis segundo o modelo Random Forest:

| Variável | Importância |
|---|---|
| Histórico de incidentes | 0,230 |
| Umidade | 0,131 |
| Temperatura | 0,130 |
| Tempo de uso | 0,118 |

Três métodos independentes convergiram para o mesmo resultado: a análise de correlação (0,75), o peso atribuído no score (45%) e a importância no modelo apontaram o **histórico de incidentes** como o principal preditor de risco.

A relevância de temperatura e umidade valida a decisão de refinamento do score: na versão inicial essas variáveis tinham correlação praticamente nula com o risco, porque o modelo usava apenas a condição climática categórica. Após incorporá-las ao score ambiental, tornaram-se o segundo e terceiro fatores mais importantes.

### Desempenho do modelo

O Random Forest atingiu **83,3% de acurácia** no conjunto de teste. A matriz de confusão revelou um comportamento desejável: todos os erros ocorreram entre classes adjacentes (Baixo↔Médio, Médio↔Alto), sem nenhuma confusão entre Baixo e Alto. O modelo nunca classificou um equipamento seguro como crítico, ou vice-versa.

### Impacto das ações de mitigação

A simulação de cenários quantifica o benefício de cada ação preventiva. Para o equipamento de maior risco da base:

| Cenário | Score | Variação |
|---|---|---|
| Situação atual | 92,28 | — |
| Mudança para campo aberto | 79,68 | −12,60 |
| Operação em clima normal | 76,28 | −16,00 |
| Ambas as ações combinadas | 63,68 | −28,60 |

Em escala, mover todos os equipamentos que operam próximo à água para campo aberto reduziria o score médio desse grupo em 12,60 pontos.

---

## Limitações e próximos passos

### Limitações atuais

- A base é sintética. Os padrões refletem as regras usadas na geração, não comportamento real de campo
- Os pesos do score foram definidos por critério analítico e precisariam ser calibrados com dados históricos reais de sinistros
- A análise temporal não identificou sazonalidade, o que é esperado dada a distribuição aleatória das datas na geração
- O sistema processa dados pontuais, sem acompanhamento da evolução de um mesmo equipamento ao longo do tempo

### Evoluções possíveis

- Integração com APIs meteorológicas reais para obter condições climáticas em tempo real
- Recalibração dos pesos a partir de dados históricos de sinistros da seguradora
- Persistência dos resultados em banco de dados para análise de séries temporais
- Exposição do motor de risco como API REST para consumo por outros sistemas
- Interface web substituindo o menu de terminal
