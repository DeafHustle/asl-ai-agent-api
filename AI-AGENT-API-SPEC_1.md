# 🤖 AmericanSignLanguage.eth AI Agent API Specification v1.0

## Overview

RESTful API enabling AI agents to provision on-demand ASL interpreters for real-time video communication. Blockchain-native payment system with automatic token distribution.

**Base URL:** `https://api.americansignlanguage.xyz/v1`

**Authentication:** Bearer token (API key)

**Payment:** $ASL ERC-20 tokens on Ethereum/Base

---

## 🔑 Authentication

### Get API Key

```http
POST /auth/register
Content-Type: application/json

{
  "agent_name": "MoltbookAgent_CustomerService",
  "company": "Acme AI Corp",
  "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "email": "dev@acme.ai",
  "use_case": "customer_service"
}

Response 201:
{
  "api_key": "asl_live_4f8k2j9d0s7h3m1q6p",
  "api_secret": "sk_live_a9b8c7d6e5f4g3h2i1j0",
  "rate_limit": "1000/hour",
  "webhook_url": null,
  "created_at": "2025-01-31T10:30:00Z"
}
```

### All Requests Require Header

```http
Authorization: Bearer asl_live_4f8k2j9d0s7h3m1q6p
```

---

## 📞 Core Endpoints

### 1. Request Interpreter (Instant Matching)

```http
POST /interpreter/request
Content-Type: application/json
Authorization: Bearer {api_key}

{
  "user_wallet": "0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7",
  "urgency": "high",
  "estimated_duration": 30,
  "specialization": "medical",
  "language_pair": "ASL-English",
  "metadata": {
    "agent_id": "moltbook_agent_789",
    "user_id": "user_12345",
    "session_type": "appointment_booking",
    "context": "Medical appointment scheduling for patient"
  }
}

Response 200 (Interpreter Available):
{
  "request_id": "req_9f8e7d6c5b4a",
  "status": "matched",
  "interpreter": {
    "id": "interp_a1b2c3d4",
    "name": "Professional ASL Interpreter",
    "certifications": ["RID", "NAD-IV"],
    "specializations": ["medical", "legal"],
    "rating": 4.9,
    "total_sessions": 1247
  },
  "video_room": {
    "url": "https://americansignlanguage.xyz/room/xyz789",
    "room_id": "xyz789",
    "expires_at": "2025-01-31T11:30:00Z",
    "join_token_agent": "eyJhbGc...",
    "join_token_user": "eyJhbGc...",
    "join_token_interpreter": "eyJhbGc..."
  },
  "pricing": {
    "rate_per_minute": 20,
    "currency": "ASL",
    "estimated_cost": 600,
    "escrow_required": 600,
    "escrow_tx_hash": null
  },
  "websocket_url": "wss://api.americansignlanguage.xyz/v1/sessions/req_9f8e7d6c5b4a",
  "estimated_wait_time": 0,
  "matched_at": "2025-01-31T10:30:05Z"
}

Response 202 (No Immediate Match):
{
  "request_id": "req_9f8e7d6c5b4a",
  "status": "searching",
  "estimated_wait_time": 45,
  "queue_position": 2,
  "websocket_url": "wss://api.americansignlanguage.xyz/v1/sessions/req_9f8e7d6c5b4a",
  "message": "Searching for available interpreter. You will be notified via webhook when matched."
}

Response 402 (Insufficient Funds):
{
  "error": "insufficient_balance",
  "required_amount": 600,
  "current_balance": 120,
  "deficit": 480,
  "wallet_address": "0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7",
  "top_up_url": "https://americansignlanguage.xyz/wallet/top-up"
}
```

**Field Descriptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_wallet` | string | Yes | Ethereum wallet address of end user (Deaf person) |
| `urgency` | enum | No | `low`, `medium`, `high`, `emergency` (default: `medium`) |
| `estimated_duration` | integer | Yes | Expected call duration in minutes (1-240) |
| `specialization` | enum | No | `general`, `medical`, `legal`, `educational`, `business` |
| `language_pair` | string | No | Default: `ASL-English`. Future: `ASL-Spanish`, etc. |
| `metadata` | object | No | Custom data for tracking/analytics |

---

### 2. Get Session Status

```http
GET /sessions/{request_id}
Authorization: Bearer {api_key}

Response 200:
{
  "request_id": "req_9f8e7d6c5b4a",
  "status": "active",
  "interpreter_id": "interp_a1b2c3d4",
  "started_at": "2025-01-31T10:31:00Z",
  "duration_minutes": 12,
  "estimated_end": "2025-01-31T11:01:00Z",
  "video_room_url": "https://americansignlanguage.xyz/room/xyz789",
  "participants": {
    "user_connected": true,
    "interpreter_connected": true,
    "agent_connected": false
  },
  "current_cost": 240,
  "token_escrow": {
    "amount": 600,
    "tx_hash": "0x4f3d2e1c9b8a7f6e5d4c3b2a1",
    "released": false
  }
}
```

**Session Status Values:**
- `searching` - Looking for interpreter
- `matched` - Interpreter assigned, waiting for connection
- `active` - Call in progress
- `ending` - One party ended call
- `completed` - Call ended, tokens distributed
- `cancelled` - Request cancelled
- `failed` - System error

---

### 3. End Session

```http
POST /sessions/{request_id}/end
Authorization: Bearer {api_key}

{
  "ended_by": "agent",
  "reason": "task_completed",
  "rating": 5,
  "feedback": "Excellent service, appointment booked successfully"
}

Response 200:
{
  "request_id": "req_9f8e7d6c5b4a",
  "status": "completed",
  "duration_minutes": 28,
  "final_cost": 560,
  "token_distribution": {
    "interpreter": 252,
    "platform": 252,
    "user_cashback": 56,
    "total": 560
  },
  "transactions": [
    {
      "to": "0xInterpreterWallet",
      "amount": 252,
      "tx_hash": "0xabc123...",
      "status": "confirmed"
    },
    {
      "to": "0xPlatformWallet",
      "amount": 252,
      "tx_hash": "0xdef456...",
      "status": "confirmed"
    },
    {
      "to": "0xUserWallet",
      "amount": 56,
      "tx_hash": "0xghi789...",
      "status": "confirmed"
    }
  ],
  "refund": {
    "amount": 40,
    "tx_hash": "0xjkl012...",
    "reason": "unused_escrow"
  },
  "ended_at": "2025-01-31T10:59:00Z"
}
```

---

### 4. Cancel Request

```http
POST /sessions/{request_id}/cancel
Authorization: Bearer {api_key}

{
  "reason": "user_unavailable"
}

Response 200:
{
  "request_id": "req_9f8e7d6c5b4a",
  "status": "cancelled",
  "refund": {
    "amount": 600,
    "tx_hash": "0xmno345...",
    "status": "pending"
  },
  "cancellation_fee": 0,
  "cancelled_at": "2025-01-31T10:32:00Z"
}
```

**Cancellation Fees:**
- Before match: $0
- After match, before start: 10% (2 minutes fee)
- After start: Full session cost

---

### 5. List Available Interpreters

```http
GET /interpreters/available
Authorization: Bearer {api_key}

Query Parameters:
- specialization (optional): medical, legal, educational
- min_rating (optional): 4.5
- max_rate (optional): 25
- language_pair (optional): ASL-English

Response 200:
{
  "total": 12,
  "available_now": 12,
  "interpreters": [
    {
      "id": "interp_a1b2c3d4",
      "specializations": ["medical", "legal"],
      "certifications": ["RID", "NAD-V"],
      "rating": 4.9,
      "total_sessions": 1247,
      "response_time_avg": 8,
      "rate_per_minute": 20,
      "available": true,
      "next_available": null
    },
    {
      "id": "interp_x9y8z7",
      "specializations": ["general", "business"],
      "certifications": ["RID"],
      "rating": 4.7,
      "total_sessions": 892,
      "response_time_avg": 12,
      "rate_per_minute": 18,
      "available": true,
      "next_available": null
    }
  ]
}
```

---

### 6. Get Pricing

```http
GET /pricing
Authorization: Bearer {api_key}

Response 200:
{
  "base_rate_per_minute": 20,
  "currency": "ASL",
  "specialization_multipliers": {
    "general": 1.0,
    "medical": 1.0,
    "legal": 1.2,
    "educational": 1.0,
    "emergency": 1.5
  },
  "urgency_multipliers": {
    "low": 1.0,
    "medium": 1.0,
    "high": 1.2,
    "emergency": 2.0
  },
  "revenue_split": {
    "interpreter_percent": 45,
    "platform_percent": 45,
    "user_cashback_percent": 10
  },
  "minimum_session_minutes": 5,
  "api_fee_per_request": 0,
  "monthly_subscription_options": [
    {
      "tier": "starter",
      "price": 99,
      "included_calls": 100,
      "overage_rate": 1.0
    },
    {
      "tier": "growth",
      "price": 499,
      "included_calls": 1000,
      "overage_rate": 0.5
    },
    {
      "tier": "enterprise",
      "price": "custom",
      "included_calls": "unlimited",
      "overage_rate": 0
    }
  ]
}
```

---

## 🔔 Webhooks

### Configure Webhook

```http
POST /webhooks
Authorization: Bearer {api_key}

{
  "url": "https://your-ai-agent.com/webhooks/asl",
  "events": [
    "interpreter.matched",
    "session.started",
    "session.ended",
    "payment.completed"
  ],
  "secret": "whsec_your_webhook_secret_key"
}

Response 201:
{
  "webhook_id": "wh_1a2b3c4d",
  "url": "https://your-ai-agent.com/webhooks/asl",
  "events": ["interpreter.matched", "session.started", "session.ended", "payment.completed"],
  "active": true,
  "created_at": "2025-01-31T10:30:00Z"
}
```

### Webhook Events

#### interpreter.matched

```json
{
  "event": "interpreter.matched",
  "timestamp": "2025-01-31T10:30:05Z",
  "data": {
    "request_id": "req_9f8e7d6c5b4a",
    "interpreter_id": "interp_a1b2c3d4",
    "video_room_url": "https://americansignlanguage.xyz/room/xyz789",
    "join_token_agent": "eyJhbGc...",
    "estimated_cost": 600
  }
}
```

#### session.started

```json
{
  "event": "session.started",
  "timestamp": "2025-01-31T10:31:00Z",
  "data": {
    "request_id": "req_9f8e7d6c5b4a",
    "interpreter_id": "interp_a1b2c3d4",
    "participants": {
      "user_connected": true,
      "interpreter_connected": true,
      "agent_connected": false
    }
  }
}
```

#### session.ended

```json
{
  "event": "session.ended",
  "timestamp": "2025-01-31T10:59:00Z",
  "data": {
    "request_id": "req_9f8e7d6c5b4a",
    "duration_minutes": 28,
    "final_cost": 560,
    "ended_by": "user",
    "reason": "completed"
  }
}
```

#### payment.completed

```json
{
  "event": "payment.completed",
  "timestamp": "2025-01-31T10:59:15Z",
  "data": {
    "request_id": "req_9f8e7d6c5b4a",
    "token_distribution": {
      "interpreter": 252,
      "platform": 252,
      "user_cashback": 56
    },
    "transactions": [
      {
        "to": "0xInterpreterWallet",
        "amount": 252,
        "tx_hash": "0xabc123..."
      }
    ]
  }
}
```

---

## 🔌 WebSocket Real-Time Updates

### Connect to Session WebSocket

```javascript
const ws = new WebSocket('wss://api.americansignlanguage.xyz/v1/sessions/req_9f8e7d6c5b4a');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'authenticate',
    api_key: 'asl_live_4f8k2j9d0s7h3m1q6p'
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Status update:', data);
};
```

### WebSocket Messages

**Status Updates:**
```json
{
  "type": "status_update",
  "request_id": "req_9f8e7d6c5b4a",
  "status": "active",
  "duration_minutes": 12,
  "current_cost": 240
}
```

**Interpreter Joined:**
```json
{
  "type": "participant_joined",
  "participant": "interpreter",
  "timestamp": "2025-01-31T10:31:00Z"
}
```

**User Joined:**
```json
{
  "type": "participant_joined",
  "participant": "user",
  "timestamp": "2025-01-31T10:31:05Z"
}
```

**Call Ended:**
```json
{
  "type": "session_ended",
  "ended_by": "user",
  "final_cost": 560,
  "duration_minutes": 28
}
```

---

## 💳 Payment & Token Management

### Check Balance

```http
GET /wallet/balance
Authorization: Bearer {api_key}

Response 200:
{
  "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "balance": 5000,
  "currency": "ASL",
  "balance_usd": 10000,
  "reserved": 1200,
  "available": 3800,
  "last_updated": "2025-01-31T10:30:00Z"
}
```

### Top Up Wallet

```http
POST /wallet/deposit
Authorization: Bearer {api_key}

{
  "amount": 10000,
  "tx_hash": "0x9f8e7d6c5b4a3c2b1a0"
}

Response 200:
{
  "deposit_id": "dep_1a2b3c",
  "amount": 10000,
  "tx_hash": "0x9f8e7d6c5b4a3c2b1a0",
  "confirmations": 3,
  "status": "confirmed",
  "new_balance": 15000,
  "credited_at": "2025-01-31T10:35:00Z"
}
```

### Get Transaction History

```http
GET /wallet/transactions
Authorization: Bearer {api_key}

Query Parameters:
- limit: 50 (default: 20, max: 100)
- offset: 0
- type: all|deposit|payment|refund

Response 200:
{
  "total": 234,
  "transactions": [
    {
      "id": "tx_9f8e7d6c",
      "type": "payment",
      "amount": -560,
      "description": "Session req_9f8e7d6c5b4a - 28 minutes",
      "request_id": "req_9f8e7d6c5b4a",
      "tx_hash": "0xabc123...",
      "timestamp": "2025-01-31T10:59:00Z"
    },
    {
      "id": "tx_8e7d6c5b",
      "type": "deposit",
      "amount": 10000,
      "description": "Wallet top-up",
      "tx_hash": "0x9f8e7d6c5b4a3c2b1a0",
      "timestamp": "2025-01-31T10:35:00Z"
    }
  ]
}
```

---

## 📊 Analytics & Reporting

### Get Usage Stats

```http
GET /analytics/usage
Authorization: Bearer {api_key}

Query Parameters:
- start_date: 2025-01-01
- end_date: 2025-01-31
- grouping: day|week|month

Response 200:
{
  "period": {
    "start": "2025-01-01T00:00:00Z",
    "end": "2025-01-31T23:59:59Z"
  },
  "summary": {
    "total_sessions": 156,
    "total_minutes": 4234,
    "total_cost": 84680,
    "average_session_duration": 27.14,
    "success_rate": 98.7,
    "average_wait_time": 8.2
  },
  "by_specialization": {
    "medical": { "sessions": 89, "minutes": 2401 },
    "general": { "sessions": 45, "minutes": 1203 },
    "legal": { "sessions": 22, "minutes": 630 }
  },
  "top_interpreters": [
    {
      "interpreter_id": "interp_a1b2c3d4",
      "sessions": 42,
      "rating": 4.9
    }
  ]
}
```

---

## 🔐 Security & Compliance

### Rate Limits

| Tier | Requests/Hour | Requests/Day | Cost |
|------|---------------|--------------|------|
| Free | 100 | 1,000 | $0 |
| Starter | 1,000 | 10,000 | $99/mo |
| Growth | 10,000 | 100,000 | $499/mo |
| Enterprise | Unlimited | Unlimited | Custom |

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1643673600
```

### IP Whitelisting

```http
POST /security/whitelist
Authorization: Bearer {api_key}

{
  "ip_addresses": [
    "192.168.1.1",
    "10.0.0.0/24"
  ]
}

Response 200:
{
  "whitelist_id": "wl_1a2b3c",
  "ip_addresses": ["192.168.1.1", "10.0.0.0/24"],
  "active": true
}
```

### Webhook Signature Verification

```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}

// In your webhook handler:
app.post('/webhooks/asl', (req, res) => {
  const signature = req.headers['x-asl-signature'];
  const isValid = verifyWebhook(
    JSON.stringify(req.body),
    signature,
    'whsec_your_webhook_secret_key'
  );
  
  if (!isValid) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook
  res.status(200).send('OK');
});
```

---

## 📱 SDK Examples

### JavaScript/TypeScript SDK

```javascript
npm install @americansignlanguage/ai-agent-sdk
```

```typescript
import { ASLAgent } from '@americansignlanguage/ai-agent-sdk';

const asl = new ASLAgent({
  apiKey: 'asl_live_4f8k2j9d0s7h3m1q6p',
  webhookUrl: 'https://your-agent.com/webhooks/asl'
});

// Request interpreter
const session = await asl.requestInterpreter({
  userWallet: '0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7',
  urgency: 'high',
  estimatedDuration: 30,
  specialization: 'medical',
  metadata: {
    agentId: 'moltbook_agent_789',
    context: 'Medical appointment booking'
  }
});

console.log('Video room:', session.videoRoom.url);
console.log('Status:', session.status);

// Listen for real-time updates
session.on('status_update', (data) => {
  console.log('Current cost:', data.current_cost);
});

session.on('ended', async (data) => {
  console.log('Session completed:', data.duration_minutes, 'minutes');
  console.log('Final cost:', data.final_cost, 'ASL tokens');
});

// End session
await session.end({
  rating: 5,
  feedback: 'Excellent service'
});
```

### Python SDK

```python
pip install americansignlanguage-ai-agent
```

```python
from asl_agent import ASLAgent

asl = ASLAgent(
    api_key='asl_live_4f8k2j9d0s7h3m1q6p',
    webhook_url='https://your-agent.com/webhooks/asl'
)

# Request interpreter
session = asl.request_interpreter(
    user_wallet='0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7',
    urgency='high',
    estimated_duration=30,
    specialization='medical',
    metadata={
        'agent_id': 'moltbook_agent_789',
        'context': 'Medical appointment booking'
    }
)

print(f"Video room: {session.video_room.url}")
print(f"Status: {session.status}")

# Wait for completion
result = session.wait_for_completion(timeout=3600)
print(f"Completed in {result.duration_minutes} minutes")
print(f"Cost: {result.final_cost} ASL tokens")
```

---

## 🚨 Error Codes

| Code | Message | Description |
|------|---------|-------------|
| 400 | `invalid_request` | Malformed request |
| 401 | `unauthorized` | Invalid API key |
| 402 | `insufficient_balance` | Not enough tokens |
| 404 | `session_not_found` | Session doesn't exist |
| 409 | `session_already_active` | Can't modify active session |
| 429 | `rate_limit_exceeded` | Too many requests |
| 500 | `internal_error` | Server error |
| 503 | `no_interpreters_available` | All interpreters busy |

**Error Response Format:**
```json
{
  "error": "insufficient_balance",
  "message": "Your wallet has insufficient ASL tokens for this request",
  "required": 600,
  "available": 120,
  "deficit": 480,
  "timestamp": "2025-01-31T10:30:00Z",
  "request_id": "req_9f8e7d6c5b4a",
  "docs_url": "https://docs.americansignlanguage.eth/errors/insufficient_balance"
}
```

---

## 🎯 Integration Examples

### Moltbook Agent Integration

```javascript
// In your Moltbook agent code
import { MoltbookAgent } from '@moltbook/sdk';
import { ASLAgent } from '@americansignlanguage/ai-agent-sdk';

const agent = new MoltbookAgent();
const asl = new ASLAgent({ apiKey: process.env.ASL_API_KEY });

agent.on('call_initiated', async (call) => {
  // Detect if user needs ASL
  if (call.user.language === 'ASL' || call.user.preferences.asl) {
    
    // Request interpreter
    const session = await asl.requestInterpreter({
      userWallet: call.user.wallet,
      urgency: 'high',
      estimatedDuration: 15,
      specialization: call.category,
      metadata: {
        agentId: agent.id,
        callId: call.id
      }
    });
    
    // Add interpreter to call
    await call.addParticipant(session.videoRoom.url);
    
    // Proceed with agent task
    await agent.completeTask(call);
    
    // End interpreter session
    await session.end({ rating: 5 });
  }
});
```

### Customer Service Bot

```python
from your_ai_bot import CustomerServiceBot
from asl_agent import ASLAgent

bot = CustomerServiceBot()
asl = ASLAgent(api_key=os.getenv('ASL_API_KEY'))

@bot.on_message
async def handle_message(message):
    if message.language == 'ASL':
        # Start ASL interpretation session
        session = asl.request_interpreter(
            user_wallet=message.user.wallet,
            urgency='medium',
            estimated_duration=10,
            specialization='customer_service'
        )
        
        # Connect user to video room
        await bot.send_video_link(
            user=message.user,
            url=session.video_room.url
        )
        
        # Handle support query with interpreter present
        await bot.resolve_support_ticket(message, interpreter=session)
        
        # Close session
        await session.end()
```

---

## 📖 Additional Resources

**Documentation:** https://docs.americansignlanguage.eth/api

**API Status:** https://status.americansignlanguage.eth

**Community:** https://discord.gg/americansignlanguage

**Support:** api-support@americansignlanguage.eth

**GitHub:** https://github.com/americansignlanguage/ai-agent-sdk

---

## 🔄 API Versioning

Current version: **v1**

Version included in base URL: `https://api.americansignlanguage.xyz/v1`

Breaking changes will result in new version (v2, v3, etc.)

v1 will be supported for minimum 12 months after v2 launch

---

## ⚖️ Terms of Service

By using this API, you agree to:

1. Only use for legitimate communication purposes
2. Respect interpreter and user privacy
3. Comply with ADA and accessibility laws
4. Not abuse rate limits
5. Pay for services rendered
6. Maintain API key security

**Full Terms:** https://americansignlanguage.xyz/api-terms

---

**API Specification v1.0**
**Last Updated:** January 31, 2025
**Developed by DeafDev**
**© 2025 americansignlanguage.eth**
