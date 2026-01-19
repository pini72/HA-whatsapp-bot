# Green API WhatsApp Conversation Blueprint for Home Assistant

Control your Home Assistant via WhatsApp! This blueprint integrates Green API to receive WhatsApp messages, process them with any conversation agent (Google Gemini, OpenAI, Claude, etc.), and send smart replies back.

## Features

✅ **Webhook Integration** - Receives WhatsApp messages via Green API webhooks  
✅ **Authorized Numbers** - Only specified phone numbers can interact with the bot  
✅ **Conversation Context** - Maintains conversation history throughout the day (resets daily)  
✅ **Automatic Language Detection** - Responds in the same language as the message  
✅ **Any Conversation Agent** - Works with Home Assistant, Google Gemini, OpenAI, Claude, etc.  
✅ **Queue Mode** - Handles up to 10 simultaneous messages

## Requirements

### 1. Configure Assist in Home Assistant

Set up a conversation agent (AI) to process messages:

**Recommended: Google Gemini (Free & Good)**
1. Follow the setup guide: [Google Generative AI Conversation Integration](https://www.home-assistant.io/integrations/google_generative_ai_conversation/)
2. After adding the integration, go to **Settings > Voice Assistants**
3. Select your Gemini agent as default
4. Configure the language you want

**Other Options:**
- Home Assistant's built-in agent (English only)
- OpenAI ChatGPT
- Anthropic Claude
- Any other conversation agent

### 2. Dedicated WhatsApp Account

⚠️ **Important:** Your personal WhatsApp account won't work!

**Why:** Green API free tier limits you to **3 chats per month**. The first 3 chats that send messages are locked for the entire month - no way to change them.

**Solution:** Create a dedicated WhatsApp account:
1. Download **WhatsApp Business** (separate app)
2. Register with a different phone number
3. Use this account exclusively for the bot

### 3. Green API Account

1. Sign up at [green-api.com](https://green-api.com)
2. Create an instance
3. Get your **Instance ID** and **API Token**
4. Free tier: 3 chats/month, 1000 messages/month

### 4. REST Command Configuration

Add this to your `configuration.yaml`:

```yaml
rest_command:
  green_api_send_message:
    url: "{{ api_url }}/waInstance{{ id_instance }}/sendMessage/{{ api_token }}"
    method: POST
    headers:
      Content-Type: "application/json"
    payload: >-
      {
        "chatId": "{{ chat_id }}",
        "message": "{{ message }}"
      }
    content_type: "application/json"
    timeout: 30
    verify_ssl: true
```

Then **restart Home Assistant**.

### 5. External Access to Home Assistant

You need a public HTTPS URL for webhooks to work.

**Options:**
- ✅ **Nabu Casa Cloud** - Easiest ($6.50/month)
- ✅ **DuckDNS** - Free (requires port forwarding + SSL setup)
- ✅ **Cloudflare Tunnel** - Requires domain (~$10-15/year)

**Firewall/Security Layer:**

If you use additional security (firewall, reverse proxy, etc.), whitelist these Green API IPs:
```
46.101.109.139
51.250.12.167
51.250.84.44
51.250.95.149
89.169.137.216
158.160.49.84
165.22.93.202
167.172.162.71
104.248.252.93
158.160.139.176
64.226.111.11
207.154.255.195
```

## Installation

### Step 1: Import Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/pini72/HA-whatsapp-bot/main/green_api_whatsapp_conversation.yaml)

Or manually:
1. Go to **Settings > Automations & Scenes > Blueprints**
2. Click **"Import Blueprint"**
3. Paste the blueprint URL

### Step 2: Create Automation

1. **Settings > Automations & Scenes**
2. Click **"Create Automation"** > **"Create new automation from blueprint"**
3. Select **"Green API WhatsApp Conversation"**

### Step 3: Configure Blueprint

Fill in the following fields:

**Webhook ID:**
- Enter a unique random ID
- Example: `GSAhFI5ZwxvcXrvtHbs`
- 💡 Use a password generator for security
- **Remember this ID** - you'll need it in Step 4

**ID Instance:**
- Your Green API Instance ID
- Example: `7103136020`
- Find in Green API dashboard

**API Token Instance:**
- Your Green API Instance Token
- Find in Green API dashboard

**Allowed Phone Numbers:**
- Enter phone numbers (digits only, with country code)
- No spaces, dashes, or + sign
- Example: `972555555555`
- One number per line
- ⚠️ **Free tier limit: 3 numbers maximum**

**Conversation Agent:**
- Select the agent you configured in Step 1
- Example: `Google Generative AI Conversation`

**Save the automation!** ✅

### Step 4: Configure Green API Webhook

Now configure the webhook in Green API:

1. Log in to [Green API Console](https://console.green-api.com)
2. Select your instance
3. Go to **Settings > Webhooks**
4. Set **Webhook URL** by combining these 3 parts:

```
[Your HA External URL] + /api/webhook/ + [Webhook ID from Step 3]
```

**Example:**
```
https://abcdef123456.ui.nabu.casa/api/webhook/GSAhFI5ZwxvcXrvtHbs
```

**URL Structure:**
- `https://abcdef123456.ui.nabu.casa` ← Your external Home Assistant URL
- `/api/webhook/` ← Fixed path (don't change)
- `GSAhFI5ZwxvcXrvtHbs` ← The Webhook ID you created in Step 3

5. Enable: **Receive webhooks on incoming messages and files**
6. Click **Save**

✅ **Done! Send a WhatsApp message to test!**

## Usage

### Basic Interaction

The bot automatically detects and responds in the language you use:

**English:**
```
You: Turn on the living room lights
Bot: I've turned on the living room lights
```

**Hebrew:**
```
You: הדלק את האורות בסלון
Bot: הדלקתי את האורות בסלון
```

**Any other language works too!**

### Conversation Context

The bot maintains context throughout the day:

**Message 1 (09:00):**
```
You: Turn on the bedroom light
Bot: There are 2 lights in the bedroom - ceiling light or bedside lamp. Which one?
```

**Message 2 (09:01):**
```
You: Bedside lamp
Bot: I've turned on the bedside lamp
```

**Message 3 (14:30):**
```
You: Turn it off
Bot: I've turned off the bedside lamp
```

The context automatically resets at midnight.

## How It Works

### Conversation ID

Each user gets a unique daily conversation ID:

```
Format: YYYYMMDD_[8-char-hash]
Example: 20260116_8a3f2b1c
```

**Components:**
- Date: `20260116` (changes at midnight)
- Hash: `8a3f2b1c` (unique per phone number)

**Behavior:**
- Same day + same user = same conversation
- New day = automatic reset
- Different users = separate conversations

### Workflow

1. **Message received** → Green API webhook
2. **Authorization check** → Is number in allowed list?
3. **Message processing** → conversation.process with context
4. **Response generation** → AI agent generates reply
5. **Send to WhatsApp** → REST command sends message back

## Advanced Configuration

### Multiple Users

The blueprint automatically handles multiple users:
- Each user has their own conversation context
- Conversations are isolated from each other
- All users share the same authorized number list

## Troubleshooting

### Messages Not Received

**Check logs:**
```
Settings > System > Logs
Search: "WhatsApp message received"
```

**Verify webhook:**
```bash
curl -X POST https://YOUR_HA_URL/api/webhook/YOUR_WEBHOOK_ID \
  -H "Content-Type: application/json" \
  -d '{"typeWebhook":"incomingMessageReceived","senderData":{"sender":"972501234567@c.us"},"messageData":{"typeMessage":"textMessage","textMessageData":{"textMessage":"test"}}}'
```

**Check Green API webhook:**
- Green API Console > Your Instance > Webhooks
- Verify URL is correct
- Check webhook is enabled

### No Response Sent

**Check logs:**
```
Search: "Preparing to send WhatsApp message"
```

Verify:
- API URL is automatically constructed from Instance ID (first 4 digits)
- Instance ID is correct
- API Token is valid
- REST command is configured

**Test REST command:**
```yaml
service: rest_command.green_api_send_message
data:
  api_url: "https://7103.api.greenapi.com"
  id_instance: "YOUR_ID"
  api_token: "YOUR_TOKEN"
  chat_id: "972501234567@c.us"
  message: "Test message"
```

### Unauthorized Sender

If a non-authorized number sends a message:
- The automation stops at the condition check
- No response is sent
- Check logs: "Stopped because a condition failed"

**Solution:** Add the number to "Allowed Phone Numbers"

### Instance Not Authorized

```bash
curl "https://api.green-api.com/waInstance{YOUR_ID}/getStateInstance/{YOUR_TOKEN}"
```

Should return:
```json
{"stateInstance": "authorized"}
```

If not "authorized":
- Log in to Green API Console
- Scan QR code again

### Context Not Working

**Verify in logs:**
```
Conversation ID: 20260116_8a3f2b1c
```

- Same day = same ID ✅
- Different day = different ID ✅

**Test:**
1. Send: "Turn on living room light"
2. Send: "Turn it off"
3. If bot understands "it" = context works ✅

## Logging

The blueprint logs to Home Assistant system logs:

**Message received:**
```
WhatsApp message received from John (972501234567): Hello
Agent: conversation.google_generative_ai
Conversation ID: 20260116_8a3f2b1c
```

**Response generated:**
```
Conversation response: Hi! How can I help you today?
```

**Sending details:**
```
Preparing to send WhatsApp message:
API URL: https://7103.api.greenapi.com
Instance ID: 7103128030
Chat ID: 972501234567@c.us
Message length: 32 chars
```

**Success:**
```
WhatsApp message sent successfully to 972501234567@c.us
```

## Security Best Practices

🔒 **Authorization:**
- Only add trusted phone numbers
- Review the list regularly
- Remove numbers when no longer needed

🔒 **API Credentials:**
- Keep API token secret
- Don't share in public forums
- Use environment secrets if possible

🔒 **Network Access:**
- Use HTTPS only
- Verify SSL certificates
- Consider IP whitelisting if available

🔒 **Logging:**
- Review logs periodically
- Watch for unauthorized access attempts
- Monitor for unusual patterns

## FAQ

### Q: What languages are supported?
**A:** All languages! The bot automatically detects the language of your message and responds in the same language.

### Q: Can I use this with multiple Green API instances?
**A:** Yes, create separate automations for each instance with different webhook IDs.

### Q: Does this work with WhatsApp groups?
**A:** Yes, but you'll need to authorize the group ID instead of individual numbers.

### Q: How much does Green API cost?
**A:** Check [Green API pricing](https://green-api.com/pricing.html). They offer free tier (3 chats/month) and paid plans.

### Q: Can I customize the bot's responses?
**A:** Yes, through your Conversation Agent's configuration (e.g., system prompts in Google Gemini).

### Q: What's the message rate limit?
**A:** Depends on Green API plan. Free tier: 1000 messages/month.

### Q: Can I send images or files?
**A:** This blueprint handles text only. Green API supports media, but requires code modifications.

### Q: Does this work offline?
**A:** No, requires internet connection for both Home Assistant and Green API.

### Q: Can I run multiple automations?
**A:** Yes, with different webhook IDs and configurations.

## Support

- **Issues:** Report on GitHub
- **Questions:** Home Assistant Community Forum
- **Green API Support:** [support@green-api.com](mailto:support@green-api.com)

## Credits

Created for the Home Assistant community.

Powered by:
- [Home Assistant](https://www.home-assistant.io/)
- [Green API](https://green-api.com/)

## License

This blueprint is provided as-is under MIT License.

## Changelog

### v1.0.0 (2026-01-19)

- Initial release
- Basic WhatsApp integration
- Conversation context support
- Multi-user support
- Comprehensive logging
- English documentation

---

**Enjoy your WhatsApp-powered smart home! 🏠📱**
