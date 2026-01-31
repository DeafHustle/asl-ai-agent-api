# 🤖 ASL AI Agent API

> **The first API enabling AI agents to provision on-demand ASL interpreters**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

## 🎯 What This Is

AI agents can now provide real-time ASL interpretation for Deaf users through a simple REST API.

**Problem:** 48 million Deaf/hard-of-hearing people can't use AI agents for phone calls, appointments, or customer service.

**Solution:** API-first ASL interpretation that AI agents can call programmatically.

## ✨ Features

- 🚀 **Instant Matching** - Connect to certified ASL interpreters in <10 seconds
- 💰 **Blockchain Payments** - $ASL token escrow and automatic distribution
- 🔌 **Real-Time Updates** - WebSocket connections for live session status
- 📊 **Analytics** - Track usage, costs, and interpreter performance
- 🔐 **Enterprise Ready** - Rate limiting, webhooks, IP whitelisting

## 🎬 Quick Start
```javascript
import { ASLAgent } from '@americansignlanguage/ai-agent-sdk';

const asl = new ASLAgent({
  apiKey: 'asl_live_your_api_key'
});

// Request interpreter
const session = await asl.requestInterpreter({
  userWallet: '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb',
  urgency: 'high',
  estimatedDuration: 30,
  specialization: 'medical'
});

console.log('Video room:', session.videoRoom.url);

// End session
await session.end({ rating: 5 });
```

## 📖 Documentation

**Full API Specification:** [AI-AGENT-API-SPEC.md](./AI-AGENT-API-SPEC.md)

**Key Endpoints:**
- `POST /interpreter/request` - Request ASL interpreter
- `GET /sessions/{id}` - Get session status
- `POST /sessions/{id}/end` - End session
- `GET /interpreters/available` - List available interpreters

## 💡 Use Cases

### Medical Appointments
AI agent books doctor appointment → Calls API for interpreter → Seamless three-way video call

### Customer Service
AI chatbot detects ASL user → Provisions interpreter instantly → Full accessibility

### Emergency Services
AI calls 911 for Deaf user → Auto-provisions interpreter → Life-saving communication

### Business Calls
AI schedules sales call → Deaf prospect joins → Interpreter provided automatically

## 💰 Pricing

| Tier | Requests/Hour | Monthly Cost |
|------|---------------|--------------|
| Free | 100 | $0 |
| Starter | 1,000 | $99 |
| Growth | 10,000 | $499 |
| Enterprise | Unlimited | Custom |

**Per-Minute:** $20 ASL tokens
- 45% to interpreter
- 45% to platform
- 10% cashback to user

## 🚀 Why This Matters

**Current VRI Market:**
- Manual booking process (15-30 min wait)
- No API for automation
- No blockchain payments
- Expensive ($3-5/min)

**Our Solution:**
- API-first, instant matching
- Blockchain-native payments
- Lower rates ($2.25/min)
- Built for AI agents

## 🔧 Integration Examples

**Moltbook Agent:**
```javascript
agent.on('call_initiated', async (call) => {
  if (call.user.language === 'ASL') {
    const session = await asl.requestInterpreter({
      userWallet: call.user.wallet,
      urgency: 'high',
      estimatedDuration: 15
    });
    await call.addParticipant(session.videoRoom.url);
  }
});
```

**Customer Service Bot:**
```python
@bot.on_message
async def handle_message(message):
    if message.language == 'ASL':
        session = asl.request_interpreter(
            user_wallet=message.user.wallet,
            urgency='medium',
            estimated_duration=10
        )
        await bot.send_video_link(message.user, session.video_room.url)
```

## 🌟 Roadmap

- [x] API specification complete
- [ ] Backend implementation (Weeks 1-2)
- [ ] JavaScript SDK (Week 3)
- [ ] Python SDK (Week 4)
- [ ] Moltbook integration (Week 5)
- [ ] Public beta launch (Week 6)
- [ ] Production launch - April 15, 2026 (National ASL Day)

## 🤝 Contributing

We welcome contributions! Areas where we need help:

- Backend API implementation
- SDK development (JS, Python, Go)
- AI agent integrations
- Documentation improvements
- Testing and QA

**See:** [CONTRIBUTING.md](./CONTRIBUTING.md) (coming soon)

## 📞 Contact

**Support:** asltranscribe@gmail.com

**Website:** https://americansignlanguage.xyz

**Social:**
- Instagram: [@americansignlanguage.eth](https://www.instagram.com/americansignlanguage.eth/)
- X/Twitter: [@aslnfts](https://x.com/aslnfts)

## 📜 License

MIT License - see [LICENSE](./LICENSE) for details

## 🙏 Acknowledgments

Built by the Deaf community, for everyone.

**Developed by DeafDev**

© 2025 americansignlanguage.eth

---

**Making AI agents accessible to 48 million Deaf and hard-of-hearing people** 🚀
