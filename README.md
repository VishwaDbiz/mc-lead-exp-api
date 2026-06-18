# mc-lead-exp-api

## Overview

This is a Mule Experience API for Lead management. It exposes REST endpoints for creating and retrieving Salesforce Leads (`POST /api/leads`, `GET /api/leads/{leadId}`) and includes a **Salesforce Lead Change Data Capture (CDC) listener** that sends email notifications whenever a Lead record is created, updated, or deleted in Salesforce.

---

## Project Structure

```
src/main/mule/
  mc-lead-exp.xml          — Experience API HTTP flows (APIKit)
  global.xml               — Global API Autodiscovery config
  common.xml               — Shared sub-flows
  lead-sf-listener.xml     — Salesforce CDC listener + email notification flow

src/main/resources/
  config.properties        — Experience API properties (client_id, client_secret, PRC URL)
  config.yaml              — Salesforce + Email connection base properties
  dev.yaml                 — Dev environment-specific property overrides
```

---

## Salesforce Lead Change Data Capture (CDC) — Setup Guide

### Prerequisites

- Salesforce org with API access enabled
- A Salesforce user with the **Streaming API** permission
- Lead Change Data Capture must be enabled in Salesforce Setup

---

### Step 1: Enable Change Data Capture for the Lead Object

1. Log in to your **Salesforce org** as an Administrator.
2. Navigate to **Setup** → search for **Change Data Capture** in the Quick Find box.
3. Under **Entities Available for Change Data Capture**, locate **Lead** in the left column.
4. Select **Lead** and click the **›** (right arrow) to move it to the **Selected Entities** column.
5. Click **Save**.

> ✅ Once saved, Salesforce will publish `LeadChangeEvent` messages to the `/data/LeadChangeEvent` streaming channel for every Lead create, update, delete, and undelete.

---

### Step 2: Verify Streaming API User Permissions

The Salesforce user configured in `config.yaml` must have:

- **API Enabled** — System Permissions
- **Streaming API** — System Permissions (available in Enterprise, Unlimited, and Developer editions)

To verify:
1. Go to **Setup** → **Profiles** (or **Permission Sets**).
2. Open the profile assigned to the integration user.
3. Under **System Permissions**, confirm **API Enabled** and **Streaming API** are checked.

---

### Step 3: Configure Connection Properties

Update `src/main/resources/config.yaml` (base config) or `src/main/resources/dev.yaml` (dev environment) with your Salesforce and email credentials:

```yaml
# Salesforce Connection
salesforce:
  username: "your-sf-user@example.com"        # Salesforce login username
  password: "your-sf-password"                 # Salesforce login password
  token: "your-sf-security-token"              # Security token from Salesforce → My Settings → Security Token
  url: "https://login.salesforce.com/services/Soap/u/56.0"  # Use https://test.salesforce.com for sandbox

# Email (SMTP) Connection
email:
  host: "smtp.gmail.com"                       # SMTP server hostname
  port: "587"                                  # SMTP port (587 = STARTTLS, 465 = SSL)
  username: "sender@gmail.com"                 # SMTP authentication username
  password: "your-app-password"                # SMTP password or app password
  from: "noreply@example.com"                  # Sender email address
  to: "admin@example.com"                      # Recipient email address
```

> ⚠️ **Never commit real credentials.** Use secure properties or environment-specific variable injection in production.

---

### Step 4: Salesforce Security Token

If you do not know your Salesforce security token:

1. Log in to Salesforce as the integration user.
2. Click the user avatar (top-right) → **My Settings**.
3. In the left sidebar, go to **Personal** → **Reset My Security Token**.
4. Click **Reset Security Token**. The token will be emailed to the user's email address.
5. Copy the token and paste it into `config.yaml` under `salesforce.token`.

---

### Step 5: Sandbox vs Production

| Environment | Salesforce Login URL |
|---|---|
| Production | `https://login.salesforce.com/services/Soap/u/56.0` |
| Sandbox | `https://test.salesforce.com/services/Soap/u/56.0` |

Set `salesforce.url` accordingly in your environment YAML file.

---

### Step 6: CDC Replay ID Options

The `replayId` attribute in `lead-sf-listener.xml` controls which events are received on startup:

| Value | Behavior |
|---|---|
| `-2` (default) | Receive only **new events** published after the application starts |
| `-1` | Receive **all retained events** (up to 72 hours) plus new events |
| `<specific ID>` | Replay from a specific event ID forward |

The current configuration uses `-1` (receive all retained + new events). Change this in `lead-sf-listener.xml` if needed:

```xml
<salesforce:replay-channel-listener
    config-ref="salesforceConfig"
    channelName="/data/LeadChangeEvent"
    replayId="-1"   <!-- Change to -2 for new events only -->
    ...>
```

---

## Email Notification

When a Lead record changes in Salesforce, the application sends an email with the following content:

```
Subject: Salesforce Lead Change Notification

Lead record has been updated

Lead Id     : 00Q1ABC2345DEF6GHI
Change Type : UPDATE
Replay Id   : 1234
Timestamp   : 2024-01-15T10:30:00.000Z
```

**Change Type values:** `CREATE`, `UPDATE`, `DELETE`, `UNDELETE`

---

## Building the Application

```bash
mvn clean package
```

The compiled Mule application JAR will be at:

```
target/mc-lead-exp-1.0.0-SNAPSHOT-mule-application.jar
```

---

## Running Locally

Deploy the JAR to a local Mule runtime or use Anypoint Code Builder to run in debug mode.

Ensure the following properties are set before starting:

- `salesforce.username`
- `salesforce.password`
- `salesforce.token`
- `salesforce.url`
- `email.host`, `email.port`, `email.username`, `email.password`, `email.from`, `email.to`

---

## Connectors Used

| Connector | Version | Purpose |
|---|---|---|
| `mule-salesforce-connector` | 10.19.4 | Salesforce CDC listener (Lead Change Events) |
| `mule-email-connector` | 1.8.0 | SMTP email notification |
| `mule-http-connector` | 1.10.5 | Experience API HTTP listener |
| `mule-apikit-module` | 1.11.7 | API routing from RAML spec |