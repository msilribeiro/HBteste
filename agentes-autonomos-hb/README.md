# Agentes Autônomos HB — Arquitetura e Implementação

## Stack Recomendada

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Orquestração | **Make (Integromat)** | Já conectado ao ambiente HB. Dispara agentes, conecta CRM, Slack, Calendar |
| LLM | **Claude API (Sonnet)** | Qualidade de texto em português, raciocínio jurídico, custo acessível |
| CRM | **RD Station CRM** | Já é a fonte de verdade do playbook. API disponível via MCP |
| Comunicação | **Slack** | Já usado para alertas entre etapas |
| Enriquecimento | **Apollo/Clearbit API** ou **scraping LinkedIn** | Dados firmográficos |
| Busca web | **Tavily / SerpAPI** | Para pesquisa de empresas em tempo real |
| Storage | **Google Sheets / SharePoint** | Já usado pelo time |
| Transcrição | **AssemblyAI / Whisper** | Para o agente pós-reunião |

---

## Visão Geral da Arquitetura

```
                    ┌─────────────┐
                    │   MAKE.com   │  (Orquestrador)
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ RD Stn  │       │  Slack  │       │ Claude  │
   │  CRM    │       │         │       │   API   │
   └─────────┘       └─────────┘       └─────────┘
```

Cada agente é um **cenário no Make** que:
1. Recebe um trigger (webhook, schedule, ou mudança no CRM)
2. Coleta dados de fontes externas (web, LinkedIn, CRM)
3. Envia prompt estruturado para Claude API
4. Distribui a saída (CRM, Slack, e-mail, Google Docs)

---

## Os 10 Agentes

### 1. ICP Scanner

**Trigger:** Webhook manual ou novo lead no RD Station

**Fluxo Make:**
```
Webhook/RD trigger
  → Buscar site da empresa (HTTP request)
  → Buscar LinkedIn (Apollo API ou scraping)
  → Buscar notícias recentes (Tavily/SerpAPI)
  → Montar prompt com contexto
  → Claude API: classificar ICP + gerar briefing
  → Salvar resultado no RD Station (campo customizado)
  → Notificar Milena via Slack
```

**Prompt base para Claude:**
```
Você é analista comercial do escritório Hartmann Burmeister, especializado em Direito Empresarial para startups e empresas inovadoras.

Com base nas informações abaixo, classifique a empresa em um dos ICPs:
- Startup (Launch / Growth / Shield)
- Empresa Tradicional com olhar de inovação
- Administradora/Garantidora de condomínio

DADOS DA EMPRESA:
- Nome: {{empresa}}
- Site: {{site_content}}
- LinkedIn: {{linkedin_data}}
- Notícias recentes: {{news}}

RETORNE em formato estruturado:
1. ICP identificado
2. Porte estimado (colaboradores, faturamento se disponível)
3. Sinais encontrados (captação, contratações, expansão, problemas)
4. Principais riscos jurídicos prováveis
5. Dores prováveis (baseado no ICP)
6. Oferta sugerida (Diagnóstico, Launch Pack, Growth, Shield, Recuperação de Crédito)
7. Probabilidade de fit (0-100%)
8. Justificativa da classificação
```

**Campos customizados no RD Station:**
- `icp_classificacao`
- `icp_score`
- `icp_dores_provaveis`
- `icp_oferta_sugerida`
- `icp_briefing_completo`

**Esforço:** Médio (~2 semanas)
**Impacto:** Alto — elimina 80% do tempo de pesquisa da Milena

---

### 2. Diagnóstico Comercial

**Trigger:** Lead qualificado no RD Station (mudança de etapa)

**Fluxo Make:**
```
RD Station: lead mudou para "Qualificado"
  → Buscar dados do lead no RD
  → Buscar dados do ICP Scanner (campo customizado)
  → Buscar site da empresa
  → Buscar LinkedIn dos decisores
  → Claude API: gerar diagnóstico + perguntas
  → Salvar no RD Station
  → Criar doc no Google Docs
  → Notificar Juliana via Slack com link do doc
```

**Prompt base:**
```
Você é consultor jurídico estratégico do Hartmann Burmeister.

Com base no perfil abaixo, gere um diagnóstico comercial pré-reunião.

EMPRESA: {{empresa}}
ICP: {{icp}}
PORTE: {{porte}}
SINAIS: {{sinais}}
INFORMAÇÕES COLETADAS: {{info_milena}}

RETORNE:
1. Hipóteses de dor (3-5, ordenadas por probabilidade)
2. Riscos jurídicos prováveis
3. Perguntas recomendadas para a reunião (estilo BANT suave, conforme playbook HB)
4. Pontos de conexão para rapport
5. Oferta inicial recomendada
6. Argumentos de valor (sem juridiquês, linguagem de negócios)
```

**Esforço:** Baixo (~1 semana, reutiliza dados do ICP Scanner)
**Impacto:** Alto — Juliana entra na reunião com diagnóstico pronto

---

### 3. Personalização de Mensagens

**Trigger:** Novo lead classificado ou solicitação manual via Slack

**Fluxo Make:**
```
Trigger (Slack command ou RD)
  → Buscar dados do lead + ICP
  → Claude API: gerar mensagens personalizadas
  → Devolver via Slack (ou salvar no RD como nota)
```

**Prompt base:**
```
Você é o time comercial do Hartmann Burmeister. Gere mensagens personalizadas para o lead abaixo.

LEAD: {{nome}}
EMPRESA: {{empresa}}
ICP: {{icp}}
CANAL DE ORIGEM: {{canal}}
DORES PROVÁVEIS: {{dores}}
CONTEXTO: {{contexto}} (ex: evento, LinkedIn, indicação)

GERE mensagens para cada canal, seguindo o tom de voz HB (consultivo, humano, sem juridiquês, sem venda direta):

1. LinkedIn (pedido de conexão) — máx 300 caracteres
2. LinkedIn (primeira mensagem pós-conexão) — consultivo, sem CTA de venda
3. WhatsApp (primeira abordagem) — leve, com CTA para conversa
4. E-mail (primeiro contato) — assunto curto + corpo consultivo
5. Follow-up WhatsApp (dia 3) — com material de valor ou insight
6. Follow-up e-mail (dia 5) — aprofundar dor sem pressão

REGRAS:
- Nunca usar "gostaríamos de oferecer nossos serviços"
- Nunca usar linguagem genérica
- Referenciar algo real da empresa
- Usar placeholders apenas se dado não disponível
```

**Esforço:** Baixo (~1 semana)
**Impacto:** Médio-Alto — elimina criação manual de mensagens

---

### 4. Briefing para Reunião

**Trigger:** Reunião agendada no Google Calendar (15 min antes)

**Fluxo Make:**
```
Google Calendar: evento com tag "HB Reunião" em 15 min
  → Buscar lead no RD Station pelo nome/empresa
  → Buscar dados do ICP Scanner
  → Buscar histórico de contatos no RD
  → Buscar diagnóstico comercial (se existir)
  → Claude API: compilar briefing executivo
  → Enviar via Slack para Juliana
  → Enviar por e-mail como backup
```

**Prompt base:**
```
Compile um briefing executivo para reunião comercial.

LEAD: {{nome}} — {{empresa}}
ORIGEM: {{origem}}
ICP: {{icp}}
DORES PROVÁVEIS: {{dores}}
HISTÓRICO DE CONTATOS: {{historico}}
DIAGNÓSTICO: {{diagnostico}}

FORMATO:
- Lead e empresa (1 linha)
- Origem e canal
- ICP e porte
- Dor provável principal
- Histórico resumido (contatos, reuniões, propostas)
- Objetivo da reunião
- Perguntas-chave para a reunião
- Alertas (objeções anteriores, riscos, pendências)
```

**Esforço:** Baixo (~3-5 dias)
**Impacto:** Alto — passagem de bastão automática

---

### 5. Pós-Reunião

**Trigger:** Gravação da reunião disponível (Google Meet/Zoom)

**Fluxo Make:**
```
Webhook: gravação disponível
  → Enviar áudio para AssemblyAI/Whisper (transcrição)
  → Claude API: analisar transcrição
  → Salvar resumo no RD Station
  → Criar tarefas no RD Station
  → Notificar via Slack
  → Atualizar estágio do lead se necessário
```

**Prompt base:**
```
Você é assistente comercial do Hartmann Burmeister. Analise a transcrição da reunião abaixo.

TRANSCRIÇÃO: {{transcricao}}
LEAD: {{lead}}
ICP: {{icp}}

RETORNE:
1. Resumo executivo (máx 5 linhas)
2. Dores confirmadas pelo cliente
3. Dores não confirmadas / descartadas
4. Próximos passos acordados
5. Escopo sugerido (com base nas dores confirmadas)
6. Objeções levantadas
7. Riscos percebidos
8. Temperatura do lead (frio / morno / quente / muito quente)
9. Tarefas para CRM:
   - Para quem
   - O que fazer
   - Prazo sugerido
10. Oportunidades de upsell/cross-sell identificadas
```

**Esforço:** Médio (~2 semanas, integração de transcrição)
**Impacto:** Muito Alto — maior economia de tempo individual

---

### 6. Proposta

**Trigger:** Solicitação via Slack ou mudança de etapa no RD ("Proposta")

**Fluxo Make:**
```
RD: lead em etapa "Proposta"
  → Buscar dados completos do lead
  → Buscar diagnóstico e pós-reunião
  → Claude API: gerar proposta
  → Criar Google Doc com template HB
  → Notificar Thiago via Slack para revisão
```

**Prompt base:**
```
Gere uma proposta comercial para o Hartmann Burmeister.

EMPRESA: {{empresa}}
ICP: {{icp}}
DORES CONFIRMADAS: {{dores}}
ESCOPO VALIDADO: {{escopo}}
FAIXA DE PREÇO: {{faixa}}

ESTRUTURA DA PROPOSTA:
1. Contexto e entendimento do cenário
2. Diagnóstico das necessidades identificadas
3. Solução proposta (escopo detalhado)
4. Metodologia de trabalho
5. Investimento e condições
6. Próximos passos

TOM: Consultivo, claro, sem juridiquês. Linguagem de negócios.
Não usar "gostaríamos de oferecer". Focar em valor e resultado.
```

**Esforço:** Médio (~2 semanas, template + ajustes)
**Impacto:** Alto — Thiago apenas revisa ao invés de criar do zero

---

### 7. CRM Automático

**Trigger:** Múltiplos — após reunião, após e-mail, após WhatsApp, schedule diário

**Fluxo Make:**
```
Cenário A — Pós-reunião:
  Resultado do Agente Pós-Reunião
    → Atualizar estágio no RD
    → Criar tarefas no RD
    → Atualizar campos customizados

Cenário B — Diário (schedule):
  Buscar deals ativos no RD
    → Para cada deal sem atividade recente:
      → Criar alerta no Slack
      → Sugerir próxima ação

Cenário C — E-mail/WhatsApp (futuro):
  Integração com e-mail/WhatsApp
    → Claude API: extrair informações relevantes
    → Atualizar RD automaticamente
```

**Campos auto-atualizados:**
- Estágio do funil
- Última interação
- Próxima ação
- Objeções registradas
- Temperatura do lead
- Resumo do último contato

**Esforço:** Alto (~3-4 semanas, múltiplos cenários)
**Impacto:** Muito Alto — elimina preenchimento manual do CRM

---

### 8. Follow-up

**Trigger:** Schedule diário (manhã)

**Fluxo Make:**
```
Schedule: 8h todos os dias úteis
  → Buscar deals no RD Station
  → Filtrar:
    - Sem resposta há 5+ dias
    - Proposta enviada há 10+ dias
    - Reunião sem retorno há 15+ dias
  → Para cada lead:
    → Claude API: gerar mensagem de follow-up personalizada
    → Enviar sugestão via Slack para Milena
    → Milena aprova → disparo automático (WhatsApp/e-mail)
```

**Prompt base:**
```
Gere follow-up para o lead abaixo.

LEAD: {{nome}} — {{empresa}}
ÚLTIMO CONTATO: {{ultimo_contato}}
CANAL: {{canal}}
ESTÁGIO: {{estagio}}
CONTEXTO: {{contexto_ultimo_contato}}
OBJEÇÕES: {{objecoes}}

TIPO DE FOLLOW-UP: {{tipo}} (sem resposta / proposta pendente / pós-reunião)

REGRAS:
- Não pressionar
- Referenciar o último contato
- Oferecer valor adicional (insight, material, caso similar)
- Tom consultivo HB
- Máx 3 linhas para WhatsApp, 5 linhas para e-mail
```

**Esforço:** Baixo-Médio (~1-2 semanas)
**Impacto:** Alto — nenhum lead esquecido

---

### 9. Eventos

**Trigger:** Upload de planilha (webhook) ou integração com ferramenta de eventos

**Fluxo Make:**
```
Webhook: planilha de contatos do evento
  → Para cada contato:
    → Limpar e padronizar dados
    → Enriquecer via Apollo/LinkedIn
    → Claude API: classificar ICP + priorizar
    → Criar lead no RD Station
    → Gerar mensagem de abordagem pós-evento
  → Gerar relatório consolidado
  → Enviar via Slack
```

**Prompt base:**
```
Classifique e priorize o contato captado no evento {{evento}}.

CONTATO: {{nome}}
EMPRESA: {{empresa}}
CARGO: {{cargo}}
DADOS ENRIQUECIDOS: {{dados}}

RETORNE:
1. ICP provável
2. Prioridade (A/B/C)
3. Dor provável
4. Mensagem de abordagem pós-evento (WhatsApp, máx 3 linhas)
5. Justificativa da priorização
```

**Esforço:** Médio (~2 semanas)
**Impacto:** Alto — transforma evento em pipeline qualificado em horas, não semanas

---

### 10. Inteligência Comercial

**Trigger:** Schedule semanal (sexta-feira)

**Fluxo Make:**
```
Schedule: sexta 14h
  → Buscar todos os deals do RD (últimos 30 dias)
  → Buscar deals ganhos e perdidos
  → Buscar objeções registradas
  → Claude API: análise de padrões
  → Gerar relatório
  → Enviar via Slack para Juliana e Thiago
  → Salvar no Google Sheets (histórico)
```

**Prompt base:**
```
Analise os dados comerciais do Hartmann Burmeister da última semana.

DEALS ATIVOS: {{deals_ativos}}
DEALS GANHOS: {{ganhos}}
DEALS PERDIDOS: {{perdidos}}
OBJEÇÕES REGISTRADAS: {{objecoes}}
MOTIVOS DE PERDA: {{motivos_perda}}

RETORNE:
1. Resumo da semana (pipeline, conversões, perdas)
2. Top 3 objeções mais frequentes
3. ICP com maior taxa de conversão
4. Canal mais eficiente (LinkedIn, WhatsApp, evento, indicação)
5. Motivos de perda mais comuns
6. Temas/dores mais citados pelos leads
7. Recomendações de ação para próxima semana
8. Alertas (deals parados, SLAs estourados, tendências negativas)
```

**Esforço:** Médio (~2 semanas)
**Impacto:** Alto — melhoria contínua automática

---

## Roadmap de Implementação

### Fase 1 — Quick Wins (Semanas 1-3)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 3. Personalização de Mensagens | 1 sem | Médio-Alto |
| 4. Briefing para Reunião | 3-5 dias | Alto |
| 8. Follow-up | 1-2 sem | Alto |

**Por quê primeiro:** Baixo esforço, resultado imediato, não depende de integrações complexas.

### Fase 2 — Core (Semanas 3-6)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 1. ICP Scanner | 2 sem | Alto |
| 2. Diagnóstico Comercial | 1 sem | Alto |
| 10. Inteligência Comercial | 2 sem | Alto |

**Por quê segundo:** Constrói a base de dados que alimenta os outros agentes.

### Fase 3 — Advanced (Semanas 6-10)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 5. Pós-Reunião | 2 sem | Muito Alto |
| 6. Proposta | 2 sem | Alto |
| 9. Eventos | 2 sem | Alto |

**Por quê terceiro:** Requer integração de transcrição e templates mais complexos.

### Fase 4 — Full Automation (Semanas 10-14)
| Agente | Esforço | Impacto |
|--------|---------|---------|
| 7. CRM Automático | 3-4 sem | Muito Alto |

**Por quê último:** Depende de todos os outros agentes funcionando para ter dados de qualidade.

---

## Custos Estimados (mensal)

| Item | Custo estimado |
|------|---------------|
| Claude API (Sonnet) | R$ 200-500/mês |
| Make.com (Pro) | R$ 150-300/mês |
| Apollo.io (enriquecimento) | R$ 200-400/mês |
| AssemblyAI (transcrição) | R$ 100-200/mês |
| Tavily/SerpAPI (busca) | R$ 50-100/mês |
| **Total** | **R$ 700-1.500/mês** |

Comparado ao custo de uma pessoa fazendo essas tarefas manualmente (~R$ 4.000-6.000/mês), o ROI é claro.

---

## Decisões Pendentes

1. **Aprovação humana vs. automação total:** Para quais ações a Milena/Juliana querem aprovar antes do disparo? (Recomendação: aprovar mensagens no início, depois ir liberando conforme confiança cresce)
2. **WhatsApp:** Usar API oficial (WABA) ou ferramentas como Z-API? A API oficial é mais confiável mas mais cara.
3. **LinkedIn:** Dripify já está no stack. Integrar com Make ou manter separado?
4. **Transcrição:** Google Meet grava nativamente. Qual ferramenta de transcrição preferem?
5. **Template de proposta:** Já existe um template padrão no Google Docs/Word?
