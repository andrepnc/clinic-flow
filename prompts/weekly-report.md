# Prompt: Relatório Semanal de KPIs (WF5)

## Objetivo
Transformar os números agregados da semana (calculados em n8n a partir do Google Sheets) num resumo executivo legível para a gerência da clínica.

## Input
JSON com os KPIs já calculados pelo n8n (o modelo NÃO calcula nada):

```json
{
  "periodo": "13-19 Jul 2026",
  "total_leads": 24,
  "leads_por_servico": {"fisioterapia": 10, "nutricao": 6, "psicologia": 5, "avaliacao_geral": 3},
  "leads_por_urgencia": {"alta": 4, "media": 12, "baixa": 8},
  "consultas_marcadas": 9,
  "taxa_conversao": "37.5%",
  "semana_anterior": {"total_leads": 18, "consultas_marcadas": 6}
}
```

## Output esperado
Texto (não JSON): 2 parágrafos + bullets, pronto para corpo de email.

## Prompt (System)

```
És analista de marketing da Clínica Vitalis. Escreves o resumo semanal para a
gerência com base em KPIs já calculados. Escreves em português europeu.

Regras:
- Usa APENAS os números fornecidos. Nunca inventes, estimes ou recalcules valores.
- Estrutura: 1 parágrafo com a leitura geral da semana (incluindo comparação com a
  semana anterior) → lista de bullets com os KPIs principais → 1 parágrafo final
  com uma recomendação prática baseada nos dados (ex.: reforçar resposta a leads
  de urgência alta, investir no serviço com mais procura).
- Nos bullets usa hífen (-) no início de cada linha, nunca asteriscos.
- Máximo 180 palavras. Tom profissional e direto, sem jargão.
- Responde só com o texto do email, sem assunto nem saudação inicial.
```

## Prompt (User)

```
KPIs da semana:
{{ JSON.stringify($json.kpis, null, 2) }}
```

## Notas de design
- Decisão de arquitetura: os cálculos são feitos em nós Code/Aggregate do n8n, não
  pelo LLM — números têm de ser exatos e auditáveis; o modelo só redige.

## Iterações reais (testes de 18 Jul 2026)
- Primeira versão devolvia os bullets com asteriscos markdown (`*   `) num email
  de texto simples. A regra explícita "usa hífen (-), nunca asteriscos" resolveu
  à primeira tentativa.
- Com `semana_anterior` a zeros (primeira semana de atividade), o modelo interpretou
  corretamente como "início da atividade" em vez de inventar variações percentuais —
  validação de que fornecer números crus + regra "nunca recalcules" é suficiente.
- Verificação numérica: todos os valores do email correspondem exatamente aos KPIs
  calculados pelo n8n; nenhum número inventado ou recalculado.
