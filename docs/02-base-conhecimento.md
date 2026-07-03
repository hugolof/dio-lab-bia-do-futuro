# Base de Conhecimento

## Dados Utilizados

Descreva se usou os arquivos da pasta `data`, por exemplo:

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `perfil_investidor.json` | JSON | Identifica o perfil do cliente (João Silva), sua renda mensal estável e os prazos/valores de suas metas de vida (reserva e apartamento) |
| `transacoes.csv` | CSV | Analisa o histórico recente de entradas e saídas para mapear os maiores ralos financeiros do cliente (ex: alimentação e lazer) |
| `produtos_financeiros.json` | JSON | Filtra opções de investimentos exclusivamente de baixo risco e alta liquidez para sugerir como o "cofrinho" ideal do objetivo |
| `historico_atendimento.csv` | CSV | Fornece o histórico de interações passadas para garantir que o agente saiba se o cliente já tirou dúvidas sobre metas antes |
| `desafios_economia.json` | JSON | Catálogo de dinâmicas e micro-economias semanais que a IA escolhe para recomendar ao usuário |

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

- Criação do Catálogo de Desafios (desafios_economia.json): Os arquivos originais detalhavam o comportamento do cliente, no entanto estava faltando as "ações" que o agente poderia propor. Foi criado um arquivo JSON com desafios de economia focados nas mesmas categorias de gastos que aparecem no transacoes.csv.

- Filtro de Segurança em produtos_financeiros.json: Embora o arquivo original contenha opções de renda variável (como Fundos de Ações), o agente irá ler o arquivo e ignorar ativamente opções de médio e alto risco, focando apenas em produtos adequados para a construção de metas.

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

Os dados são carregados de forma assíncrona logo no início da inicialização da aplicação (inicialização do script Streamlit). Em vez de enviar arquivos brutos e massivos para a API da LLM — o que aumentaria drasticamente o custo de tokens e confundiria o modelo —, o backend em Python atua como uma camada de tratamento de dados.A biblioteca nativa json é utilizada para ler as estruturas de perfil e desafios, enquanto o pandas manipula e sumariza as tabelas em formato .csv. Os dados das transações financeiras passam por uma agregação estatística automatizada (calculando totais de receitas, despesas e agrupamentos por categoria) antes de serem embutidos como texto estruturado no prompt do agente.  

```python
import pandas as pd
import json
import os

def carregar_base_conhecimento(pasta_data="data"):

    contexto_consolidado = {}
    
    # 1. Carregando dados estáticos e metas do Cliente
    caminho_perfil = os.path.join(pasta_data, "perfil_investidor.json")
    with open(caminho_perfil, "r", encoding="utf-8") as f:
        perfil = json.load(f)
        contexto_consolidado["cliente"] = perfil

    # 2. Carregando e Filtrando Produtos Financeiros (Regra de Segurança)
    caminho_produtos = os.path.join(pasta_data, "produtos_financeiros.json")
    with open(caminho_produtos, "r", encoding="utf-8") as f:
        todos_produtos = json.load(f)
        # Filtra apenas investimentos de baixo risco para segurança do PoupaSonho
        produtos_seguros = [p for p in todos_produtos if p["risco"] == "baixo"]
        contexto_consolidado["produtos_recomendados"] = produtos_seguros

    # 3. Carregando e Criando o Catálogo de Desafios de Economia
    caminho_desafios = os.path.join(pasta_data, "desafios_economia.json")
    if os.path.exists(caminho_desafios):
        with open(caminho_desafios, "r", encoding="utf-8") as f:
            contexto_consolidado["desafios"] = json.load(f)

    # 4. Processando e Sumarizando o Histórico de Transações (Engine Pandas)
    caminho_transacoes = os.path.join(pasta_data, "transacoes.csv")
    df_transacoes = pd.read_csv(caminho_transacoes)
    
    # Agregações matemáticas essenciais
    total_receitas = df_transacoes[df_transacoes['tipo'] == 'entrada']['valor'].sum()
    total_despesas = df_transacoes[df_transacoes['tipo'] == 'saida']['valor'].sum()
    sobra_real = total_receitas - total_despesas
      
    # 5. Mapeia os maiores gastos agrupados por categoria
    gastos_por_categoria = df_transacoes[df_transacoes['tipo'] == 'saida'].groupby('categoria')['valor'].sum().to_dict()
    
    # Adicionando a saúde financeira ao contexto consolidado
    contexto_consolidado["saude_financeira"] = {
        "total_receitas": total_receitas,
        "total_despesas": total_despesas,
        "sobra_real": sobra_real,
        "maiores_gastos_categoria": gastos_por_categoria
    }
    
    return contexto_consolidado

```

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Os dados não vão de forma massiva ou crua para a LLM. O script Python faz um resumo prévio (métrica de renda, soma de gastos por categoria e metas ativas) e injeta esse compilado estruturado diretamente dentro do contexto da conversa (System Prompt) a cada nova interação da pessoa usuária. Isso economiza tokens e evita que a IA faça cálculos errados de soma.

---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

Abaixo está o exemplo de como o script Python formata e entrega os dados consolidados da pasta `data/` para a inteligência artificial nos bastidores:

```text

CONTEXTO DO SISTEMA - ASSISTENTE POUPASONHO (INJETADO DINAMICAMENTE)


[DADOS DO CADASTRO DO CLIENTE] (Fonte: perfil_investidor.json)
- Nome: João Silva
- Idade: 32 anos
- Profissão: Analista de Sistemas
- Renda Mensal Estável: R$ 5.000,00
- Perfil de Risco: Moderado

[METAS ATIVAS NO SISTEMA] (Fonte: perfil_investidor.json)
1. Meta: Completar Reserva de Emergência
   - Valor Necessário Remanescente: R$ 5.000,00 (Meta total: R$ 15.000,00)
   - Prazo Limite: 2026-06
2. Meta: Entrada do Apartamento
   - Valor Necessário: R$ 50.000,00
   - Prazo Limite: 2027-12

[DIAGNÓSTICO DA SAÚDE FINANCEIRA REAL] (Agregação via transacoes.csv)
- Total de Entradas (Receitas): R$ 5.000,00
- Total de Saídas (Despesas): R$ 2.488,90
- Margem de Sobra Livre: R$ 2.511,10
- Distribuição de Gastos por Categoria:
  * moradia: R$ 1.380,00
  * alimentacao: R$ 570,00
  * transporte: R$ 295,00
  * saude: R$ 188,00
  * lazer: R$ 55,90

[CATÁLOGO DE DESAFIOS DISPONÍVEIS] (Fonte: desafios_economia.json)
- DES001: "Semana Zero Delivery" | Categoria: alimentacao | Economia: R$ 120,00/semana
- DES002: "Corte de Streaming Fantasma" | Categoria: lazer | Economia: R$ 55,90/mês
- DES003: "Fim de Semana Offline de Compras" | Categoria: lazer | Economia: R$ 100,00/semana

[PRODUTOS DE INVESTIMENTO SEGUROS SELECIONADOS] (Filtrado de produtos_financeiros.json)
- Tesouro Selic (Rentabilidade: 100% da Selic | Aporte Mínimo: R$ 30,00)
- CDB Liquidez Diária (Rentabilidade: 102% do CDI | Aporte Mínimo: R$ 100,00)

DIRETRIZ DE COMPORTAMENTO: Use estritamente as métricas reais acima para montar as respostas. Se o João Silva perguntar sobre seus hábitos ou reclamar que faltam recursos para bater as metas, aponte os gastos exatos dele em 'alimentacao' (R$ 570,00) ou 'lazer' (R$ 55,90) e sugira os desafios DES001 ou DES002 como o gatilho prático de economia.

```
