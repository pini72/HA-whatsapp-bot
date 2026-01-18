# Green API WhatsApp Conversation Blueprint for Home Assistant

A complete Home Assistant blueprint that enables WhatsApp integration through Green API, allowing you to control your smart home and interact with Home Assistant's Conversation/Assist via WhatsApp messages.

## Features

✅ **Webhook Integration** - Receives WhatsApp messages via Green API webhooks  
✅ **Authorized Numbers** - Only specified phone numbers can interact with the bot  
✅ **Conversation Context** - Maintains conversation history throughout the day (resets daily)  
✅ **Automatic Language Detection** - Responds in the same language as the message (Hebrew, English, etc.)  
✅ **Any Conversation Agent** - Works with Home Assistant, Google Gemini, OpenAI, Anthropic Claude, etc.  
✅ **Automatic Responses** - Sends replies back to WhatsApp automatically  
✅ **Queue Mode** - Handles up to 10 simultaneous messages  
✅ **Detailed Logging** - Comprehensive logging for debugging  

## Requirements

### 1. Green API Account
- Sign up at [green-api.com](https://green-api.com)
- Get your `idInstance` and `apiTokenInstance`
- Set up webhook URL in Green API dashboard

### 2. Home Assistant
- Home Assistant 2023.8 or newer
- **External access (REQUIRED for webhooks):**
  - ✅ **Nabu Casa Cloud** (easiest, recommended - $6.50/month)
  - ✅ **DuckDNS** with Let's Encrypt SSL (free)
  - ✅ **Cloudflare Tunnel** (requires domain purchase, ~$10-15/year)
  - ✅ **Custom domain** with valid SSL certificate
  - ❌ **Local network only** will NOT work (no port 19283 or local IP)

<details>
<summary><b>📖 What is "External Access"?</b></summary>

External access means your Home Assistant is reachable from the internet with a public HTTPS URL.

**Options:**

1. **Nabu Casa Cloud** ($6.50/month)
   - Easiest setup - works in 2 minutes
   - No port forwarding, no configuration
   - URL: `https://abcdef123456.ui.nabu.casa`
   - ✅ Recommended for beginners

2. **DuckDNS** (Free)
   - Free dynamic DNS service
   - Requires port forwarding on router
   - Requires Let's Encrypt SSL setup
   - URL: `https://yourdomain.duckdns.org`
   - ⚙️ Requires technical knowledge

3. **Cloudflare Tunnel** (~$10-15/year for domain)
   - Secure tunnel to your HA via Cloudflare
   - No port forwarding needed!
   - Works behind CGNAT
   - Requires purchasing a domain (~$10-15/year)
   - Cloudflare Tunnel itself is free
   - URL: `https://yourdomain.com` (your own domain)
   - ⚙️ Requires Cloudflare account and domain

4. **Custom Domain**
   - Use your own domain (e.g., `home.yourdomain.com`)
   - Requires domain, port forwarding, and SSL certificate
   - Most flexible but most complex
   - ⚙️ Requires advanced technical knowledge

**Why is this needed?**
Green API needs to send webhooks (HTTP requests) to your Home Assistant when you receive WhatsApp messages. Without external access, Green API cannot reach your Home Assistant.

**Local URLs that DON'T work:**
- ❌ `http://192.168.1.100:8123`
- ❌ `http://homeassistant.local:8123`
- ❌ `http://localhost:8123`

These only work on your local network, not from the internet.
</details>

### 3. REST Command Configuration
Add to your `configuration.yaml`:

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

Then restart Home Assistant.

## Installation

### Step 1: Import Blueprint

[[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/pini72/HA-whatsapp-bot/main/green_api_whatsapp_conversation.yaml)

Or manually:
1. Go to Settings > Automations & Scenes > Blueprints
2. Click "Import Blueprint"
3. Paste the blueprint URL or upload the file

### Step 2: Create Automation and Choose Webhook ID

1. Settings > Automations & Scenes
2. Create Automation > Create new automation from blueprint
3. Select "Green API WhatsApp Conversation"

4. **Create a Webhook ID** - Enter a unique, random identifier:
   
   **Example:** `GSAhFI5ZwxvcXrvtHbs`
   
   💡 **Tip:** Use a password generator or random characters (8-20 characters recommended)

5. Note down this webhook ID for the next step
6. Configure the other fields (see Step 4)
7. **Don't save yet!** First configure Green API (Step 3)

### Step 3: Configure Green API Webhook

Now configure the webhook in Green API using the ID from Step 2:

1. Log in to [Green API Console](https://console.green-api.com)
2. Select your instance
3. Go to Settings > Webhooks
4. Set Webhook URL using your Home Assistant URL and the webhook ID you chose:
   
   **Example:**
   ```
   https://abcdef123456.ui.nabu.casa/api/webhook/GSAhFI5ZwxvcXrvtHbs
   ```
   
   **YOUR_HA_URL options:**
   - **Nabu Casa:** `https://abcdef123456.ui.nabu.casa` (easiest)
   - **DuckDNS:** `https://yourdomain.duckdns.org`
   - **Cloudflare Tunnel:** `https://yourdomain.com` (your custom domain)
   - **Custom domain:** Your own domain with valid SSL
   
   ⚠️ **IMPORTANT:** You MUST have external access to Home Assistant for webhooks to work!
   Local URLs like `http://192.168.1.100:8123` will NOT work.
   
5. Enable: `incomingMessageReceived`
6. Save settings

### Step 4: Complete Automation Setup

Return to Home Assistant and complete the configuration:

#### Configuration Fields:

**Webhook ID:**
- Enter a unique, random identifier
- Example: `GSAhFI5ZwxvcXrvtHbs`
- **Important:** Must match what you configured in Green API (Step 3)
- Use a password generator for random characters

**ID Instance:**
- Example: `7103128030`
- Find in Green API dashboard
- **Note:** The API URL is automatically constructed from the first 4 digits (e.g., `https://7103.api.greenapi.com`)

**API Token Instance:**
- Your instance API token
- Find in Green API dashboard

**Allowed Phone Numbers:**
- Enter phone numbers only (digits), including country code
- No spaces, no dashes, no plus sign (+)
- Example: `972555555555`
- One number per line
- Multiple numbers example:
  ```
  972555555555
  972501234567
  972521234567
  ```

**Conversation Agent:**
- Select from dropdown
- Options: Home Assistant, Google Gemini, OpenAI, etc.
- Default: Home Assistant

**Note:** The bot will automatically detect and respond in the same language as the message received. No manual language configuration needed.

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
curl -X POST https://YOUR_HA_URL/api/webhook/green_api_whatsapp \
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
  api_url: "https://7103.api.greenapi.com"  # First 4 digits of your instance ID
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
**A:** All languages! The bot automatically detects the language of your message and responds in the same language. Works best with Google Gemini, OpenAI, or Claude agents.

### Q: Can I force a specific language?
**A:** The bot responds in the language you use. If you want to force a language, you can add a note in your message like "please respond in English" or just write in that language.

### Q: Does this work with Home Assistant's default agent?
**A:** Yes, but the default Home Assistant agent primarily responds in English. For best multilingual support, use Google Gemini, OpenAI, or Claude.

### Q: Can I use this with multiple Green API instances?
**A:** Yes, create separate automations for each instance with different webhook IDs.

### Q: Does this work with WhatsApp groups?
**A:** Yes, but you'll need to authorize the group ID instead of individual numbers.

### Q: How much does Green API cost?
**A:** Check [Green API pricing](https://green-api.com/pricing.html). They offer free tier and paid plans.

### Q: Can I customize the bot's responses?
**A:** Yes, through your Conversation Agent's configuration (e.g., system prompts in Google Gemini).

### Q: What's the message rate limit?
**A:** Depends on Green API plan. Check your plan details.

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

### v1.0.0 (2026-01-16)
- Initial release
- Basic WhatsApp integration
- Conversation context support
- Multi-user support
- Comprehensive logging
- English documentation

---

**Enjoy your WhatsApp-powered smart home! 🏠📱**
