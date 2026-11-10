# AI Customer Support Automation

An AI-assisted customer support ticket classification and routing workflow built with **n8n** and **Ollama**. It processes incoming support requests, identifies their category, and prepares the appropriate team notification or customer response.

## Overview

Support teams receive requests about billing, technical issues, account access, and general enquiries. Manually sorting these requests can take time.

This workflow automates the initial classification and routing process:

- Receives support tickets through an HTTP webhook.
- Uses a locally hosted LLM to classify incoming requests.
- Applies keyword-based priority rules to improve routing consistency.
- Routes tickets to the appropriate support category.
- Generates a customer email draft for general enquiries.
- Prepares structured notifications for Billing, Technical, and Account teams.
- Returns a JSON response containing the processing result.

## Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Ollama | Local LLM execution |
| Llama 3.2 (3B) | AI-based ticket classification |
| JavaScript | Classification logic and response processing |
| HTTP / JSON | Webhook input and API response |
| Docker | Local n8n environment |

## Workflow Architecture
![Workflow Overview](workflow-overview.png)
```text
Incoming Support Ticket
          |
          v
     Webhook (POST)
          |
          v
     Mock CRM Data
          |
          v
   Ollama - Llama 3.2
   AI Classification
          |
          v
    Parse AI Response
          |
          v
    Route by Category
          |
    +-----+------+-------+------+
    |            |       |      |
 Technical    Billing  General Account
    |            |       |      |
    v            v       v      v
 Tech Alert  Billing   Customer Account
             Alert     Response Alert
    |            |       |      |
    +------------+-------+------+
                 |
                 v
       Build Final Response
                 |
                 v
          JSON Webhook Response
```

## Ticket Categories

| Category | Example requests | Routing |
|---|---|---|
| Account | Login problems, forgotten passwords, account access | Account Team |
| Billing | Payments, duplicate charges, refunds, invoices | Billing Team |
| Technical | Application crashes, bugs, upload errors | Technical Team |
| General | Opening hours and other general enquiries | Customer email draft |

The workflow uses AI classification alongside deterministic keyword-based rules. Account-related keywords receive priority, followed by billing and technical keywords. Other requests are classified as General.

## Setup and Installation

### Prerequisites

- Docker Desktop
- n8n
- Ollama
- Llama 3.2 (3B)

### 1. Install Ollama and download the model

Install Ollama from [ollama.com](https://ollama.com/).

Then run:

```bash
ollama pull llama3.2:3b
```

### 2. Start n8n using Docker

```bash
docker run -d --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Europe/Berlin" -e TZ="Europe/Berlin" -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n:latest
```

Open n8n at:

```text
http://localhost:5678
```

### 3. Import the workflow

1. Download `customer-support-ai-workflow.json` from this repository.
2. Open n8n.
3. Import the downloaded workflow JSON.
4. Save the workflow.

### 4. Configure the Ollama connection

The HTTP Request node uses:

```text
http://host.docker.internal:11434/api/generate
```

This endpoint is configured for a local Windows and Docker setup where Ollama runs on the host machine and n8n runs inside Docker.

If you use another operating system or deployment environment, adjust the endpoint accordingly.

### 5. Test the workflow

1. Open the imported workflow in n8n.
2. Click **Execute Workflow** to listen for a test webhook request.
3. Send a POST request to:

```text
http://localhost:5678/webhook-test/support-ticket
```

Example PowerShell request:

```powershell
$body = @{
    customer_email = "customer@example.com"
    customer_name = "John Smith"
    subject = "Unable to access my account"
    message = "I forgot my password and cannot log in."
    source = "website"
    customer_tier = "standard"
    purchase_count = 2
    last_contact = "2026-11-09"
} | ConvertTo-Json

Invoke-RestMethod `
    -Uri "http://localhost:5678/webhook-test/support-ticket" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

The test webhook is intended for manual testing and requires the workflow to be listening. For a production webhook URL, activate the workflow and use its production webhook endpoint.

## Testing and Validation

The workflow was tested end-to-end with four different support scenarios.

| Test scenario | Expected category | Result |
|---|---|---|
| Support opening hours | General | Passed |
| Duplicate subscription payment and refund | Billing | Passed |
| Application crash during document upload | Technical | Passed |
| Forgotten password and login issue | Account | Passed |

Each successful test returned a JSON response containing the ticket ID, category, routing result, explanation, and processing timestamp.

## Example Response

```json
{
  "status": "success",
  "ticket_id": "TICKET-20261110-152015",
  "route": "account_team",
  "category": "Account",
  "explanation": "The customer needs help accessing their account.",
  "notification": "ACCOUNT SUPPORT TICKET...",
  "customer_name": "Sarah Ahmed",
  "customer_email": "customer@example.com",
  "processed_at": "2026-11-10T15:20:15.000+01:00"
}
```

The values above illustrate the response structure; actual values depend on the incoming ticket.

## Current Scope and Limitations

- The workflow generates email drafts and team notification text; it does not send actual emails or external team alerts.
- CRM information is represented by incoming webhook data and a mock CRM data-processing node. No external CRM is connected.
- Tickets are not persisted in a database.
- Classification depends on the local LLM and the configured keyword-based routing rules.
- The workflow currently runs in a local development environment.

## Future Improvements

- Integrate Gmail or Outlook for automated customer replies.
- Connect Slack or Microsoft Teams for team notifications.
- Integrate a real CRM system.
- Store ticket history in PostgreSQL.
- Add priority and sentiment classification.
- Introduce human approval for sensitive or high-impact requests.
- Add error handling, logging, and monitoring.

## Project Structure

```text
ai-customer-support-automation/
│
├── customer-support-ai-workflow.json
└── README.md
```

## Author

**Abdul Wasay**

- GitHub: [wasaysiyal](https://github.com/wasaysiyal)
- LinkedIn: [Abdul Wasay](https://www.linkedin.com/in/abdul-wasay-70a672421/)
