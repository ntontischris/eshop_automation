# WooCommerce Order Notification Workflow - Documentation

## Αρχιτεκτονική Workflow

```
WooCommerce Webhook
        │
        ▼
  Parse Order Data (Code Node)
        │
        ├──► Slack #orders (HTTP Request)
        │
        ├──► SMS Warehouse (HTTP Request)
        │
        ├──► Google Sheets - Orders Log
        │
        └──► VIP Check (IF Node)
                   │
                   ▼ (if total > 200€)
             Slack #vip-orders
```

---

## 1. WooCommerce Webhook Setup

### Στο WooCommerce Admin:
**WooCommerce → Settings → Advanced → Webhooks → Add webhook**

```
Name: n8n New Order Notification
Status: Active
Topic: Order created
Delivery URL: https://your-n8n-instance.com/webhook/woocommerce-new-order
Secret: your-webhook-secret-key
API Version: WP REST API Integration v3
```

### Webhook Payload Structure (WooCommerce sends):
```json
{
  "id": 12345,
  "number": "12345",
  "status": "processing",
  "currency": "EUR",
  "total": "259.99",
  "payment_method": "stripe",
  "payment_method_title": "Credit Card (Stripe)",
  "date_created": "2024-01-15T10:30:00",
  "customer_note": "Παρακαλώ δέμα με ανακυκλώσιμα υλικά",
  "billing": {
    "first_name": "Γιάννης",
    "last_name": "Παπαδόπουλος",
    "email": "giannis@example.com",
    "phone": "+306912345678",
    "address_1": "Ερμού 25",
    "address_2": "",
    "city": "Αθήνα",
    "postcode": "10563",
    "country": "GR"
  },
  "shipping": {
    "first_name": "Γιάννης",
    "last_name": "Παπαδόπουλος",
    "address_1": "Ερμού 25",
    "address_2": "",
    "city": "Αθήνα",
    "postcode": "10563",
    "country": "GR"
  },
  "line_items": [
    {
      "id": 1,
      "name": "Premium Widget Pro",
      "quantity": 2,
      "sku": "WGT-PRO-001",
      "total": "159.98"
    },
    {
      "id": 2,
      "name": "Accessory Pack",
      "quantity": 1,
      "sku": "ACC-001",
      "total": "100.01"
    }
  ]
}
```

---

## 2. HTTP Requests - Αναλυτικά

### 2.1 Slack #orders Channel

**HTTP Request:**
```http
POST https://slack.com/api/chat.postMessage
Content-Type: application/json
Authorization: Bearer xoxb-your-slack-bot-token

{
  "channel": "#orders",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "🛒 Νέα Παραγγελία #12345",
        "emoji": true
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Πελάτης:*\nΓιάννης Παπαδόπουλος"
        },
        {
          "type": "mrkdwn",
          "text": "*Σύνολο:*\n€259.99"
        },
        {
          "type": "mrkdwn",
          "text": "*Email:*\ngiannis@example.com"
        },
        {
          "type": "mrkdwn",
          "text": "*Τηλέφωνο:*\n+306912345678"
        },
        {
          "type": "mrkdwn",
          "text": "*Τρόπος Πληρωμής:*\nCredit Card (Stripe)"
        },
        {
          "type": "mrkdwn",
          "text": "*Κατάσταση:*\nprocessing"
        }
      ]
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*📦 Προϊόντα:*\n• Premium Widget Pro x2 - €159.98\n• Accessory Pack x1 - €100.01"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*📍 Διεύθυνση Αποστολής:*\nΕρμού 25, Αθήνα, 10563, GR"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*📝 Σημείωση Πελάτη:*\nΠαρακαλώ δέμα με ανακυκλώσιμα υλικά"
      }
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": "Ημερομηνία: 2024-01-15T10:30:00"
        }
      ]
    }
  ]
}
```

**cURL Command:**
```bash
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer xoxb-your-slack-bot-token" \
  -d '{
    "channel": "#orders",
    "blocks": [
      {
        "type": "header",
        "text": {
          "type": "plain_text",
          "text": "🛒 Νέα Παραγγελία #12345",
          "emoji": true
        }
      },
      {
        "type": "section",
        "fields": [
          {"type": "mrkdwn", "text": "*Πελάτης:*\nΓιάννης Παπαδόπουλος"},
          {"type": "mrkdwn", "text": "*Σύνολο:*\n€259.99"}
        ]
      }
    ]
  }'
```

---

### 2.2 SMS Warehouse (Yuboto/Apifon Example)

**HTTP Request (Yuboto):**
```http
POST https://api.yuboto.com/sms/v2/send
Content-Type: application/json
Authorization: Bearer your-yuboto-api-key

{
  "messages": [
    {
      "to": ["+306912345678", "+306987654321"],
      "from": "ESHOP",
      "text": "📦 ΝΕΑ ΠΑΡΑΓΓΕΛΙΑ #12345\nΓιάννης Παπαδόπουλος\n€259.99\n2 προϊόντα\nΔιεύθυνση: Ερμού 25, Αθήνα, 10563"
    }
  ]
}
```

**HTTP Request (Apifon):**
```http
POST https://ars.apifon.com/services/api/v1/sms/send
Content-Type: application/json
X-Api-Key: your-apifon-api-key

{
  "subscriber": {
    "number": "+306912345678"
  },
  "message": {
    "text": "📦 ΝΕΑ ΠΑΡΑΓΓΕΛΙΑ #12345\nΓιάννης Παπαδόπουλος\n€259.99\n2 προϊόντα\nΔιεύθυνση: Ερμού 25, Αθήνα",
    "sender_id": "ESHOP"
  }
}
```

**HTTP Request (Twilio):**
```http
POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(AccountSid:AuthToken)

To=+306912345678
From=+15551234567
Body=📦 ΝΕΑ ΠΑΡΑΓΓΕΛΙΑ #12345 - Γιάννης Παπαδόπουλος - €259.99
```

---

### 2.3 Slack #vip-orders Channel

**HTTP Request:**
```http
POST https://slack.com/api/chat.postMessage
Content-Type: application/json
Authorization: Bearer xoxb-your-slack-bot-token

{
  "channel": "#vip-orders",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "⭐ VIP ΠΑΡΑΓΓΕΛΙΑ #12345",
        "emoji": true
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*🎉 Μεγάλη παραγγελία άνω των €200!*"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Πελάτης:*\nΓιάννης Παπαδόπουλος"
        },
        {
          "type": "mrkdwn",
          "text": "*Σύνολο:*\n💰 €259.99"
        },
        {
          "type": "mrkdwn",
          "text": "*Email:*\ngiannis@example.com"
        },
        {
          "type": "mrkdwn",
          "text": "*Τηλέφωνο:*\n+306912345678"
        }
      ]
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*📦 Προϊόντα:*\n• Premium Widget Pro x2 - €159.98\n• Accessory Pack x1 - €100.01"
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "📞 Κάλεσε Πελάτη"
          },
          "url": "tel:+306912345678",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "📧 Στείλε Email"
          },
          "url": "mailto:giannis@example.com"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "🔗 Δες στο WooCommerce"
          },
          "url": "https://yourstore.com/wp-admin/post.php?post=12345&action=edit"
        }
      ]
    }
  ]
}
```

---

## 3. Google Sheets Integration

### 3.1 Sheet Structure - "Orders Log"

| Column | Header | Data Type |
|--------|--------|-----------|
| A | Order ID | Number |
| B | Order Number | Text |
| C | Date | DateTime |
| D | Customer Name | Text |
| E | Customer Email | Email |
| F | Customer Phone | Phone |
| G | Billing Address | Text |
| H | Shipping Address | Text |
| I | Items | Text |
| J | Total | Currency |
| K | Currency | Text |
| L | Payment Method | Text |
| M | Status | Text |
| N | Customer Note | Text |
| O | VIP Order | Text (YES/NO) |

### 3.2 n8n Code Node για Google Sheets Data Preparation

```javascript
// Full code for parsing and preparing data for Google Sheets
const order = $input.first().json.body || $input.first().json;

// Parse order data
const orderData = {
  order_id: order.id || order.order_id,
  order_number: order.number || order.order_number || order.id,
  status: order.status,
  total: parseFloat(order.total || 0),
  currency: order.currency || 'EUR',
  payment_method: order.payment_method_title || order.payment_method,

  // Customer info
  customer_name: `${order.billing?.first_name || ''} ${order.billing?.last_name || ''}`.trim(),
  customer_email: order.billing?.email || '',
  customer_phone: order.billing?.phone || '',

  // Billing address
  billing_address: [
    order.billing?.address_1,
    order.billing?.address_2,
    order.billing?.city,
    order.billing?.postcode,
    order.billing?.country
  ].filter(Boolean).join(', '),

  // Shipping address
  shipping_address: [
    order.shipping?.address_1,
    order.shipping?.address_2,
    order.shipping?.city,
    order.shipping?.postcode,
    order.shipping?.country
  ].filter(Boolean).join(', '),

  // Order items
  items: (order.line_items || []).map(item => ({
    name: item.name,
    quantity: item.quantity,
    price: parseFloat(item.total || 0),
    sku: item.sku || 'N/A'
  })),

  // Formatted items for display
  items_formatted: (order.line_items || []).map(item =>
    `• ${item.name} x${item.quantity} - €${parseFloat(item.total || 0).toFixed(2)}`
  ).join('\n'),

  // For sheets (semicolon separated)
  items_for_sheets: (order.line_items || []).map(item =>
    `${item.name} x${item.quantity}`
  ).join('; '),

  // Additional info
  customer_note: order.customer_note || '',
  date_created: order.date_created || new Date().toISOString(),

  // VIP flag
  is_vip_order: parseFloat(order.total || 0) > 200
};

return { json: orderData };
```

### 3.3 Google Sheets API Direct HTTP Request

```http
POST https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/Orders%20Log!A:O:append?valueInputOption=USER_ENTERED
Content-Type: application/json
Authorization: Bearer {oauth2-access-token}

{
  "values": [
    [
      12345,
      "12345",
      "2024-01-15T10:30:00",
      "Γιάννης Παπαδόπουλος",
      "giannis@example.com",
      "+306912345678",
      "Ερμού 25, Αθήνα, 10563, GR",
      "Ερμού 25, Αθήνα, 10563, GR",
      "Premium Widget Pro x2; Accessory Pack x1",
      259.99,
      "EUR",
      "Credit Card (Stripe)",
      "processing",
      "Παρακαλώ δέμα με ανακυκλώσιμα υλικά",
      "YES"
    ]
  ]
}
```

---

## 4. Environment Variables (n8n)

Πρόσθεσε στο n8n settings:

```env
# Slack
SLACK_BOT_TOKEN=xoxb-your-token-here

# SMS Provider
SMS_API_URL=https://api.yuboto.com/sms/v2/send
SMS_API_KEY=your-sms-api-key
WAREHOUSE_PHONE=+306912345678

# Google Sheets
GOOGLE_SHEET_ID=1ABCdefghijklmnop...

# WooCommerce
WOOCOMMERCE_STORE_URL=https://yourstore.com
```

---

## 5. Credentials Setup στο n8n

### 5.1 Slack Bot Token
```
Type: HTTP Header Auth
Header Name: Authorization
Header Value: Bearer xoxb-your-slack-bot-token
```

### 5.2 SMS API Key
```
Type: HTTP Header Auth
Header Name: Authorization
Header Value: Bearer your-sms-api-key
```

### 5.3 Google Sheets OAuth2
```
Type: Google Sheets OAuth2
Client ID: your-client-id.apps.googleusercontent.com
Client Secret: your-client-secret
Scopes: https://www.googleapis.com/auth/spreadsheets
```

---

## 6. Error Handling - Enhanced Workflow

### Code Node για Error Handling:

```javascript
// Wrap in try-catch για safe execution
try {
  const order = $input.first().json.body || $input.first().json;

  // Validate required fields
  if (!order.id && !order.order_id) {
    throw new Error('Missing order ID');
  }

  if (!order.billing?.email) {
    console.log('Warning: No customer email provided');
  }

  // Parse order data...
  const orderData = {
    order_id: order.id || order.order_id,
    // ... rest of parsing
  };

  return { json: { ...orderData, success: true } };

} catch (error) {
  // Log error and return fallback
  console.error('Order parsing failed:', error.message);

  return {
    json: {
      success: false,
      error: error.message,
      raw_data: $input.first().json
    }
  };
}
```

---

## 7. Testing

### Test Webhook με cURL:

```bash
curl -X POST https://your-n8n-instance.com/webhook/woocommerce-new-order \
  -H "Content-Type: application/json" \
  -d '{
    "id": 99999,
    "number": "99999",
    "status": "processing",
    "currency": "EUR",
    "total": "259.99",
    "payment_method_title": "Test Payment",
    "date_created": "2024-01-15T10:30:00",
    "billing": {
      "first_name": "Test",
      "last_name": "Customer",
      "email": "test@example.com",
      "phone": "+306900000000",
      "address_1": "Test Street 1",
      "city": "Athens",
      "postcode": "10000",
      "country": "GR"
    },
    "shipping": {
      "first_name": "Test",
      "last_name": "Customer",
      "address_1": "Test Street 1",
      "city": "Athens",
      "postcode": "10000",
      "country": "GR"
    },
    "line_items": [
      {
        "name": "Test Product",
        "quantity": 2,
        "sku": "TEST-001",
        "total": "259.99"
      }
    ]
  }'
```

---

## 8. Slack Bot Permissions Required

Στο Slack App Dashboard, πρόσθεσε τα εξής OAuth Scopes:

- `chat:write` - Αποστολή μηνυμάτων
- `chat:write.public` - Αποστολή σε public channels χωρίς invite
- `channels:read` - Ανάγνωση channel info (optional)

---

## 9. Alternative: Slack Incoming Webhook (Simpler)

Αν δεν θέλεις Bot Token, μπορείς να χρησιμοποιήσεις Incoming Webhook:

```http
POST https://hooks.slack.com/services/YOUR_TEAM_ID/YOUR_BOT_ID/YOUR_WEBHOOK_TOKEN
Content-Type: application/json

{
  "channel": "#orders",
  "username": "WooCommerce Bot",
  "icon_emoji": ":shopping_cart:",
  "blocks": [...]
}
```

---

## Author
Created for eshop_automation project
