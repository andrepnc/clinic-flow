# Clinic Flow — Automação Lead→Consulta com n8n + IA

> 🎥 **Vídeo demo:** _(disponível assim que todos os workflows estiverem concluídos)_

Clínicas perdem pacientes porque o follow-up de leads é lento e manual. O **Clinic Flow**
automatiza o funil completo: um lead preenche o formulário e, sem intervenção humana,
é qualificado por IA, registado no CRM, recebe um email personalizado com link de
marcação, confirmação e lembrete de consulta, follow-up pós-consulta — e a gerência
recebe um relatório semanal de KPIs redigido por IA.

## Estado do projeto (18 Jul 2026)

**Os 5 workflows estão concluídos, testados individualmente e publicados.**
Em falta: tratamento de erros global, teste end-to-end final e vídeo demo.
O histórico de commits reflete o progresso real, sessão a sessão.

## Stack

| Componente | Ferramenta |
|---|---|
| Orquestração | [n8n](https://n8n.io) (self-hosted, Docker) |
| IA | Google Gemini Flash (classificação + redação) |
| CRM | HubSpot |
| Marcações | Cal.com (API + webhooks) |
| Email | Gmail API |
| Log / KPIs | Google Sheets |

## Arquitetura

Cinco workflows independentes que comunicam por webhooks e estado partilhado
(CRM/Sheets) — ver [docs/architecture.md](docs/architecture.md) para o diagrama
completo e as decisões de design.

1. ✅ **Lead Intake** — formulário → Gemini classifica serviço/urgência (JSON estrito) → HubSpot + log
2. ✅ **Follow-up** — Gemini redige email personalizado ao lead com link de marcação
3. ✅ **Booking** — webhook do Cal.com → confirmação + lembrete 24h antes (scan horário idempotente)
4. ✅ **Pós-consulta** — agradecimento e pedido de feedback no dia seguinte
5. ✅ **Relatório semanal** — KPIs calculados em n8n, resumo executivo redigido por Gemini

## Prompt Engineering

Todos os prompts estão documentados em [prompts/](prompts/) — objetivo, input,
schema de output, guardrails (sem diagnósticos médicos, sem inventar preços/moradas)
e notas de design. Os workflows exportados estão em [workflows/](workflows/).

## Como correr

1. `docker compose up -d` (ver [docker-compose.yml](docker-compose.yml))
2. Seguir o guia de credenciais e túnel em [docs/setup.md](docs/setup.md)
3. Importar os JSONs de `workflows/` no n8n e associar as credenciais

---

_Projeto em construção — última atualização 18 Jul 2026._
