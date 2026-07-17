# Setup — checklist do Dia 1 (sexta)

Ordem recomendada. Cada passo tem um critério de verificação — não avances sem ele passar.

## 1. Contas e chaves (~1h)

- [ ] **HubSpot Free** — criar conta em hubspot.com (CRM Free).
  - Settings → Integrations → Private Apps → criar app "n8n" com scopes
    `crm.objects.contacts.read` e `crm.objects.contacts.write` → copiar o token.
  - ✔ Verificar: token copiado e guardado.
- [ ] **Cal.com Free** — criar conta em cal.com.
  - Criar event type **"Consulta de Avaliação (30 min)"**.
  - Settings → Developer → API Keys → criar chave.
  - ✔ Verificar: consegues abrir o link público de marcação no browser.
- [ ] **Gemini API** — aistudio.google.com → Get API key.
  - ✔ Verificar: chave criada (o free tier do modelo Flash chega para todo o projeto).
- [ ] **Google Cloud** (para Gmail + Sheets no n8n) — console.cloud.google.com:
  - Criar projeto "clinic-flow" → ativar **Gmail API** e **Google Sheets API**.
  - OAuth consent screen: tipo External, adicionar o teu email como test user.
  - Credentials → OAuth Client ID (Web application) → redirect URI do n8n
    (o n8n mostra o URI exato no ecrã da credencial: `http://localhost:5678/rest/oauth2-credential/callback`).
  - ✔ Verificar: Client ID + Secret copiados.

## 2. Túnel para webhooks (~30 min)

O Cal.com precisa de alcançar o n8n local. Opção recomendada: **cloudflared** (grátis, sem conta):

```powershell
winget install Cloudflare.cloudflared
cloudflared tunnel --url http://localhost:5678
```

Copia o URL `https://<aleatorio>.trycloudflare.com` gerado e arranca o n8n com ele:

```powershell
$env:WEBHOOK_URL = "https://<aleatorio>.trycloudflare.com/"
docker compose up -d
```

- ⚠ O URL do trycloudflare muda a cada arranque do túnel — se reiniciares o túnel,
  atualiza `WEBHOOK_URL` e o webhook no Cal.com.
- ✔ Verificar: abrir `https://<aleatorio>.trycloudflare.com` no browser mostra o n8n.

## 3. Credenciais no n8n (~1h)

Criar em Credentials:

- [ ] **Google Gemini (PaLM) API** — colar a API key.
- [ ] **HubSpot App Token** — colar o Private App token.
- [ ] **Gmail OAuth2** — Client ID/Secret do passo 1 → "Sign in with Google".
- [ ] **Google Sheets OAuth2** — reutiliza o mesmo Client ID/Secret.
- ✔ Verificar cada uma com um workflow descartável de 1 nó (ex.: Gmail "Get labels",
  Sheets "Read", Gemini com prompt "diz olá", HubSpot "Get all contacts").

## 4. Google Sheet de log (~15 min)

- [ ] Criar sheet **"clinic-flow-log"** com 3 abas:
  - `leads`: timestamp | nome | email | telefone | servico | urgencia | resumo
  - `bookings`: timestamp | nome | email | data_consulta | status | lembrete_enviado | followup_enviado
  - `emails`: timestamp | tipo | destinatario | assunto
- ✔ Verificar: nó Sheets no n8n lê as abas.

## 5. Webhook do Cal.com (quando construíres o WF3)

- Cal.com → Settings → Developer → Webhooks → novo webhook:
  - URL: o Production URL do nó Webhook do WF3 (baseado no `WEBHOOK_URL` do túnel).
  - Evento: **Booking Created**.
- ✔ Verificar: marcar uma consulta de teste dispara a execução no n8n.

## Dados de teste

Usar aliases Gmail (`o-teu-email+paciente1@gmail.com`, `+paciente2`, etc.) para simular
pacientes fictícios — todos chegam à mesma caixa de entrada, sem precisar de contas extra.
