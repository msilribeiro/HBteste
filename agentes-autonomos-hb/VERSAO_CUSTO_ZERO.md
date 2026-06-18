# Agentes Autônomos HB — Versão Custo Zero

> Esta versão prioriza custo zero ou mínimo, aproveitando ferramentas que o escritório já paga (Microsoft 365 Premium, RD Station, Slack, ChatGPT, Claude Pro) e opções gratuitas de IA e busca web. Os 10 agentes, os prompts e a lógica são os mesmos da versão completa — o que muda é a infraestrutura.

---

## Stack — Custo Zero

| Camada | Versão completa (paga) | Versão custo zero | Economia |
|--------|----------------------|------------------|----------|
| Orquestração | Make.com (~R$ 90-165/mês) | **Power Automate** (já incluso no M365 Premium) | R$ 90-165 |
| LLM | Claude API (~R$ 200-500/mês) | **Google Gemini API** (grátis) | R$ 200-500 |
| Enriquecimento | Apollo.io (~R$ 200-400/mês) | **Busca web + LLM** (grátis) | R$ 200-400 |
| Busca web | Tavily pago (~R$ 140/mês) | **Tavily free** (1.000 buscas/mês) | R$ 140 |
| CRM | RD Station | RD Station | R$ 0 (já pago) |
| Comunicação | Slack | Slack | R$ 0 (já pago) |
| Calendário | — | Outlook Calendar (M365) | R$ 0 (já pago) |
| Transcrição | — | Teams (M365) | R$ 0 (já pago) |
| Docs/Storage | — | SharePoint / Word / Excel Online (M365) | R$ 0 (já pago) |
| Base de tom de voz | — | **Custom GPT (ChatGPT)** com playbook HB | R$ 0 (já pago) |

**Economia total: R$ 630-1.205/mês**
**Custo adicional: R$ 0/mês**

---

## Explicação de cada ferramenta

### 1. Power Automate — o orquestrador (substitui o Make.com)

**O que é:** Ferramenta de automação da Microsoft que **já está incluída no Microsoft 365 Premium** que vocês pagam. Funciona como um maestro: quando algo acontece em um sistema (ex: novo lead no RD, reunião agendada no Outlook), ele dispara uma sequência automática de ações em outros sistemas.

**O que faz na prática:**
- "Quando chegar um novo lead no RD → buscar dados da empresa → chamar a IA → classificar o ICP → atualizar o CRM → avisar a Milena no Slack"
- "15 minutos antes de uma reunião → buscar tudo sobre o lead → montar briefing → enviar para a Juliana no Slack"
- "Toda manhã às 8h → verificar leads sem resposta há 5 dias → gerar mensagens de follow-up → enviar sugestões no Slack"

**Por que não estávamos usando:** Provavelmente ninguém sabia que já estava incluso no plano. O Power Automate faz o mesmo que o Make.com — a diferença é que ele já é de vocês.

**Limitação honesta:** O conector com o RD Station não é nativo (no Make.com é). Solução: usar o módulo HTTP do Power Automate para chamar a API do RD Station diretamente. Funciona igual, só exige um pouco mais de configuração no início.

**Onde acessar:** power.automate.microsoft.com (entrar com o e-mail do M365)

### 2. Google Gemini API — o cérebro dos agentes (substitui Claude API)

**O que é:** A inteligência artificial do Google, disponível gratuitamente como API (Interface de Programação). Quando o Power Automate precisa "pensar" — classificar um lead, gerar uma mensagem, analisar uma transcrição — ele envia os dados para o Gemini e recebe a resposta.

**Por que é gratuito:** O Google oferece um tier gratuito generoso para promover adoção:
- 15 requisições por minuto
- 1.500 requisições por dia
- Modelos de alta qualidade (Gemini 2.5 Flash / Pro)

Para a operação da HB (estimativa de 50-200 chamadas/dia nos 10 agentes), o tier gratuito é mais que suficiente.

**Qualidade em português:** Muito boa. Para 90% dos casos de uso dos agentes (classificar ICP, gerar mensagens, resumir reuniões, montar briefings), o Gemini entrega resultado equivalente ao Claude.

**Quando NÃO é suficiente:** Textos jurídicos sofisticados (proposta detalhada, parecer). Se isso acontecer, migra-se apenas o Agente de Proposta (agente 6) para Claude API (~R$ 50-100/mês de custo marginal). Mas vale testar primeiro.

**"E o ChatGPT e o Claude Pro que já pagamos?"**
Continuam úteis — cada um com seu papel. A assinatura do ChatGPT e do Claude Pro dá acesso ao chat (conversa manual). A API do Gemini é outra coisa: chamadas programáticas que o Power Automate faz sozinho. São três ferramentas complementares:

| Item | ChatGPT (já pago) | Claude Pro (já pago) | Gemini API (grátis) |
|------|-------------------|---------------------|-------------------|
| Como funciona | Chat manual | Chat manual | Power Automate chama automaticamente |
| Papel | Tom de voz do HB (Custom GPT) | Análise profunda, brainstorm | Motor dos 10 agentes |
| Quem usa | Milena, time comercial | Juliana, Thiago, Milena | Power Automate (automático) |
| Custo | Já pago | Já pago | R$ 0 |

**Como contratar o Gemini API:** Acessar ai.google.dev → criar projeto → gerar chave de API. 5 minutos, zero custo.

### 3. Tavily (free) — busca web para os agentes

**O que é:** Serviço de busca web otimizado para IA. Quando o agente precisa "pesquisar uma empresa na internet" — buscar notícias, ver o que a empresa faz, encontrar sinais de captação — o Tavily faz essa busca e retorna o conteúdo já limpo, pronto para a IA analisar.

**Por que não usar Google direto:** O Google retorna links. O Tavily retorna o conteúdo das páginas já extraído. É a diferença entre "aqui estão 10 links" e "aqui está o que esses sites dizem sobre a empresa".

**Plano gratuito:** 1.000 buscas/mês. Para uma operação que processa 30-80 leads/mês, é suficiente.

**Como contratar:** tavily.com → criar conta → gerar API key. Grátis.

### 4. Microsoft 365 Premium — tudo que vocês já pagam

O M365 Premium já inclui tudo isso que os agentes vão usar:

| Ferramenta | Uso nos agentes |
|-----------|----------------|
| **Teams** | Gravações e transcrições de reunião → Agente Pós-Reunião lê automaticamente |
| **Outlook Calendar** | Trigger de reunião → Agente de Briefing dispara 15min antes |
| **SharePoint** | Armazena briefings, propostas, relatórios gerados pelos agentes |
| **Word Online** | Gera propostas e documentos automaticamente |
| **Excel Online** | Relatórios de inteligência comercial, histórico de métricas |
| **Power Automate** | Orquestra todos os 10 agentes |

**Custo adicional: zero. Tudo já contratado.**

### 5. Enriquecimento de dados — sem Apollo, sem custo

Na versão paga, o Apollo.io (R$ 200-400/mês) buscaria dados de empresas automaticamente. Na versão custo zero, substituímos por:

1. **Busca direta ao site da empresa** (módulo HTTP do Power Automate) → extrai informações do site
2. **Tavily free** → busca notícias, captações, expansão
3. **LinkedIn público** (perfil da empresa, acessível sem API paga) → porte, setor, número de funcionários
4. **IA analisa tudo junto** → classifica ICP, identifica dores, calcula probabilidade de fit

**É menos preciso que o Apollo?** Sim — faltam dados como faturamento estimado e tecnologias usadas. **Funciona para a operação atual?** Perfeitamente. O Apollo pode entrar depois se o volume crescer e justificar.

### 6. Custom GPT (ChatGPT) — a voz do escritório

**O que é:** Um GPT personalizado dentro do ChatGPT que vocês já assinam, treinado com o Playbook Comercial, os scripts, o tom de voz, os arquétipos (Sábio 40%, Prestativo 35%, Criador 25%) e os exemplos reais de mensagens do HB. Ele se torna o **repositório vivo do jeito de falar do escritório**.

**Por que isso é importante:** O maior risco de usar IA para gerar mensagens comerciais é soar genérico ou "robotizado". O Custom GPT resolve isso porque ele já conhece:
- O tom consultivo, humano, sem juridiquês
- As frases-chave do HB ("O jurídico que chega antes do problema", "Crescer sem estrutura cobra a conta")
- O que funciona (diagnóstico, conversa leve, perguntas abertas) e o que não funciona (proposta genérica, venda direta, pressão)
- As diferenças entre ICPs (startup vs. empresa tradicional vs. administradora)
- Os scripts de LinkedIn, WhatsApp, e-mail e ligação
- As objeções e como responder a cada uma

**Como funciona na prática — dois usos:**

#### Uso 1: Gerador de referência de tom (manual)
A Milena ou a Juliana abrem o Custom GPT no ChatGPT e pedem:
- "Gera uma mensagem de LinkedIn para uma fintech que acabou de captar"
- "Como abordar uma administradora de condomínio por WhatsApp?"
- "Reescreve essa mensagem no tom do HB"

Ele responde já no tom certo. Isso já funciona hoje, sem automação nenhuma.

#### Uso 2: Base de referência para os agentes automatizados
O Custom GPT gera **exemplos de referência por ICP e canal** que alimentam os prompts do Gemini API nos agentes. Funciona assim:

```
1. No Custom GPT (manual, uma vez):
   → "Gere 10 exemplos de mensagem de LinkedIn para startups em captação"
   → "Gere 10 exemplos de WhatsApp para empresas tradicionais"
   → "Gere 10 exemplos de follow-up para propostas enviadas"

2. Esses exemplos viram o "banco de referência de tom"
   → Salvos no SharePoint em um arquivo de referência

3. Nos agentes automatizados (Power Automate + Gemini):
   → O prompt inclui: "Use os exemplos abaixo como referência de tom..."
   → O Gemini gera a mensagem personalizada NAQUELE tom
   → Resultado: mensagens automáticas que soam como o HB, não como robô
```

**O que colocar dentro do Custom GPT:**

| Conteúdo | De onde vem |
|----------|------------|
| Tom de voz e arquétipos | Playbook seção 1.3 |
| Frases-chave e conceitos | Playbook seção 1.4 |
| O que funciona / não funciona | Playbook seção 1.3 |
| Scripts de LinkedIn | Playbook seção 6.3, 7.1 |
| Scripts de WhatsApp | Playbook seção 7.1 |
| Scripts de ligação | Playbook seção 7.2 |
| Scripts de e-mail | Playbook seção 7.1 |
| Gatilhos para startups | Playbook seção 6.4 |
| Placeholders | Playbook seção 6.5 |
| Gestão de objeções | Playbook seção 9 |
| Diferenças por ICP | Playbook seção 3 |
| Método TRUST | Playbook seção 4 |

**Como criar:** Dentro do ChatGPT → "Explore GPTs" → "Create" → colar as instruções com todo o conteúdo acima. Leva ~1 hora para montar bem. Zero custo adicional.

**Resultado:** Qualquer pessoa do time abre o Custom GPT e gera mensagens no tom exato do HB. E os agentes automatizados usam os exemplos gerados como referência de estilo.

---

### Como ChatGPT, Claude Pro e Gemini API trabalham juntos

Vocês têm três IAs. Cada uma com um papel diferente:

```
┌─────────────────────────────────────────────────────────┐
│                    IAs do escritório                      │
├───────────────┬──────────────────┬───────────────────────┤
│  Custom GPT   │   Claude Pro     │    Gemini API         │
│  (ChatGPT)    │   (chat)         │    (automação)        │
├───────────────┼──────────────────┼───────────────────────┤
│ Tom de voz    │ Análise profunda │ Motor dos 10 agentes  │
│ do escritório │ e estratégica    │                       │
├───────────────┼──────────────────┼───────────────────────┤
│ Gera exemplos │ Brainstorm       │ ICP Scanner           │
│ de mensagens  │ jurídico         │ Diagnóstico           │
│ por ICP/canal │                  │ Mensagens             │
│               │ Revisão de       │ Briefing              │
│ Repositório   │ propostas        │ Pós-reunião           │
│ de referência │ complexas        │ Proposta              │
│ de comunicação│                  │ CRM automático        │
│               │ Análise de       │ Follow-up             │
│ Treinamento   │ contratos e      │ Eventos               │
│ do time       │ documentos       │ Inteligência          │
├───────────────┼──────────────────┼───────────────────────┤
│ Quem usa:     │ Quem usa:        │ Quem usa:             │
│ Milena, time  │ Juliana, Thiago  │ Power Automate        │
│ comercial     │ Milena           │ (automático)          │
├───────────────┼──────────────────┼───────────────────────┤
│ Custo: já pago│ Custo: já pago   │ Custo: R$ 0 (grátis)  │
└───────────────┴──────────────────┴───────────────────────┘
```

**Fluxo completo de uma mensagem automatizada:**

```
Custom GPT (uma vez)          Gemini API (automático, toda vez)
─────────────────────         ─────────────────────────────────
Gera 30 exemplos de     ──►   Prompt do agente inclui:
mensagens no tom HB           "Referência de tom: [exemplos]"
por ICP e canal               + dados do lead específico
                              ──►  Mensagem personalizada
                                   no tom HB
                              ──►  Slack para aprovação
                              ──►  Disparo
```

---

## Visão Geral da Arquitetura (versão custo zero)

```
                ┌──────────────────┐
                │  Power Automate  │  (M365 — já pago)
                │   (Orquestrador) │
                └────────┬─────────┘
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
┌────▼────┐        ┌────▼────┐        ┌─────▼─────┐
│ RD Stn  │        │  Slack  │        │  Gemini   │
│  CRM    │        │         │        │   API     │
│(já pago)│        │(já pago)│        │  (grátis) │
└─────────┘        └─────────┘        └───────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌────▼────┐  ┌─────▼─────┐
    │  Outlook  │  │  Teams  │  │ SharePoint│
    │ Calendar  │  │(transc.)│  │(docs/arq.)│
    │ (já pago) │  │(já pago)│  │ (já pago) │
    └───────────┘  └─────────┘  └───────────┘
```

---

## O que muda nos agentes

Os 10 agentes continuam exatamente os mesmos. Os prompts são idênticos. O que muda:

| Elemento | Versão completa | Versão custo zero |
|---------|----------------|------------------|
| Onde monta o fluxo | Make.com | Power Automate |
| Quem "pensa" | Claude API | Gemini API |
| Onde busca dados de empresa | Apollo.io | Tavily free + HTTP request |
| Onde gera documentos | Make → Word | Power Automate → Word (nativo) |
| Onde monitora calendário | Make → Outlook | Power Automate → Outlook (nativo) |
| Onde lê transcrições | Make → Teams/SharePoint | Power Automate → Teams (nativo) |

**Vantagens da versão custo zero:**
- Integração nativa do Power Automate com Teams, Outlook, SharePoint → menos configuração
- Sem dependência de ferramentas externas pagas
- Escala gradual — só paga quando realmente precisar

**Desvantagens:**
- Gemini pode ser inferior ao Claude em textos jurídicos sofisticados
- Sem Apollo, dados firmográficos são menos precisos
- Power Automate tem curva de aprendizado diferente do Make
- Conector com RD Station exige configuração manual (HTTP)

---

## Tabela de custos comparativa

| Item | Versão completa | Versão custo zero |
|------|----------------|------------------|
| Orquestração | Make.com Pro: R$ 90-165/mês | Power Automate: **R$ 0** (M365) |
| LLM (IA) | Claude API: R$ 200-500/mês | Gemini API: **R$ 0** (free tier) |
| Enriquecimento | Apollo.io: R$ 200-400/mês | Busca web + IA: **R$ 0** |
| Busca web | Tavily pago: R$ 140/mês | Tavily free: **R$ 0** (1.000/mês) |
| CRM | RD Station: já pago | RD Station: já pago |
| Comunicação | Slack: já pago | Slack: já pago |
| M365 | já pago | já pago |
| ChatGPT | já pago (Custom GPT = tom de voz) | já pago |
| Claude Pro | já pago (análise e brainstorm) | já pago |
| **Total adicional** | **R$ 630-1.205/mês** | **R$ 0/mês** |

---

## Quando faz sentido começar a pagar?

A ideia é: **começar com R$ 0 e só pagar quando um gargalo real aparecer.**

| Situação | Ação | Custo adicional |
|----------|------|----------------|
| Gemini não entrega qualidade suficiente para proposta jurídica | Migrar apenas o Agente de Proposta para Claude API | ~R$ 50-100/mês |
| Volume de leads cresce muito (>100/mês) e estoura o Tavily free | Tavily pago | ~R$ 140/mês |
| Precisa de dados firmográficos precisos (porte, faturamento) | Apollo.io | ~R$ 200/mês |
| Power Automate fica complexo ou atinge limite | Migrar para Make.com | ~R$ 90/mês |
| Quer qualidade máxima em todos os agentes | Claude API em tudo | ~R$ 200-500/mês |

---

## Justificativa de ROI

| Métrica | Sem agentes | Com agentes |
|---------|-------------|-------------|
| Tempo de pesquisa por lead | 30-60 min | 2 min (automático) |
| Tempo de briefing pré-reunião | 20-30 min | 0 min (entregue no Slack) |
| Tempo de pós-reunião (resumo + CRM) | 30-45 min | 5 min (revisão) |
| Tempo de criação de mensagens | 15-20 min/lead | 1 min (aprovação) |
| Leads esquecidos sem follow-up | Frequente | Zero |
| Tempo de processamento pós-evento | 2-5 dias | 2-4 horas |
| Preenchimento manual de CRM | 15-20 min/interação | Automático |

**Investimento mensal adicional: R$ 0**
**Economia mensal em tempo: 15-20 horas/semana (~R$ 3.000-5.000/mês em trabalho qualificado)**
**ROI: infinito (custo zero, economia real)**

O único investimento é **tempo de configuração** — montar os fluxos no Power Automate, configurar os prompts, testar e ajustar.

---

## Roadmap de Implementação (versão custo zero)

Mesmo roadmap da versão completa. A única diferença é trocar "Make" por "Power Automate" e "Claude API" por "Gemini API".

### Fase 1 — Quick Wins (Semanas 1-3)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 3. Personalização de Mensagens | 1 sem | Médio-Alto |
| 4. Briefing para Reunião | 3-5 dias | Alto |
| 8. Follow-up | 1-2 sem | Alto |

### Fase 2 — Core (Semanas 3-6)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 1. ICP Scanner | 2 sem | Alto |
| 2. Diagnóstico Comercial | 1 sem | Alto |
| 10. Inteligência Comercial | 2 sem | Alto |

### Fase 3 — Advanced (Semanas 6-10)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 5. Pós-Reunião | 2 sem | Muito Alto |
| 6. Proposta | 2 sem | Alto |
| 9. Eventos | 2 sem | Alto |

### Fase 4 — Full Automation (Semanas 10-14)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 7. CRM Automático | 3-4 sem | Muito Alto |

---

## Decisões Pendentes

1. **Aprovação humana vs. automação total:** Para quais ações a Milena/Juliana querem aprovar antes do disparo?
2. **WhatsApp:** Usar API oficial (WABA) ou ferramentas como Z-API?
3. **LinkedIn:** Dripify já está no stack. Integrar com Power Automate ou manter separado?
4. **Power Automate:** Alguém do time já usou? Se não, a curva de aprendizado é de 1-2 semanas.
5. **Template de proposta:** Já existe um template padrão no Word/SharePoint?
