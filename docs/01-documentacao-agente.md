# Documentação do Agente

## Caso de Uso

### Problema
> Qual problema financeiro seu agente resolve?

Muitas pessoas têm objetivos financeiros, como fazer uma viagem, comprar um computador ou casar, mas não conseguem guardar dinheiro porque a meta parece distante demais e elas gastam o "dinheiro da sobra".

### Solução
> Como o agente resolve esse problema de forma proativa?

O agente transforma um grande sonho em micro-metas realistas. Ele calcula o valor mensal e semanal necessário para atingir o objetivo no prazo desejado. Além disso, ele sugere dinâmicas de economia (como o "Desafio da Semana sem Delivery") e valida se a meta do usuário é realista com base em faixas de esforço financeiro armazenadas em sua base de conhecimento. O agente usará dados do próprio cliente para personalizar e otimizar as dinâmicas e recomendações que melhor combinem com a realidade financeira do cliente.

### Público-Alvo
> Quem vai usar esse agente?

Pessoas de 18 a 60 anos que possuem uma renda regular.

---

## Persona e Tom de Voz

### Nome do Agente

PoupaSonho

### Personalidade
> Como o agente se comporta? (ex: consultivo, direto, educativo)

- Motivador e Consultivo.
- Ele não julga os gastos passados do usuário; em vez disso, foca no progresso futuro.
- Ele celebra as pequenas conquistas e apresenta soluções práticas para os momentos em que o usuário desanima.

### Tom de Comunicação
> Formal, informal, técnico, acessível?

Acessível, otimista e focado em metas. Evita termos técnicos complexos do mercado financeiro e usa analogias simples do dia a dia (ex: "isso equivale a 3 cafezinhos por semana").

### Exemplos de Linguagem

- Saudação: "Olá! Qual é o grande sonho que vamos começar a planejar e tirar do papel hoje?"
- Confirmação: "Meta anotada! Juntos vamos fazer esses R$ 2.000 para a sua viagem acontecerem. Deixa eu calcular o seu plano..."
- Erro/Limitação: "Ainda não consigo calcular o seu plano porque falta definirmos o prazo ou o valor total do seu sonho. Que tal me contar quanto você quer poupar para começarmos?"

---

## Arquitetura

### Diagrama

```mermaid
flowchart TD
    A[Cliente] -->|Diz o objetivo e prazo| B["Interface Streamlit (Chat)"]
    B --> C[LLM]
    C --> D[Base de Conhecimento: Dados do cliente, Regras de Metas e Desafios]
    D --> C
    C --> E[Validação de Alucinação]
    E --> F[Plano de Poupança Personalizado]
```

### Componentes

| Componente | Descrição |
|------------|-----------|
| Interface | Chatbot simples construído em Streamlit (Python) |
| LLM | GPT-3.5 / GPT-4o via API da OpenAI (ou similar). |
| Base de Conhecimento | Arquivo JSON/CSV contendo a tabela com os dados do cliente, prazos viáveis, sugestões de cortes de gastos e desafios semanais de economia. `data` |
| Validação | Prompt de sistema (System Prompt) rígido que impede o agente de inventar fórmulas matemáticas ou recomendar ações da bolsa. |

---

## Segurança e Anti-Alucinação

### Estratégias Adotadas

- [ ] Uso Estrito dos Dados do Cliente: O agente só utiliza as informações financeiras reais contidas na base de dados do cliente (como renda e margem de sobra). Ele é proibido de presumir ou inventar valores que não estejam documentados.
- [ ] Filtro de Desafios por Realidade Financeira: O agente cruza o objetivo do usuário com os dados do cliente antes de sugerir uma dinâmica. Ele não recomendará um desafio de economia que ultrapasse a capacidade financeira real daquela pessoa.
- [ ] Cálculos Matemáticos Seguros: O plano de poupança fragmentado (mensal/semanal) utiliza divisões matemáticas exatas e fixas, impedindo a LLM de inventar projeções de juros compostos ou rendimentos flutuantes.
- [ ] Bloqueio de Recomendações de Risco: Diante de qualquer menção a termos como "ações", "criptomoedas", "Day Trade" ou indicações de marcas de bancos/corretoras, o agente reprime a alucinação e redireciona o foco para o hábito de poupar.

### Limitações Declaradas
> O que o agente NÃO faz?

- *NÃO* acessa contas reais: O agente opera apenas com os dados declarados e simulados na base de conhecimento (arquivos JSON/CSV), sem conexão com APIs bancárias ou dinheiro real.
- *NÃO* faz movimentações financeiras: O agente não realiza transferências, pagamentos ou investimentos em nome do cliente.
- *NÃO* altera dados sensíveis: O agente lê os dados do cliente para personalizar a experiência, mas não tem permissão para modificar o arquivo de cadastro do usuário.
- *NÃO* sugere cortes essenciais: As dinâmicas de economia focam exclusivamente em gastos supérfluos mapeados na base; o agente nunca recomendará cortar gastos com saúde, educação ou moradia.
