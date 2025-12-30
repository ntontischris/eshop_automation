# AI Customer Service Agent - Complete Documentation

## Overview

Αυτό είναι ένας **πλήρης AI Customer Service Agent** που αυτοματοποιεί την εξυπηρέτηση πελατών για e-shop με WooCommerce.

### Τι κάνει:

| Λειτουργία | Περιγραφή | Tool |
|------------|-----------|------|
| **Stock Check** | Έλεγχος διαθεσιμότητας προϊόντων | `check_stock` |
| **Order Tracking** | Παρακολούθηση παραγγελίας | `track_order` |
| **Product Search** | Αναζήτηση στον κατάλογο | `search_products` |
| **Product Info** | Λεπτομέρειες προϊόντος | `get_product_info` |
| **Return Request** | Αίτημα επιστροφής | `create_return_request` |
| **Stock Alert** | Ειδοποίηση διαθεσιμότητας | `set_stock_alert` |
| **Human Transfer** | Μεταφορά σε agent | `transfer_to_human` |

### Κανάλια:
- 💬 **Chat Widget** (website)
- 📞 **Voice** (Vapi AI)
- 📱 **SMS** (Yuboto)

---

## Quick Setup (5 λεπτά)

### 1. Import το Workflow

```
n8n → Workflows → Import from File → ai_customer_service_agent_COMPLETE.json
```

### 2. Δημιούργησε Credentials

#### A. Anthropic API (για Claude AI)
```
n8n → Settings → Credentials → Add → Anthropic API
Name: Anthropic API
API Key: sk-ant-xxxxx (από https://console.anthropic.com)
```

#### B. WooCommerce API
```
n8n → Settings → Credentials → Add → HTTP Basic Auth
Name: WooCommerce API
Username: ck_xxxxx (Consumer Key)
Password: cs_xxxxx (Consumer Secret)
```

Για να πάρεις WooCommerce API keys:
```
WordPress Admin → WooCommerce → Settings → Advanced → REST API → Add Key
```

#### C. (Optional) Yuboto SMS
```
n8n → Settings → Credentials → Add → Header Auth
Name: Yuboto API
Header Name: Authorization
Header Value: Bearer YOUR_YUBOTO_API_KEY
```

### 3. Αντικατέστησε τα Placeholders

Στο workflow, βρες και αντικατέστησε:

| Placeholder | Τι να βάλεις |
|-------------|--------------|
| `ANTHROPIC_CREDENTIAL_ID` | Το ID του Anthropic credential |
| `WOO_CREDENTIAL_ID` | Το ID του WooCommerce credential |
| `YOUR_SLACK_WEBHOOK` | Slack Incoming Webhook URL |
| `YOUR_GOOGLE_SHEET_ID` | Google Sheet ID για logging |
| `$credentials.woocommerceApi.url` | Το URL του WooCommerce site |

### 4. Activate!

Κάνε enable το workflow και είσαι έτοιμος!

---

## Webhook URLs

Μετά την ενεργοποίηση, θα έχεις αυτά τα URLs:

```
Chat:  https://your-n8n.com/webhook/ai-customer-service
Voice: https://your-n8n.com/webhook/ai-voice-agent
SMS:   https://your-n8n.com/webhook/ai-sms-agent
```

---

## Chat Widget Integration

### Απλό HTML/JavaScript Widget

```html
<!-- Βάλε πριν το </body> -->
<div id="ai-chat-widget"></div>
<script>
(function() {
  const WEBHOOK_URL = 'https://your-n8n.com/webhook/ai-customer-service';

  // Create chat button
  const button = document.createElement('div');
  button.innerHTML = '💬';
  button.style.cssText = 'position:fixed;bottom:20px;right:20px;width:60px;height:60px;background:#007bff;border-radius:50%;display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:24px;box-shadow:0 4px 12px rgba(0,0,0,0.3);z-index:9999;';

  // Create chat container
  const chat = document.createElement('div');
  chat.style.cssText = 'position:fixed;bottom:90px;right:20px;width:350px;height:500px;background:white;border-radius:12px;box-shadow:0 4px 20px rgba(0,0,0,0.2);display:none;flex-direction:column;z-index:9999;overflow:hidden;';
  chat.innerHTML = `
    <div style="background:#007bff;color:white;padding:15px;font-weight:bold;">🤖 Βοηθός Εξυπηρέτησης</div>
    <div id="chat-messages" style="flex:1;overflow-y:auto;padding:15px;"></div>
    <div style="padding:10px;border-top:1px solid #eee;display:flex;gap:10px;">
      <input id="chat-input" type="text" placeholder="Γράψτε το μήνυμά σας..." style="flex:1;padding:10px;border:1px solid #ddd;border-radius:20px;outline:none;">
      <button id="chat-send" style="background:#007bff;color:white;border:none;border-radius:20px;padding:10px 20px;cursor:pointer;">→</button>
    </div>
  `;

  document.body.appendChild(button);
  document.body.appendChild(chat);

  let sessionId = 'session_' + Date.now();

  button.onclick = () => {
    chat.style.display = chat.style.display === 'none' ? 'flex' : 'none';
  };

  const addMessage = (text, isUser) => {
    const msg = document.createElement('div');
    msg.style.cssText = `margin:10px 0;padding:10px 15px;border-radius:15px;max-width:80%;${isUser ? 'background:#007bff;color:white;margin-left:auto;' : 'background:#f0f0f0;'}`;
    msg.textContent = text;
    document.getElementById('chat-messages').appendChild(msg);
    document.getElementById('chat-messages').scrollTop = 99999;
  };

  const sendMessage = async () => {
    const input = document.getElementById('chat-input');
    const text = input.value.trim();
    if (!text) return;

    addMessage(text, true);
    input.value = '';

    try {
      const res = await fetch(WEBHOOK_URL, {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({
          message: text,
          session_id: sessionId,
          channel: 'chat'
        })
      });
      const data = await res.json();
      addMessage(data.response, false);
    } catch(e) {
      addMessage('Σφάλμα σύνδεσης. Δοκιμάστε ξανά.', false);
    }
  };

  document.getElementById('chat-send').onclick = sendMessage;
  document.getElementById('chat-input').onkeypress = (e) => {
    if (e.key === 'Enter') sendMessage();
  };

  // Welcome message
  setTimeout(() => addMessage('Γεια σας! 👋 Πώς μπορώ να σας βοηθήσω;', false), 500);
})();
</script>
```

---

## Vapi Voice Integration

### 1. Δημιούργησε Vapi Assistant

Πήγαινε στο https://vapi.ai και:

```json
{
  "name": "Eshop Customer Service",
  "model": {
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514"
  },
  "voice": {
    "provider": "elevenlabs",
    "voiceId": "greek-voice-id"
  },
  "serverUrl": "https://your-n8n.com/webhook/ai-voice-agent"
}
```

### 2. Σύνδεσε με Yuboto SIP

Στο Yuboto MyVoIP:
```
SIP Server: sip.yuboto-telephony.gr
Forward to: Vapi SIP URI
```

---

## SMS Integration (Yuboto)

### Incoming SMS Webhook

Στο Yuboto Octapush panel:
```
Webhook URL: https://your-n8n.com/webhook/ai-sms-agent
Method: POST
Format: JSON
```

### Outgoing SMS

Αυτόματα στέλνει απάντηση μέσω Yuboto API.

---

## Παραδείγματα Συνομιλιών

### Stock Check
```
👤: Έχετε διαθέσιμο το iPhone 15 Pro;

🤖: ✅ Βρέθηκαν 2 διαθέσιμα προϊόντα:

• iPhone 15 Pro 128GB (SKU: IP15P-128)
  Τιμή: 1.199€ | Stock: 5 τεμάχια

• iPhone 15 Pro 256GB (SKU: IP15P-256)
  Τιμή: 1.349€ | Stock: 3 τεμάχια

Θέλετε περισσότερες πληροφορίες για κάποιο;
```

### Order Tracking
```
👤: Πού είναι η παραγγελία μου #12345;

🤖: 📦 Παραγγελία #12345

Κατάσταση: 🚚 Απεστάλη
Ημερομηνία: 28/12/2024
Σύνολο: 89.99€

Προϊόντα:
• Ασύρματα ακουστικά x1 - 89.99€

🚚 Tracking Number: ACS123456789

📬 Η παραγγελία σας έχει αποσταλεί!
Αναμένετε παράδοση εντός 1-3 εργάσιμων ημερών.
```

### Product Search
```
👤: Ψάχνω για laptop gaming μέχρι 1000€

🤖: 🔍 Βρέθηκαν 4 προϊόντα:

1. ASUS TUF Gaming F15
   Τιμή: ~~999€~~ 849€ 🏷️ | ✅ Διαθέσιμο
   Intel i5, RTX 4050, 16GB RAM...

2. Lenovo IdeaPad Gaming 3
   Τιμή: 799€ | ✅ Διαθέσιμο
   AMD Ryzen 5, RTX 3050, 8GB RAM...

💡 Για περισσότερες πληροφορίες, πείτε μου
τον αριθμό του προϊόντος που σας ενδιαφέρει.
```

### Return Request
```
👤: Θέλω να επιστρέψω το προϊόν από την παραγγελία #12345

🤖: Λυπάμαι που δεν μείνατε ικανοποιημένος.
Ποιος είναι ο λόγος επιστροφής;

👤: Δεν μου ταιριάζει

🤖: ✅ Αίτημα Επιστροφής Καταχωρήθηκε

Αριθμός Αιτήματος: RET-ABC123
Παραγγελία: #12345
Λόγος: Δεν μου ταιριάζει

📧 Θα λάβετε email με τις οδηγίες επιστροφής.

⏱️ Η ομάδα μας θα επεξεργαστεί το αίτημά σας
εντός 24 ωρών.
```

---

## Logging & Analytics

### Slack Notifications
Κάθε συνομιλία καταγράφεται στο Slack με:
- Channel (chat/voice/sms)
- Customer info
- Message & Response

### Google Sheets
Όλες οι συνομιλίες αποθηκεύονται σε spreadsheet:
- Timestamp
- Channel
- Customer
- Message
- AI Response
- Session ID

---

## Customization

### Αλλαγή Γλώσσας AI

Στο node "🧠 AI Agent", τροποποίησε το System Message:

```
Είσαι ο AI Βοηθός Εξυπηρέτησης Πελατών...
```

### Προσθήκη Νέου Tool

1. Δημιούργησε Sub-Workflow με Webhook trigger
2. Πρόσθεσε Tool node στον AI Agent
3. Σύνδεσε τα

### Αλλαγή AI Model

Στο node "Claude Sonnet":
- `claude-sonnet-4-20250514` (balanced)
- `claude-opus-4-20250514` (best quality)
- `claude-3-5-haiku-20241022` (fastest)

---

## Troubleshooting

### "Tool not responding"
- Βεβαιώσου ότι τα sub-workflow webhooks είναι active
- Έλεγξε τα WooCommerce credentials

### "No products found"
- Έλεγξε αν το WooCommerce API URL είναι σωστό
- Βεβαιώσου ότι υπάρχουν published products

### "SMS not sending"
- Έλεγξε το Yuboto API key
- Βεβαιώσου ότι το phone number είναι σε σωστή μορφή (+30...)

---

## Architecture Diagram

```
                    ┌─────────────────────┐
                    │     ΠΕΛΑΤΗΣ         │
                    └─────────┬───────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   ┌─────────┐          ┌─────────┐          ┌─────────┐
   │  Chat   │          │  Voice  │          │   SMS   │
   │ Widget  │          │  Vapi   │          │ Yuboto  │
   └────┬────┘          └────┬────┘          └────┬────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             ▼
                    ┌─────────────────────┐
                    │   n8n Webhooks      │
                    └─────────┬───────────┘
                              ▼
                    ┌─────────────────────┐
                    │   Normalize Input   │
                    └─────────┬───────────┘
                              ▼
                    ┌─────────────────────┐
                    │   🧠 AI AGENT       │
                    │   (Claude Sonnet)   │
                    │                     │
                    │   ┌─────────────┐   │
                    │   │   TOOLS:    │   │
                    │   │ check_stock │   │
                    │   │ track_order │   │
                    │   │ search_prod │   │
                    │   │ product_info│   │
                    │   │ return_req  │   │
                    │   │ stock_alert │   │
                    │   │ human_trans │   │
                    │   └─────────────┘   │
                    └─────────┬───────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   ┌─────────┐          ┌─────────┐          ┌─────────┐
   │WooComm  │          │  Slack  │          │ Google  │
   │  API    │          │  Logs   │          │ Sheets  │
   └─────────┘          └─────────┘          └─────────┘
```

---

## Support

Αν χρειάζεσαι βοήθεια:
1. Έλεγξε τα n8n execution logs
2. Κοίτα τα Slack logs
3. Δοκίμασε τα API endpoints manually

---

## Version History

- **v1.0.0** - Initial release with full functionality
  - 7 AI tools
  - Multi-channel support (chat, voice, SMS)
  - Greek language optimized
  - WooCommerce integration
  - Slack & Google Sheets logging
