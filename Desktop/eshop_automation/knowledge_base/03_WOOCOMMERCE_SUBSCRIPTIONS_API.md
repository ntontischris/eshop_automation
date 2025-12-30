# WooCommerce Subscriptions REST API - Complete Reference

## Overview

Base URL: `https://yourstore.com/wp-json/wc/v3/`

Requires: WooCommerce Subscriptions plugin

---

## 1. SUBSCRIPTIONS ENDPOINTS

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/subscriptions` | List subscriptions |
| GET | `/subscriptions/{id}` | Get subscription |
| POST | `/subscriptions` | Create subscription |
| PUT | `/subscriptions/{id}` | Update subscription |
| DELETE | `/subscriptions/{id}` | Delete subscription |
| POST | `/subscriptions/batch` | Batch operations |

---

## 2. SUBSCRIPTION PROPERTIES (Complete)

```json
{
  "id": 100,
  "parent_id": 99,
  "status": "active",
  "currency": "EUR",
  "version": "4.0.0",
  "prices_include_tax": false,
  "date_created": "2024-01-15T10:00:00",
  "date_modified": "2024-01-20T15:00:00",
  "discount_total": "0.00",
  "discount_tax": "0.00",
  "shipping_total": "5.00",
  "shipping_tax": "1.20",
  "cart_tax": "4.80",
  "total": "29.99",
  "total_tax": "6.00",
  "customer_id": 5,
  "order_key": "wc_order_xxxxx",
  "billing": {
    "first_name": "John",
    "last_name": "Doe",
    "company": "",
    "address_1": "123 Main St",
    "address_2": "",
    "city": "Athens",
    "state": "AT",
    "postcode": "10431",
    "country": "GR",
    "email": "john@example.com",
    "phone": "+30 210 1234567"
  },
  "shipping": {
    "first_name": "John",
    "last_name": "Doe",
    "company": "",
    "address_1": "123 Main St",
    "address_2": "",
    "city": "Athens",
    "state": "AT",
    "postcode": "10431",
    "country": "GR"
  },
  "payment_method": "stripe",
  "payment_method_title": "Credit Card (Stripe)",
  "customer_ip_address": "",
  "customer_user_agent": "",
  "created_via": "checkout",
  "customer_note": "",
  "date_completed": null,
  "date_paid": "2024-01-15T10:05:00",
  "number": "100",
  "meta_data": [],
  "line_items": [
    {
      "id": 1,
      "name": "Monthly Subscription Box",
      "product_id": 50,
      "variation_id": 0,
      "quantity": 1,
      "tax_class": "",
      "subtotal": "24.99",
      "subtotal_tax": "6.00",
      "total": "24.99",
      "total_tax": "6.00",
      "taxes": [],
      "meta_data": [],
      "sku": "SUB-001",
      "price": 24.99
    }
  ],
  "tax_lines": [],
  "shipping_lines": [
    {
      "id": 1,
      "method_title": "Flat Rate",
      "method_id": "flat_rate",
      "total": "5.00",
      "total_tax": "1.20",
      "taxes": []
    }
  ],
  "fee_lines": [],
  "coupon_lines": [],
  "billing_period": "month",
  "billing_interval": "1",
  "start_date": "2024-01-15T10:00:00",
  "trial_end_date": "",
  "next_payment_date": "2024-02-15T10:00:00",
  "end_date": "",
  "resubscribed_from": "",
  "resubscribed_subscription": "",
  "removed_line_items": [],
  "payment_details": {
    "post_meta": {
      "_stripe_customer_id": "cus_xxxxx",
      "_stripe_source_id": "src_xxxxx"
    }
  }
}
```

---

## 3. SUBSCRIPTION STATUSES

| Status | Description |
|--------|-------------|
| `pending` | Awaiting payment |
| `active` | Active and renewing |
| `on-hold` | Suspended (payment failed/manual) |
| `cancelled` | Cancelled by user or admin |
| `switched` | Switched to different subscription |
| `expired` | Reached end date |
| `pending-cancel` | Cancelled but still active until period end |

---

## 4. BILLING PERIODS

| Period | Description |
|--------|-------------|
| `day` | Daily |
| `week` | Weekly |
| `month` | Monthly |
| `year` | Yearly |

---

## 5. CREATE SUBSCRIPTION

```json
POST /wp-json/wc/v3/subscriptions

{
  "customer_id": 5,
  "status": "active",
  "billing_period": "month",
  "billing_interval": "1",
  "start_date": "2024-01-15T10:00:00",
  "next_payment_date": "2024-02-15T10:00:00",
  "payment_method": "stripe",
  "payment_method_title": "Credit Card",
  "billing": {
    "first_name": "John",
    "last_name": "Doe",
    "address_1": "123 Main St",
    "city": "Athens",
    "postcode": "10431",
    "country": "GR",
    "email": "john@example.com",
    "phone": "+30 210 1234567"
  },
  "shipping": {
    "first_name": "John",
    "last_name": "Doe",
    "address_1": "123 Main St",
    "city": "Athens",
    "postcode": "10431",
    "country": "GR"
  },
  "line_items": [
    {
      "product_id": 50,
      "quantity": 1
    }
  ],
  "shipping_lines": [
    {
      "method_id": "flat_rate",
      "method_title": "Flat Rate",
      "total": "5.00"
    }
  ],
  "payment_details": {
    "post_meta": {
      "_stripe_customer_id": "cus_xxxxx",
      "_stripe_source_id": "src_xxxxx"
    }
  }
}
```

---

## 6. UPDATE SUBSCRIPTION

### Change Status

```json
PUT /wp-json/wc/v3/subscriptions/{id}

{
  "status": "on-hold"
}
```

### Update Next Payment Date

```json
PUT /wp-json/wc/v3/subscriptions/{id}

{
  "next_payment_date": "2024-03-01T10:00:00"
}
```

### Change Billing Address

```json
PUT /wp-json/wc/v3/subscriptions/{id}

{
  "billing": {
    "address_1": "456 New Street",
    "city": "Thessaloniki"
  }
}
```

### Update Line Items

```json
PUT /wp-json/wc/v3/subscriptions/{id}

{
  "line_items": [
    {
      "id": 1,
      "quantity": 2
    }
  ]
}
```

---

## 7. BATCH OPERATIONS

```json
POST /wp-json/wc/v3/subscriptions/batch

{
  "update": [
    {
      "id": 100,
      "status": "on-hold"
    },
    {
      "id": 101,
      "status": "cancelled"
    }
  ],
  "delete": [102, 103]
}
```

---

## 8. QUERY PARAMETERS

| Parameter | Type | Description |
|-----------|------|-------------|
| `context` | string | view, edit |
| `page` | integer | Page number |
| `per_page` | integer | Items per page (max 100) |
| `search` | string | Search term |
| `after` | string | After date (ISO8601) |
| `before` | string | Before date (ISO8601) |
| `exclude` | array | Exclude IDs |
| `include` | array | Include IDs |
| `offset` | integer | Offset |
| `order` | string | asc, desc |
| `orderby` | string | date, id, include, title |
| `status` | string/array | Filter by status |
| `customer` | integer | Filter by customer ID |
| `product` | integer | Filter by product ID |
| `dp` | integer | Decimal points |

### Filter Examples

```
# Active subscriptions
GET /subscriptions?status=active

# Customer's subscriptions
GET /subscriptions?customer=5

# Subscriptions with specific product
GET /subscriptions?product=50

# Active subscriptions created after date
GET /subscriptions?status=active&after=2024-01-01T00:00:00

# Expiring soon (next payment in next 7 days)
GET /subscriptions?status=active&before=2024-01-22T00:00:00
```

---

## 9. SUBSCRIPTION ORDERS

### Get Orders for a Subscription

```
GET /wp-json/wc/v3/subscriptions/{subscription_id}/orders
```

### Get Subscriptions for an Order

```
GET /wp-json/wc/v3/orders/{order_id}/subscriptions
```

---

## 10. SUBSCRIPTION NOTES

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/subscriptions/{id}/notes` | List notes |
| GET | `/subscriptions/{id}/notes/{note_id}` | Get note |
| POST | `/subscriptions/{id}/notes` | Create note |
| DELETE | `/subscriptions/{id}/notes/{note_id}` | Delete note |

### Create Note

```json
POST /wp-json/wc/v3/subscriptions/{id}/notes

{
  "note": "Customer requested schedule change",
  "customer_note": false
}
```

---

## 11. SUBSCRIPTION WEBHOOKS

### Available Topics

| Topic | Description |
|-------|-------------|
| `subscription.created` | New subscription created |
| `subscription.updated` | Subscription updated |
| `subscription.deleted` | Subscription deleted |
| `subscription.restored` | Subscription restored from trash |

### Custom Action Topics

```
action.woocommerce_subscription_status_active
action.woocommerce_subscription_status_on-hold
action.woocommerce_subscription_status_cancelled
action.woocommerce_subscription_status_expired
action.woocommerce_subscription_status_pending-cancel
action.woocommerce_subscription_payment_complete
action.woocommerce_subscription_renewal_payment_failed
action.woocommerce_subscription_renewal_payment_complete
action.woocommerce_scheduled_subscription_payment
action.woocommerce_customer_changed_subscription_to_cancelled
action.woocommerce_subscription_trial_ended
```

---

## 12. SUBSCRIPTION PRODUCT PROPERTIES

When creating subscription products via Products API:

```json
POST /wp-json/wc/v3/products

{
  "name": "Monthly Subscription Box",
  "type": "subscription",
  "regular_price": "29.99",
  "meta_data": [
    {"key": "_subscription_price", "value": "29.99"},
    {"key": "_subscription_period", "value": "month"},
    {"key": "_subscription_period_interval", "value": "1"},
    {"key": "_subscription_length", "value": "0"},
    {"key": "_subscription_sign_up_fee", "value": "0"},
    {"key": "_subscription_trial_period", "value": "day"},
    {"key": "_subscription_trial_length", "value": "7"}
  ]
}
```

### Subscription Meta Keys

| Key | Description |
|-----|-------------|
| `_subscription_price` | Recurring price |
| `_subscription_period` | day, week, month, year |
| `_subscription_period_interval` | How often (1, 2, 3, etc.) |
| `_subscription_length` | Number of periods (0 = forever) |
| `_subscription_sign_up_fee` | One-time signup fee |
| `_subscription_trial_period` | Trial period type |
| `_subscription_trial_length` | Trial length |

---

## 13. VARIABLE SUBSCRIPTIONS

```json
POST /wp-json/wc/v3/products

{
  "name": "Subscription Plan",
  "type": "variable-subscription",
  "attributes": [
    {
      "name": "Plan",
      "visible": true,
      "variation": true,
      "options": ["Basic", "Premium", "Enterprise"]
    }
  ]
}
```

### Create Variation

```json
POST /wp-json/wc/v3/products/{product_id}/variations

{
  "regular_price": "19.99",
  "attributes": [
    {"name": "Plan", "option": "Basic"}
  ],
  "meta_data": [
    {"key": "_subscription_price", "value": "19.99"},
    {"key": "_subscription_period", "value": "month"},
    {"key": "_subscription_period_interval", "value": "1"}
  ]
}
```

---

## 14. RELATED ENDPOINTS

### Get System Status (includes subscription info)

```
GET /wp-json/wc/v3/system_status
```

### Subscription-related Reports

```
GET /wp-json/wc/v3/reports/subscriptions/totals
```

---

## 15. AUTOMATION USE CASES

### 1. Expiring Card Notification

```
Trigger: Schedule daily
→ GET /subscriptions?status=active
→ Check payment method expiry
→ If expiring within 30 days: Send notification
```

### 2. Failed Payment Recovery

```
Trigger: action.woocommerce_subscription_renewal_payment_failed
→ GET /subscriptions/{id}
→ Send recovery email with payment link
→ Schedule retry in 3 days
→ If still failed: Notify admin
```

### 3. Subscription Upgrade/Downgrade

```
Trigger: Customer request webhook
→ GET /subscriptions/{id}
→ Calculate prorated amount
→ PUT /subscriptions/{id} with new line_items
→ Update next_payment_date if needed
→ Send confirmation email
```

### 4. Churn Prevention

```
Trigger: action.woocommerce_subscription_status_pending-cancel
→ GET /subscriptions/{id}
→ Calculate customer value
→ If high value: Send retention offer
→ Create coupon with special discount
→ Send personalized email
```

### 5. Renewal Reminder

```
Trigger: Schedule daily
→ GET /subscriptions?status=active
→ Filter: next_payment_date within 7 days
→ Send reminder email with invoice preview
```

### 6. Subscription Analytics

```
Trigger: Schedule weekly
→ GET /subscriptions?status=active → Count active
→ GET /subscriptions?status=cancelled&after=last_week → Churn
→ GET /reports/subscriptions/totals
→ Calculate MRR, churn rate, LTV
→ Send report to Google Sheets
```
