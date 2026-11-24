# SAP API Management + CPI Order Approval

Projeto desenvolvido com SAP CAP, SAP API Management e SAP CPI (Integration Suite) para demonstrar um fluxo completo de aprovação de pedidos utilizando políticas de API, KVM, JavaScript e integração com iFlows.

---

# Arquitetura

```text
SAP Fiori Elements
        ↓
SAP CAP
        ↓
SAP API Management
        ↓
SAP CPI (iFlow)
        ↓
Resposta
```

---

# Fluxo de Aprovação

## Pedido aprovado

Entrada:

```json
{
  "customer": "Guilherme",
  "amount": 200
}
```

Resultado:

```json
{
  "customer": "Guilherme",
  "amount": "200",
  "status": "APROVADO"
}
```

---

## Pedido pendente

Entrada:

```json
{
  "customer": "Guilherme",
  "amount": 1500
}
```

Resultado:

```json
{
  "customer": "Guilherme",
  "amount": "1500",
  "status": "PENDENTE"
}
```

---

# Tecnologias Utilizadas

- SAP CAP (Cloud Application Programming Model)
- SAP Fiori Elements
- SAP API Management
- SAP Cloud Integration (CPI)
- SAP BTP Integration Suite
- JavaScript Policies
- OData V4
- SQLite

---

# SAP API Management

## API Provider

Backend registrado:

```text
CPIProvider
```

Responsável por direcionar requisições para o ambiente SAP CPI.

---

## API Proxy

Proxy criado:

```text
OrderApproval
```

Endpoint:

```text
https://f7e61906trial-trial.integrationsuitetrial-apim.us10.hana.ondemand.com/f7e61906trial/order
```

---

# Policies Implementadas

## Verify API Key

Validação dos consumidores da API através de Consumer Key.

Header obrigatório:

```http
apikey: <consumer_key>
```

---

## JSON Threat Protection

Proteção contra payloads maliciosos.

Restrições implementadas:

```text
Container Depth
Array Element Count
Object Entry Count
String Value Length
```

---

## Spike Arrest

Limitação de requisições:

```text
100 requests/minute
```

Objetivo:

```text
Proteger o backend contra abuso ou excesso de chamadas.
```

---

## Extract Variables

Extrai valores do JSON recebido.

Payload:

```json
{
  "customer":"Guilherme",
  "amount":1500
}
```

Variáveis criadas:

```text
purchase.customer
purchase.amount
```

---

## Key Value Map (KVM)

KVM criada:

```text
purchaseConfig
```

Configuração:

```text
approvalLimit = 1000
```

Permite alteração do limite de aprovação sem necessidade de alterar código.

---

## JavaScript Policy

Regra de aprovação implementada:

```javascript
var amount = Number(
    context.getVariable("purchase.amount")
);

var approvalLimit = Number(
    context.getVariable(
        "purchase.config.approvalLimit"
    )
);

var status = "";

if (amount > approvalLimit) {
    status = "PENDENTE";
} else {
    status = "APROVADO";
}

context.setVariable(
    "purchase.status",
    status
);
```

---

## Assign Message

Responsável por enriquecer o payload antes do envio ao CPI.

Entrada:

```json
{
  "customer":"Guilherme",
  "amount":1500
}
```

Saída:

```json
{
  "customer":"Guilherme",
  "amount":"1500",
  "status":"PENDENTE"
}
```

---

# Segurança

A API utiliza duas camadas de autenticação.

## OAuth Bearer Token

```http
Authorization: Bearer <access_token>
```

---

## API Key

```http
apikey: <consumer_key>
```

---

## Cenários

Sem Token:

```text
401 Unauthorized
```

Sem API Key:

```text
401 Unauthorized
```

Token + API Key:

```text
200 OK
```

---

# Integração CAP

O serviço CAP consome o endpoint do API Management utilizando:

```javascript
const response = await fetch(APIM_URL, {
    method: 'POST',
    headers: {
        Authorization: `Bearer ${access_token}`,
        apikey: api_key,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        customer: order.customer,
        amount: Number(order.amount)
    })
});
```

---

# Integração CPI

O iFlow recebe:

```json
{
  "customer":"Guilherme",
  "amount":"1500",
  "status":"PENDENTE"
}
```

e devolve a resposta para o CAP.

---

# Fiori Elements

Funcionalidades implementadas:

- List Report
- Object Page
- Criação de pedidos
- Persistência local SQLite
- Status com criticidade visual
- Integração com API Management
- Integração com CPI

---

# Status Suportados

| Status | Criticidade |
|----------|----------|
| APROVADO | 🟢 Verde |
| PENDENTE | 🟡 Amarelo |

---

# Estrutura do Projeto

```text
purchase_approve
│
├── app/
│   └── purchaseorders/
│
├── db/
│   └── schema.cds
│
├── srv/
│   ├── service.cds
│   └── service.js
│
├── package.json
├── .env
└── README.md
```

---

# Execução

Instalar dependências:

```bash
npm install
```

Executar aplicação:

```bash
cds watch
```

Acessar:

```text
http://localhost:4004
```

---

# Regras de Negócio

Limite configurado na KVM:

```text
approvalLimit = 1000
```

Regra:

```text
amount <= approvalLimit
→ APROVADO

amount > approvalLimit
→ PENDENTE
```

Exemplos:

```text
200
↓
APROVADO
```

```text
1500
↓
PENDENTE
```

---

# Objetivo

Demonstrar uma arquitetura completa utilizando:

- SAP CAP
- SAP Fiori Elements
- SAP API Management
- SAP CPI
- OAuth
- API Key
- KVM
- JavaScript Policies
- Assign Message
- Governança e Segurança de APIs

com processamento ponta a ponta entre Frontend, Backend, API Management e Integration Suite.
