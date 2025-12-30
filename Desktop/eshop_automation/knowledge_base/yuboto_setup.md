# Yuboto Setup Guide - SMS & Voice με Vapi Integration

## Περιεχόμενα

1. [Επισκόπηση](#επισκόπηση)
2. [ΜΕΡΟΣ Α: SMS Setup (Octapush)](#μεροσ-α-sms-setup-octapush)
3. [ΜΕΡΟΣ Β: VoIP & Τηλεφωνικό Νούμερο](#μεροσ-β-voip--τηλεφωνικό-νούμερο)
4. [ΜΕΡΟΣ Γ: Vapi Voice Agent Integration](#μεροσ-γ-vapi-voice-agent-integration)
5. [Technical Reference](#technical-reference)
6. [Κόστη & Τιμοκατάλογος](#κόστη--τιμοκατάλογος)
7. [Troubleshooting](#troubleshooting)

---

## Επισκόπηση

Η Yuboto είναι Ελληνική εταιρεία τηλεπικοινωνιών (αδειοδοτημένη από ΕΕΤΤ) που προσφέρει:

| Υπηρεσία | Platform | Χρήση |
|----------|----------|-------|
| **SMS** | Octapush | Αποστολή SMS μέσω API |
| **VoIP** | MyNumber / MyVoIP | Τηλεφωνικά νούμερα & SIP Trunk |
| **Viber** | Octapush | Business messages |

### Αρχιτεκτονική Integration

```
┌─────────────────────────────────────────────────────────────────┐
│                    YUBOTO ECOSYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│   │  OCTAPUSH   │     │  MYNUMBER   │     │   MYVOIP    │      │
│   │  (SMS/Viber)│     │  (Αριθμοί)  │     │  (Features) │      │
│   └──────┬──────┘     └──────┬──────┘     └──────┬──────┘      │
│          │                   │                   │              │
│          │                   │                   │              │
│          ▼                   ▼                   ▼              │
│   ┌─────────────────────────────────────────────────────┐      │
│   │              YUBOTO SIP INFRASTRUCTURE              │      │
│   │              sip.yuboto-telephony.gr                │      │
│   └─────────────────────────────────────────────────────┘      │
│                              │                                  │
└──────────────────────────────┼──────────────────────────────────┘
                               │
                               ▼
                    ┌─────────────────┐
                    │   YOUR APPS     │
                    │   n8n / Vapi    │
                    └─────────────────┘
```

---

## ΜΕΡΟΣ Α: SMS Setup (Octapush)

### A1. Δημιουργία Λογαριασμού

#### Βήμα 1: Εγγραφή
1. Πήγαινε στο: **https://octapush.yuboto.com/en-US/Register**
2. Συμπλήρωσε:
   - Email (θα χρησιμοποιηθεί για login)
   - Password (min 8 χαρακτήρες)
   - Όνομα εταιρείας
   - ΑΦΜ (προαιρετικό για τιμολόγηση)
3. Αποδέξου τους όρους χρήσης
4. Πάτα **Register**

#### Βήμα 2: Επιβεβαίωση Email
1. Έλεγξε το inbox σου
2. Κάνε click στο verification link
3. Ο λογαριασμός είναι έτοιμος

#### Βήμα 3: Πρώτο Login
1. Πήγαινε: **https://octapush.yuboto.com**
2. Εισήγαγε email & password
3. Θα δεις το dashboard

---

### A2. Απόκτηση API Key

#### Μέθοδος 1: Μέσω Dashboard
1. Login στο Octapush
2. Πάνω δεξιά, click στο **όνομά σου**
3. Επίλεξε **"Developers"**
4. Click **"API Key"**
5. Αντίγραψε το key (μορφή: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

#### Μέθοδος 2: Μέσω Email
1. Στείλε email στο: **support@yuboto.com**
2. Θέμα: "API Key Request - [Company Name]"
3. Θα λάβεις το key σε 24-48 ώρες

> **ΣΗΜΑΝΤΙΚΟ**: Κράτα το API Key ασφαλές! Μην το μοιράζεσαι και μην το βάζεις σε public repositories.

---

### A3. Αγορά SMS Credits

#### Βήμα 1: Προσθήκη Credits
1. Στο dashboard, πήγαινε **"Plans"** → **"SMS Plans"**
2. Επίλεξε πακέτο:

| Πακέτο | SMS | Τιμή (εκτίμηση) |
|--------|-----|-----------------|
| Starter | 100 | ~5€ |
| Basic | 500 | ~20€ |
| Pro | 2000 | ~70€ |
| Enterprise | 10000+ | Κατόπιν συμφωνίας |

3. Επίλεξε τρόπο πληρωμής:
   - Πιστωτική/Χρεωστική κάρτα
   - PayPal
   - Τραπεζική κατάθεση

#### Βήμα 2: Επιβεβαίωση
1. Μετά την πληρωμή, τα credits εμφανίζονται στο dashboard
2. Έλεγξε το **Balance** πάνω δεξιά

---

### A4. Ρύθμιση Sender ID

#### Τι είναι το Sender ID
Το όνομα που εμφανίζεται στον παραλήπτη αντί για αριθμό (π.χ. "ESHOP" αντί για "+306912345678")

#### Διαδικασία Έγκρισης
1. Πήγαινε **Settings** → **Sender IDs**
2. Click **"Add New Sender ID"**
3. Συμπλήρωσε:
   - **Sender Name**: π.χ. "ESHOP" (max 11 χαρακτήρες, χωρίς κενά)
   - **Company Name**: Η επωνυμία σου
   - **Use Case**: Περιγραφή χρήσης
4. Submit για έγκριση

> **Χρόνος έγκρισης**: 24-48 εργάσιμες ώρες

#### Περιορισμοί Sender ID
- Μέγιστο 11 χαρακτήρες
- Μόνο alphanumeric (A-Z, 0-9)
- Χωρίς κενά ή ειδικούς χαρακτήρες
- Δεν μπορεί να μοιάζει με αριθμό τηλεφώνου

---

### A5. SMS API Reference

#### Base URL
```
https://omni.yuboto.com/api/v1/
```

#### Authentication
Το API Key περνάει στο header:
```
api_key: YOUR_API_KEY
```

#### Send SMS - HTTP Request

```http
POST https://omni.yuboto.com/api/v1/send
Content-Type: application/json
api_key: YOUR_API_KEY_HERE

{
  "messages": [
    {
      "recipients": ["+306912345678"],
      "from": "ESHOP",
      "text": "Το μήνυμά σας εδώ"
    }
  ]
}
```

#### Send SMS - cURL Example

```bash
curl -X POST https://omni.yuboto.com/api/v1/send \
  -H "Content-Type: application/json" \
  -H "api_key: YOUR_API_KEY" \
  -d '{
    "messages": [
      {
        "recipients": ["+306912345678", "+306987654321"],
        "from": "ESHOP",
        "text": "Παραγγελία #12345 εστάλη!"
      }
    ]
  }'
```

#### Send SMS - JavaScript/Node.js

```javascript
const axios = require('axios');

async function sendSMS(recipients, message, senderId = 'ESHOP') {
  try {
    const response = await axios.post(
      'https://omni.yuboto.com/api/v1/send',
      {
        messages: [{
          recipients: Array.isArray(recipients) ? recipients : [recipients],
          from: senderId,
          text: message
        }]
      },
      {
        headers: {
          'Content-Type': 'application/json',
          'api_key': process.env.YUBOTO_API_KEY
        }
      }
    );

    return response.data;
  } catch (error) {
    console.error('SMS Error:', error.response?.data || error.message);
    throw error;
  }
}

// Χρήση
sendSMS('+306912345678', 'Δοκιμαστικό μήνυμα!');
```

#### Send SMS - Python

```python
import requests
import os

def send_sms(recipients, message, sender_id='ESHOP'):
    url = 'https://omni.yuboto.com/api/v1/send'

    headers = {
        'Content-Type': 'application/json',
        'api_key': os.environ.get('YUBOTO_API_KEY')
    }

    payload = {
        'messages': [{
            'recipients': recipients if isinstance(recipients, list) else [recipients],
            'from': sender_id,
            'text': message
        }]
    }

    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Χρήση
send_sms(['+306912345678'], 'Δοκιμαστικό μήνυμα!')
```

#### Response Format

**Success:**
```json
{
  "status": "success",
  "message_id": "msg_123456789",
  "credits_used": 1,
  "credits_remaining": 499
}
```

**Error:**
```json
{
  "status": "error",
  "code": "INVALID_RECIPIENT",
  "message": "Invalid phone number format"
}
```

#### Διαθέσιμα API Commands

| Command | Περιγραφή |
|---------|-----------|
| `Send` | Αποστολή SMS |
| `Dlr` | Delivery Report (status) |
| `Cost` | Κόστος μηνύματος |
| `Balance` | Υπόλοιπο credits |
| `Cancel` | Ακύρωση scheduled SMS |

---

### A6. n8n Integration για SMS

#### HTTP Request Node Configuration

```json
{
  "method": "POST",
  "url": "https://omni.yuboto.com/api/v1/send",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {
        "name": "Content-Type",
        "value": "application/json"
      },
      {
        "name": "api_key",
        "value": "={{ $env.YUBOTO_API_KEY }}"
      }
    ]
  },
  "sendBody": true,
  "specifyBody": "json",
  "jsonBody": {
    "messages": [
      {
        "recipients": ["{{ $json.customer_phone }}"],
        "from": "ESHOP",
        "text": "Παραγγελία #{{ $json.order_number }} - €{{ $json.total }}"
      }
    ]
  }
}
```

---

## ΜΕΡΟΣ Β: VoIP & Τηλεφωνικό Νούμερο

### B1. Εγγραφή στο MyNumber

#### Βήμα 1: Δημιουργία Λογαριασμού
1. Πήγαινε: **https://services.yuboto.com/mynumber/**
2. Click **"REGISTER"**
3. Συμπλήρωσε:
   - Email
   - Password
   - Στοιχεία εταιρείας/ιδιώτη
   - Τηλέφωνο επικοινωνίας
4. Επιβεβαίωσε το email

#### Βήμα 2: Πρώτο Login
1. Login στο MyNumber
2. Θα δεις το dashboard με τις διαθέσιμες υπηρεσίες

---

### B2. Αγορά Τηλεφωνικού Νούμερου

#### Διαθέσιμοι Τύποι Αριθμών

| Τύπος | Παράδειγμα | Χρήση |
|-------|------------|-------|
| **Γεωγραφικός** | 211-XXX-XXXX (Αθήνα) | Τοπική παρουσία |
| **Γεωγραφικός** | 231-XXX-XXXX (Θεσσαλονίκη) | Τοπική παρουσία |
| **800** | 800-XXX-XXXX | Δωρεάν για καλούντα |
| **Διεθνής** | +44, +49, κλπ | Παρουσία εξωτερικού |

#### Διαθέσιμες Πόλεις Ελλάδας (61 πόλεις)
- Αθήνα (211)
- Θεσσαλονίκη (231)
- Πάτρα (261)
- Ηράκλειο (281)
- Και άλλες 57 πόλεις...

#### Διαδικασία Αγοράς
1. Στο MyNumber, click **"Αγορά Νούμερου"**
2. Επίλεξε **τύπο** (γεωγραφικός/800)
3. Επίλεξε **πόλη/περιοχή**
4. Επίλεξε από τα διαθέσιμα νούμερα
5. Προσθήκη στο καλάθι
6. Ολοκλήρωση πληρωμής

---

### B3. Αγορά SIP Trunk

#### Τι είναι το SIP Trunk
Σύνδεση που επιτρέπει τη διασύνδεση του τηλεφωνικού σου συστήματος με το δίκτυο της Yuboto μέσω Internet (VoIP).

#### Διαδικασία
1. Στο MyNumber, πήγαινε **"SIP Trunk"**
2. Επίλεξε πακέτο:

| Πακέτο | Ταυτόχρονες κλήσεις | Χρήση |
|--------|---------------------|-------|
| Basic | 2 | Μικρές επιχειρήσεις |
| Standard | 5 | Μεσαίες επιχειρήσεις |
| Pro | 10+ | Μεγάλες επιχειρήσεις |

3. Μετά την αγορά θα λάβεις:
   - **SIP Username** (συνήθως το νούμερό σου ή custom)
   - **SIP Password**
   - **SIP Domain**: `sip.yuboto-telephony.gr`

---

### B4. SIP Technical Specifications

#### Server Details

| Parameter | Value |
|-----------|-------|
| **SIP Domain** | `sip.yuboto-telephony.gr` |
| **SIP Port (UDP)** | 5060 |
| **SIP Port (TCP)** | 5060 |
| **SIP Port (TLS)** | 5061 |
| **RTP Ports** | 10000-20000 |

#### Supported Codecs

| Codec | Bandwidth | Quality |
|-------|-----------|---------|
| G.711 alaw | 64 kbps | Excellent |
| G.711 ulaw | 64 kbps | Excellent |
| G.729 | 8 kbps | Good |
| GSM | 13 kbps | Fair |

#### IP Addresses για Firewall/Whitelist

```
213.144.173.67
213.144.173.68
213.144.173.69
213.144.173.70
213.144.173.75
```

#### Firewall Rules
Επίτρεψε τα παρακάτω:

```
# SIP Signaling
UDP/TCP 5060 from 213.144.173.67-75
TLS 5061 from 213.144.173.67-75

# RTP Media
UDP 10000-20000 from 213.144.173.67-75
```

#### SIP Registration Example (για softphones/PBX)

```
Username: 2111234567
Password: YourSIPPassword
Domain: sip.yuboto-telephony.gr
Outbound Proxy: sip.yuboto-telephony.gr
Transport: UDP (ή TLS για encryption)
```

---

### B5. MyVoIP - Διαχείριση Features

#### Διαθέσιμα Features

| Feature | Περιγραφή |
|---------|-----------|
| **Call Forwarding** | Προώθηση κλήσεων σε άλλο νούμερο |
| **Voicemail** | Τηλεφωνητής με email notification |
| **Call Recording** | Εγγραφή κλήσεων |
| **IVR** | Interactive Voice Response |
| **Time-based Routing** | Δρομολόγηση βάσει ώρας |

#### Πρόσβαση στο MyVoIP
1. Πήγαινε: **https://services.yuboto.com/myvoip/**
2. Login με τα ίδια credentials του MyNumber
3. Επίλεξε τον αριθμό που θέλεις να διαχειριστείς

---

## ΜΕΡΟΣ Γ: Vapi Voice Agent Integration

### C1. Επισκόπηση Αρχιτεκτονικής

```
┌─────────────────────────────────────────────────────────────────┐
│                    VOICE AGENT FLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Πελάτης καλεί                                                  │
│  211-XXX-XXXX                                                   │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────┐                                            │
│  │  PSTN Network   │  (Ελληνικό τηλεφωνικό δίκτυο)             │
│  └────────┬────────┘                                            │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │  Yuboto SIP     │                                            │
│  │  Trunk Server   │  sip.yuboto-telephony.gr                   │
│  └────────┬────────┘                                            │
│           │                                                     │
│           │  SIP INVITE                                         │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │  Vapi SIP       │                                            │
│  │  Endpoint       │  {credential_id}.sip.vapi.ai               │
│  └────────┬────────┘                                            │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐     ┌─────────────────┐                   │
│  │  Vapi Voice     │────►│  LLM (GPT-4)    │                   │
│  │  Agent Engine   │◄────│  AI Brain       │                   │
│  └────────┬────────┘     └─────────────────┘                   │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │  TTS Engine     │  (ElevenLabs / PlayHT / Azure)            │
│  │  Text-to-Speech │                                            │
│  └────────┬────────┘                                            │
│           │                                                     │
│           │  Audio Response                                     │
│           ▼                                                     │
│      Πελάτης ακούει                                             │
│      τον AI Agent                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### C2. Vapi Account Setup

#### Βήμα 1: Εγγραφή
1. Πήγαινε: **https://vapi.ai**
2. Click **"Get Started"** ή **"Sign Up"**
3. Εγγραφή με:
   - Google Account
   - GitHub Account
   - Email/Password

#### Βήμα 2: Απόκτηση API Key
1. Login στο Vapi Dashboard
2. Πήγαινε **Settings** → **API Keys**
3. Click **"Create API Key"**
4. Αντίγραψε το key (μορφή: `vapi_xxxxxxxx...`)

#### Βήμα 3: Προσθήκη Credits
1. Πήγαινε **Billing**
2. Πρόσθεσε payment method
3. Αγόρασε credits ή ενεργοποίησε pay-as-you-go

---

### C3. Δημιουργία SIP Trunk Credential

#### API Request

```bash
curl -X POST https://api.vapi.ai/credential \
  -H "Authorization: Bearer YOUR_VAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "byo-sip-trunk",
    "name": "Yuboto Greece SIP Trunk",
    "sipTrunkGateway": {
      "provider": "custom",
      "address": "sip.yuboto-telephony.gr",
      "port": 5060,
      "protocol": "udp",
      "authenticationMethod": "credentials",
      "authUsername": "YOUR_YUBOTO_SIP_USERNAME",
      "authPassword": "YOUR_YUBOTO_SIP_PASSWORD"
    },
    "sipTrunkOutboundEnabled": true,
    "sipTrunkInboundEnabled": true,
    "sipTrunkSignalingIps": [
      "213.144.173.67",
      "213.144.173.68",
      "213.144.173.69",
      "213.144.173.70",
      "213.144.173.75"
    ]
  }'
```

#### Response

```json
{
  "id": "cred_abc123xyz",
  "provider": "byo-sip-trunk",
  "name": "Yuboto Greece SIP Trunk",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

> **ΣΗΜΑΝΤΙΚΟ**: Κράτα το `id` - θα το χρειαστείς για τα επόμενα βήματα!

---

### C4. Whitelist Vapi IPs στο Yuboto

#### IPs που πρέπει να επιτρέψεις

```
44.229.228.186
44.238.177.138
```

#### Διαδικασία
1. Επικοινώνησε με Yuboto Support (13777)
2. Ζήτα να προστεθούν τα παραπάνω IPs στο whitelist του SIP trunk σου
3. Εναλλακτικά, αν έχεις πρόσβαση στο SIP trunk panel, πρόσθεσέ τα εσύ

---

### C5. Ρύθμιση Call Forwarding στο Yuboto

#### Inbound Call Routing
Ρύθμισε το Yuboto νούμερό σου να προωθεί κλήσεις στο Vapi:

**SIP URI Format:**
```
sip:+30211XXXXXXX@cred_abc123xyz.sip.vapi.ai
```

Όπου:
- `+30211XXXXXXX` = Το Yuboto νούμερό σου σε E.164 format
- `cred_abc123xyz` = Το credential ID από το Vapi

#### Μέσω MyVoIP
1. Login στο MyVoIP
2. Επίλεξε τον αριθμό σου
3. Πήγαινε **"Call Forwarding"** ή **"SIP Forwarding"**
4. Βάλε το SIP URI: `sip:+30211XXXXXXX@cred_abc123xyz.sip.vapi.ai`
5. Save

#### Μέσω Support
1. Κάλεσε 13777
2. Ζήτα να ρυθμίσουν SIP forwarding για τον αριθμό σου
3. Δώσε τους το SIP URI

---

### C6. Δημιουργία Voice Assistant

#### API Request

```bash
curl -X POST https://api.vapi.ai/assistant \
  -H "Authorization: Bearer YOUR_VAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "E-Shop Customer Support",
    "model": {
      "provider": "openai",
      "model": "gpt-4",
      "temperature": 0.7,
      "systemPrompt": "Είσαι ο εικονικός βοηθός του e-shop μας. Μιλάς Ελληνικά με φιλικό και επαγγελματικό τρόπο.\n\nΜπορείς να βοηθήσεις τους πελάτες με:\n- Πληροφορίες για παραγγελίες (tracking, status)\n- Πληροφορίες για προϊόντα\n- Πολιτική επιστροφών\n- Ώρες λειτουργίας\n- Τρόποι πληρωμής\n\nΑν δεν μπορείς να απαντήσεις, προσφέρου να συνδέσεις τον πελάτη με ανθρώπινο εκπρόσωπο.\n\nΏρες λειτουργίας: Δευτέρα-Παρασκευή 09:00-18:00\nEmail: info@eshop.gr\nΔιεύθυνση: Ερμού 25, Αθήνα"
    },
    "voice": {
      "provider": "elevenlabs",
      "voiceId": "21m00Tcm4TlvDq8ikWAM"
    },
    "firstMessage": "Γεια σας! Καλωσορίσατε στο e-shop μας. Πώς μπορώ να σας εξυπηρετήσω σήμερα;",
    "endCallMessage": "Ευχαριστούμε για την επικοινωνία! Καλή σας μέρα!",
    "transcriber": {
      "provider": "deepgram",
      "model": "nova-2",
      "language": "el"
    },
    "silenceTimeoutSeconds": 30,
    "maxDurationSeconds": 600,
    "backgroundSound": "off",
    "backchannelingEnabled": true,
    "recordingEnabled": true
  }'
```

#### Response

```json
{
  "id": "asst_xyz789abc",
  "name": "E-Shop Customer Support",
  "createdAt": "2024-01-15T10:35:00Z"
}
```

---

### C7. Σύνδεση Phone Number με Assistant

#### API Request

```bash
curl -X POST https://api.vapi.ai/phone-number \
  -H "Authorization: Bearer YOUR_VAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "byo-phone-number",
    "number": "+30211XXXXXXX",
    "credentialId": "cred_abc123xyz",
    "assistantId": "asst_xyz789abc",
    "name": "E-Shop Main Line"
  }'
```

#### Response

```json
{
  "id": "phone_123456",
  "number": "+30211XXXXXXX",
  "assistantId": "asst_xyz789abc",
  "createdAt": "2024-01-15T10:40:00Z"
}
```

---

### C8. Testing

#### Test Inbound Call
1. Κάλεσε το Yuboto νούμερό σου από ένα κινητό
2. Θα πρέπει να ακούσεις τον AI assistant να λέει: "Γεια σας! Καλωσορίσατε..."
3. Μίλα και δες αν απαντάει σωστά

#### Test via Vapi Dashboard
1. Πήγαινε στο Vapi Dashboard
2. Επίλεξε τον Assistant
3. Click **"Test"**
4. Μίλα μέσω browser

#### Troubleshooting Checklist
- [ ] Yuboto SIP credentials είναι σωστά
- [ ] Vapi IPs είναι whitelisted στο Yuboto
- [ ] Call forwarding είναι ρυθμισμένο σωστά
- [ ] Yuboto νούμερο έχει airtime/credits
- [ ] Vapi account έχει credits

---

## Technical Reference

### Yuboto API Endpoints

| Service | Endpoint |
|---------|----------|
| SMS API | `https://omni.yuboto.com/api/v1/` |
| SIP Server | `sip.yuboto-telephony.gr` |
| MyNumber | `https://services.yuboto.com/mynumber/` |
| MyVoIP | `https://services.yuboto.com/myvoip/` |

### Vapi API Endpoints

| Service | Endpoint |
|---------|----------|
| API Base | `https://api.vapi.ai/` |
| SIP Inbound | `{credential_id}.sip.vapi.ai` |

### IP Addresses Summary

#### Yuboto IPs (για Vapi whitelist)
```
213.144.173.67
213.144.173.68
213.144.173.69
213.144.173.70
213.144.173.75
```

#### Vapi IPs (για Yuboto whitelist)
```
44.229.228.186
44.238.177.138
```

---

## Κόστη & Τιμοκατάλογος

### Yuboto Εκτιμήσεις

| Υπηρεσία | Κόστος |
|----------|--------|
| SMS (Ελλάδα) | ~0.03-0.05€/SMS |
| VoIP Νούμερο | ~5-10€/μήνα |
| SIP Trunk | ~15-30€/μήνα |
| Κλήσεις (Ελλάδα) | ~0.01-0.02€/λεπτό |
| Κλήσεις (Κινητά) | ~0.05-0.08€/λεπτό |

### Vapi Εκτιμήσεις

| Υπηρεσία | Κόστος |
|----------|--------|
| Voice Agent | ~$0.05-0.10/λεπτό |
| Transcription | Included |
| Recording Storage | Included (30 days) |

### Third-Party Services

| Υπηρεσία | Κόστος |
|----------|--------|
| OpenAI GPT-4 | ~$0.03/1K input tokens |
| ElevenLabs TTS | ~$0.30/1K characters |
| Deepgram STT | ~$0.0043/λεπτό |

---

## Troubleshooting

### SMS Issues

| Πρόβλημα | Λύση |
|----------|------|
| "Invalid API Key" | Έλεγξε ότι το API key είναι σωστό |
| "Insufficient Credits" | Αγόρασε περισσότερα credits |
| "Invalid Recipient" | Χρησιμοποίησε E.164 format (+30...) |
| "Sender ID Rejected" | Περίμενε έγκριση ή χρησιμοποίησε approved ID |

### SIP/Voice Issues

| Πρόβλημα | Λύση |
|----------|------|
| "401 Unauthorized" | Έλεγξε SIP credentials |
| "No audio" | Έλεγξε firewall για RTP ports |
| "Calls not reaching Vapi" | Έλεγξε call forwarding settings |
| "Vapi can't call out" | Έλεγξε outbound settings & credits |

### Επικοινωνία Support

| Service | Contact |
|---------|---------|
| **Yuboto Technical** | 13777 (δωρεάν) |
| **Yuboto Sales** | 13850 |
| **Yuboto Email** | support@yuboto.com |
| **Vapi Support** | support@vapi.ai |
| **Vapi Discord** | discord.gg/vapi |

---

## Χρήσιμα Links

### Yuboto
- [Octapush Platform](https://octapush.yuboto.com)
- [MyNumber Platform](https://services.yuboto.com/mynumber/)
- [MyVoIP Platform](https://services.yuboto.com/myvoip/)
- [Yuboto Telephony](https://www.yuboto-telephony.gr/)
- [API Documentation PDF](https://octapush.yuboto.com/Data/pushc8b5-35c9-4b8c-9670-8f0dcb2ad364/OMNI_API_Documentation.pdf)

### Vapi
- [Vapi Website](https://vapi.ai)
- [Vapi Documentation](https://docs.vapi.ai)
- [SIP Trunking Guide](https://docs.vapi.ai/advanced/sip/sip-trunk)
- [API Reference](https://docs.vapi.ai/api-reference)

---

## Changelog

| Date | Version | Changes |
|------|---------|---------|
| 2024-12-29 | 1.0 | Initial documentation |

---

*Document created for eshop_automation project*
