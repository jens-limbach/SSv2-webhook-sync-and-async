# CRM Webhook Service

A minimal Express-based webhook microservice for SAP Sales and Service Cloud V2 that demonstrates both synchronous and asynchronous integration patterns. This service receives webhooks from SAP Sales and Service Cloud V2, calculates custom scores based on ABC classification, and updates accounts accordingly.

This is a very simplified example. You could achieve the same result also with the in-app no-code tools of the in-build customization (a simple if this than that) but I want with this example to show the different nature of the sync and async options you have when writing custom logic. Also I want to convince you that this is a very powerful option to do custom code with external microservices. Your business logic inside those "cloud functions" can then get as complicated as you want ;)

Personaly I would recommend to build this central extension microservice where you can place all your endpoints for custom logic and maintain all your custom code in a central place.

## ✨ Features

- ✅ **Synchronous Webhook** - Immediate response with calculated score, returns full payload
- ✅ **Asynchronous Webhook** - Background processing with 10-second delay and CRM API update
- ✅ **CloudEvents Support** - Handles CloudEvents format with data wrapper
- ✅ **ABC Classification Logic** - Auto-scoring (A=90, B=70, C=50)
- ✅ **CORS Enabled** - Cross-origin requests supported
- ✅ **Clean Logging** - Operational logs without verbose payload dumps
- ✅ **Secure API Integration** - Server-side Basic Auth for CRM API calls
- ✅ **Optimistic Locking** - ETag-based concurrency control for updates
- ✅ **Minimal Dependencies** - Express + dotenv only
- ✅ **Cloud Foundry Ready** - Production deployment configuration included
- ✅ **Lightweight** - ~250 lines of code, 128MB memory footprint

## 🚀 Quick Start

### Prerequisites
- Node.js 22.x or higher
- SAP Sales and Service Cloud V2 access with API credentials

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd crm-webhook-service
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure environment**
   ```bash
   copy env-template.txt .env
   ```
   
   Edit `.env` with your SAP CRM credentials:
   ```env
   CRM_BASE_URL=https://your-tenant.crm.cloud.sap
   CRM_USERNAME=your-username
   CRM_PASSWORD=your-password
   PORT=3000
   ```

4. **Start the service**
   ```bash
   npm run dev
   ```
   
   Service runs on `http://localhost:3000`

5. **Test the webhooks**

**Synchronous webhook:**
```bash
curl -X POST http://localhost:3000/webhooks/calculate-score-sync \
  -H "Content-Type: application/json" \
  -d @webhook-payload.json
```

**Asynchronous webhook:**
```bash
curl -X POST http://localhost:3000/webhooks/calculate-score-async \
  -H "Content-Type: application/json" \
  -d @webhook-payload.json
```

## 🔗 Webhook Endpoints

### Synchronous Webhook

**Endpoint:** `POST /webhooks/calculate-score-sync`

**Purpose:** Calculate and return CustomScore immediately based on ABC classification. Use this when CRM needs the result before completing the save operation.

**Request Payload (CloudEvents format):**
```json
{
  "id": "ec7e11d8-ed35-449b-affe-bccbc58b1412",
  "specversion": "0.2",
  "type": "sap.crm.custom.event.updateAccount",
  "source": "63d2b99cd28c19121118a781",
  "subject": "0197a1ea-0afb-711a-9b74-03460a24dff7",
  "time": "2026-01-13T13:59:32.934399888Z",
  "datacontenttype": "application/json",
  "data": {
    "beforeImage": {
      "id": "0197a1ea-0afb-711a-9b74-03460a24dff7",
      "displayId": "2136299",
      "customerABCClassification": "B",
      "isProspect": false,
      "extensions": {
        "CustomScore": 47
      }
    },
    "currentImage": {
      "id": "0197a1ea-0afb-711a-9b74-03460a24dff7",
      "displayId": "2136299",
      "customerABCClassification": "A",
      "isProspect": false,
      "adminData": {
        "updatedOn": "2026-01-13T13:59:32.671Z"
      },
      "extensions": {
        "CustomScore": 63
      }
    },
    "dataContext": {
      "requestedByUser": "b8adfb7e-50f3-11f0-a28d-fd9b4de46c56",
      "requestProcessedOn": "2026-01-13T13:59:32.671Z"
    }
  }
}
```

**Response:** (200 OK)
```json
{
  "data": {
    "id": "0197a1ea-0afb-711a-9b74-03460a24dff7",
    "displayId": "2136299",
    "customerABCClassification": "A",
    "isProspect": false,
    "adminData": {
      "updatedOn": "2026-01-13T13:59:32.671Z"
    },
    "extensions": {
      "CustomScore": 90
    }
  }
}
```

**Note:** The response returns the complete `currentImage` with only the `CustomScore` field updated. All other fields from the request are preserved.

**Error Response:** (400 Bad Request)
```json
{
  "error": "Missing currentImage object"
}
```

### Asynchronous Webhook

**Endpoint:** `POST /webhooks/calculate-score-async`

**Purpose:** Accept webhook, respond immediately, then process in background with a simulated 10-second delay and update CRM via API. Applies a 15-point penalty for prospects.

**Request Payload:** Same CloudEvents format as synchronous webhook

**Immediate Response:** (202 Accepted)
```json
{
  "accepted": true,
  "message": "Processing in background"
}
```

**Background Processing:**
1. Waits 10 seconds (simulated processing delay)
2. Calculates CustomScore based on ABC classification
3. Fetches current account from CRM to get fresh ETag
4. Updates account via PATCH with calculated score
5. Logs success or failure (no callback to CRM on error)

## 📊 ABC Classification Scoring Logic

The service automatically calculates `CustomScore` based on `customerABCClassification`:

| ABC Class | Base Score | Prospect Penalty | Final Score (Prospect) | Used In |
|-----------|-Score | Description |
|-----------|-------|-------------|
| **A** | 90 | Top-tier customers |
| **B** | 70 | Mid-tier customers |
| **C** | 50 | Standard customers |
| *Invalid/Missing* | 50 | Defaults to C classification |

**Notes:**
- Same scoring logic applies to both sync and async webhooksscore 50)

## 🎯 Webhook Payload Structure

### CloudEvents Format

SAP CRM sends webhooks in CloudEvents format with data wrapped in a `data` object:

```json
{
  "id": "unique-event-id",
  "specversion": "0.2",
  "type": "sap.crm.custom.event.updateAccount",
  "source": "tenant-id",
  "subject": "account-id",
  "time": "2026-01-13T13:59:32.934399888Z",
  "datacontenttype": "application/json",
  "data": {
    "beforeImage": {...},
    "currentImage": {...},
    "dataContext": {...}
  }
}
```

The service supports both CloudEvents format (with `data` wrapper) and direct format (without wrapper) for flexibility.

### Key Fields Used

- `data.currentImage.id` - Account UUID (required)
- `data.currentImage.displayId` - Human-readable account ID (for logging)
- `data.currentImage.customerABCClassification` - ABC class
- `data.currentImage.customerABCClassification` - ABC class code (A/B/C)
- `data.currentImage.extensions.CustomScore` - Current score value
- `data.currentImage.adminData.updatedOn` - Used for ETag generation

### Validation Rules

✅ Required fields:
- `data.currentImage` object (or `currentImage` if no wrapper)
- `data.currentImage.id` (account UUID)

⚠️ Optional fields:
- `data.beforeImage` (not required, but included in CloudEvents)
- `customerABCClassification` (defaults to "C"
## 🔧 API Integration Details

### CRM API Endpoint Used

**Async webhook calls:**
- `GET /sap/c4c/api/v1/account-service/accounts/{id}` - Fetch account for ETag
- `PATCH /sap/c4c/api/v1/account-service/accounts/{id}` - Update CustomScore

### Authentication

- **Method:** HTTP Basic Authentication
- **Header:** `Authorization: Basic <base64-encoded-credentials>`
- **Credentials:** Loaded from environment variables

### Optimistic Locking

The async webhook uses `If-Match` header with ETag to prevent conflicts:
1. Fetches account to get current ETag from response headers
2. Includes ETag in PATCH request via `If-Match` header
3. CRM rejects update if account was modified since fetch (409 Conflict)

## 🔍 Logging

The service includes comprehensive logging for testing and debugging:

### Webhook Receipt Logs
```lean operational logging for monitoring:

### Operational Logs
```
✅ Calculated score: 70 (ABC: B)
🔄 Starting async processing for account 0197a1ea-0afb-711a-9b74-03460a24dff7...
⏳ Simulating 10-second processing delay...
📡 Fetching current account data for 0197a1ea-0afb-711a-9b74-03460a24dff7...
✅ Received ETag: "2026-01-13T14:04:22.200Z"
📡 Updating account 0197a1ea-0afb-711a-9b74-03460a24dff7 with score 70...
✅ Successfully updated account 0197a1ea-0afb-711a-9b74-03460a24dff7 with CustomScore: 70
```

### Error Logs
```
❌ Validation failed: Missing currentImage object
❌ CRM API call failed: CRM API Error (401): Unauthorized
❌ Async webhook processing failed:
   Account ID: 0197a1ea-0afb-711a-9b74-03460a24dff7
   ABC Classification: B
   Error: CRM API Error (401):
```

**Note:** Verbose payload logging has been removed for cleaner production logs
### Cloud Foundry (SAP BTP)

```bash
# Deploy the service
cf push

# Set environment variables
cf set-env crm-webhook-service CRM_BASE_URL "https://your-tenant.crm.cloud.sap"
cf set-env crm-webhook-service CRM_USERNAME "your-username"
cf set-env crm-webhook-service CRM_PASSWORD "your-password"
cf restage crm-webhook-service

# Check health
cf app crm-webhook-service
```

### Health Check

The service includes a health check endpoint for monitoring:

```bash
curl http://localhost:3000/health
```

Response:
```json
{
  "status": "ok",
  "service": "CRM Webhook Service",
  "timestamp": "2026-01-13T10:37:27.666Z"
}
```

## 📁 Project Structure

```
crm-webhook-service/
├── server.js              # Main webhook service (~300 lines)
├── package.json           # Dependencies (express, dotenv)
├── .env                   # Your credentials (git-ignored)
├── env-template.txt       # Template for .env
├── manifest.yml           # Cloud Foundry deployment config
├── .gitignore             # Excludes credentials and node_modules
└── README.md              # This documentation
```

## 🐛 Troubleshooting

**"Missing required environment variables"**
- Create `.env` file from `env-template.txt`
- Ensure all variables are set (CRM_BASE_URL, CRM_USERNAME, CRM_PASSWORD)

**Webhook not receiving requests**
- Verify SAP CRM webhook configuration points to correct URL
- Check firewall/network allows SAP to reach your service
- Test locally with curl first

**"CRM API Error (401)" - Unauthorized**
- Verify credentials in `.env` are correct
- Check username/password have API access permissions
- **Important:** User must have **write/update permissions** for accounts in SAP CRM
- The user can read accounts (GET) but may lack permission to update (PATCH)
- Contact your SAP CRM administrator to grant proper authorization roles

**"CRM API Error (404)" - Not Found**
- Check webhook URL path is correct: `/webhooks/calculate-score-async` (not `asynch`)
- Verify the service is deployed and running
- Test the health endpoint first: `/health`

**"Missing currentImage object"**
- Ensure webhook payload follows CloudEvents format with `data` wrapper
- Check that `data.currentImage` exists in the payload
- The service logs the full payload for debugging

**"No ETag received from CRM"**
- Ensure account exists in CRM
- Check CRM API is returning proper headers
- Verify account ID in webhook payload is correct

**Async webhook fails silently**
- Check server logs for detailed error messages: `cf logs crm-webhook-service --recent`
- Errors are logged with account ID and ABC classification
- Common causes: invalid ETag, account locked, network issues, 401 permission error

**ABC classification not recognized**
- Service defaults to "C" (score 50) for invalid values
- Check for typos in classification field (should be A, B, or C)
- Look for warning in logs: `⚠️ Unknown ABC classification`

**CORS errors in browser**
- The service includes CORS headers for cross-origin requests
- If still encountering issues, check browser console for specific error
- Verify `Access-Control-Allow-Origin` is set to `*` in response headers

## 📋 Known Issues & Status

### Current Status
- ✅ Synchronous webhook working correctly
- ✅ CloudEvents format support working
- ✅ CORS enabled and working
- ✅ **All features working correctly**
- ✅ Synchronous webhook working with full payload return
- ✅ Asynchronous webhook with 10-second delay working
- ✅ CloudEvents format support working
- ✅ CORS enabled and working
- ✅ ETag fetching and optimistic locking working
- ✅ CRM updates via PATCH working (auth header bug fixed)
- ✅ Clean operational logging without verbose payloads

### Recent Fixes
1. **Fixed 401 Authorization Error** - Headers now properly merged when adding `If-Match` for PATCH requests
2. **Simplified Logic** - Removed prospect penalty, pure ABC classification scoring
3. **Clean Logging** - Removed verbose payload dumps, keeping only operational logs
### When to Use Synchronous Webhook

✅ **Use synchronous when:**
- Result needed before save completes
- Calculation is fast (<1 second)
- Want to block invalid data from being saved
- Need to return validation errors to user

❌ **Don't use synchronous for:**
- Long-running operations (>2 seconds)
- External API calls that might timeout
- Operations that can fail independently

### When to Use Asynchronous Webhook

✅ **Use asynchronous when:**
- Processing takes time (>1 second)
- Result not needed immediately
- Want to respond quickly and process later
- Can tolerate eventual consistency

❌ **Don't use asynchronous for:**
- Real-time validation
- Results needed in current transaction
- Critical business logic that must complete

## 📝 Common Commands

```bash
# Install dependencies
npm install

# Start development server (local)
npm run dev

# Deploy to Cloud Foundry
cf push

# View live logs (streaming)
cf logs crm-webhook-service

# View recent logs (last events)
cf logs crm-webhook-service --recent

# Check app status and health
cf app crm-webhook-service

# Test health endpoint
curl http://localhost:3000/health

# Test sync webhook locally
curl -X POST http://localhost:3000/webhooks/calculate-score-sync \
  -H "Content-Type: application/json" \
  -d '{"data":{"currentImage":{"id":"test-123","customerABCClassification":"A"}}}'

# Set environment variables in Cloud Foundry
cf set-env crm-webhook-service CRM_USERNAME "new-username"
cf restage crm-webhook-service
```

## 🔧 Configuration

### Environment Variables

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `CRM_BASE_URL` | SAP CRM tenant URL | `https://my1000210.de1.demo.crm.cloud.sap` | Yes |
| `CRM_USERNAME` | API user with read/write access | `api-user` or `dev` | Yes |
| `CRM_PASSWORD` | API user password | `***` | Yes |
| `PORT` | Server port (auto-set by Cloud Foundry) | `3000` or `8080` | No |
| `NODE_ENV` | Environment (set in manifest.yml) | `production` | No |

### Cloud Foundry Configuration (manifest.yml)

```yaml
applications:
- name: crm-webhook-service
  memory: 128M
  instances: 1
  buildpack: nodejs_buildpack
  command: node server.js
  env:
    NODE_ENV: production
  health-check-type: http
  health-check-http-endpoint: /health
```

## 🏗️ Architecture & Design

### Technology Stack
- **Runtime:** Node.js 22.x (auto-detected by buildpack)
- **Framework:** Express 4.18.2
- **Configuration:** dotenv 16.3.1
- **HTTP Client:** Native Fetch API (Node.js built-in)
- **Deployment:** Cloud Foundry (SAP BTP)

### Design Patterns
- **Microservice Architecture:** Single-purpose service with clear API
- **Webhook Pattern:** Event-driven integration with SAP CRM
- **Async Processing:** Fire-and-forget background jobs with `setImmediate`
- **Optimistic Locking:** ETag-based concurrency control
- **Basic Authentication:** Simple, secure API access
- **CORS Enabled:** Cross-origin resource sharing for browser access

### Memory & Performance
- **Memory Usage:** 23-25MB runtime, 128MB allocated
- **Response Time:** <50ms for webhook acceptance
- **Processing Time:** 10 seconds simulated delay + API calls (~10.5s total)
- **Concurrency:** Single instance, background tasks per request

## 🔐 Security Consi~25MB runtime, 128MB allocated
- **Response Time:** <50ms for webhook acceptance
- **Processing Time:** 10 seconds simulated delay + API calls (~10.5s total)
- **Concurrency:** Single instance, background tasks per request
- **Code Size:** ~250 lines of clean, maintainable code
- Never logged or exposed in responses
- Credentials required for all CRM API calls

### Best Practices
- ✅ Environment variables for sensitive data
- ✅ No credentials in code or version control
- ✅ HTTPS enforced by Cloud Foundry router
- ✅ Input validation on all webhook payloads
- ⚠️ Consider API key authentication for webhook endpoints in production
- ⚠️ Consider IP whitelisting for webhook sources

### Production Recommendations
1. Add webhook signature verification
2. Implement rate limiting
3. Add authentication for webhook endpoints
4. Use dedicated service account with minimal permissions
5. Enable audit logging for compliance
6. Implement retry logic with exponential backoff

## 📚 Related Projects

- **CRM Scoring Widget** - Frontend widget for manual score editing with micro-frontend pattern

## 📄 License

MIT
