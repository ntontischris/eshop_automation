# E-commerce Automation Recipes - Complete Collection

## Overview

Αυτό το αρχείο περιέχει 50+ έτοιμες "συνταγές" αυτοματισμού για WooCommerce με n8n.

---

## CATEGORY 1: ORDER MANAGEMENT

### Recipe 1.1: New Order Notification Pipeline

**Trigger:** Order Created Webhook

**Actions:**
1. Fetch full order details
2. Send Slack notification to #orders channel
3. Send SMS to warehouse team
4. Create task in project management
5. Log to Google Sheets

**HTTP Calls:**
```
GET /wp-json/wc/v3/orders/{order_id}
```

**Slack Message Template:**
```
🛒 New Order #{{order.number}}

Customer: {{order.billing.first_name}} {{order.billing.last_name}}
Email: {{order.billing.email}}
Total: €{{order.total}}

Items:
{{#each order.line_items}}
- {{this.name}} x{{this.quantity}} (€{{this.total}})
{{/each}}

Shipping: {{order.shipping.city}}, {{order.shipping.country}}
```

---

### Recipe 1.2: High-Value Order Alert

**Trigger:** Order Created Webhook

**Condition:** order.total > 200

**Actions:**
1. Send priority Slack notification
2. Add "VIP" tag to order
3. Send personal thank you email
4. Assign to senior team member

**HTTP Calls:**
```
PUT /wp-json/wc/v3/orders/{order_id}
Body: {
  "meta_data": [
    {"key": "_priority", "value": "high"},
    {"key": "_assigned_to", "value": "senior_team"}
  ]
}
```

---

### Recipe 1.3: Automatic Order Status Updates

**Trigger:** Schedule (every 15 minutes)

**Logic:**
- Orders with tracking + 2 days old → Completed
- Orders paid but not shipped after 24h → Alert

**HTTP Calls:**
```
GET /wp-json/wc/v3/orders?status=processing&before={24h_ago}
PUT /wp-json/wc/v3/orders/{id}
Body: {"status": "completed"}
```

---

### Recipe 1.4: Order Fraud Detection

**Trigger:** Order Created Webhook

**Risk Factors:**
- Different billing/shipping country (+30 risk)
- High value order (+20 risk)
- Guest checkout high value (+25 risk)
- Multiple failed orders from same IP (+40 risk)

**Actions if risk > 50:**
1. Set order to on-hold
2. Add admin note with risk factors
3. Send alert to admin
4. Request manual review

**HTTP Calls:**
```
PUT /wp-json/wc/v3/orders/{id}
Body: {"status": "on-hold"}

POST /wp-json/wc/v3/orders/{id}/notes
Body: {
  "note": "⚠️ High Risk Order (Score: 75)\nFactors: Different countries, High value",
  "customer_note": false
}
```

---

### Recipe 1.5: Failed Order Recovery

**Trigger:** Schedule (every hour)

**Target:** Orders with status=failed in last 24h

**Actions:**
1. Send recovery email with payment link
2. After 24h: Send reminder
3. After 48h: Send final notice with discount

**HTTP Calls:**
```
GET /wp-json/wc/v3/orders?status=failed&after={24h_ago}
PUT /wp-json/wc/v3/orders/{id}
Body: {
  "meta_data": [{"key": "_recovery_email_sent", "value": "1"}]
}
```

---

### Recipe 1.6: Order Split for Multiple Warehouses

**Trigger:** Order Created Webhook

**Logic:**
- Check each product's warehouse meta
- If multiple warehouses: Create child orders
- Assign each to respective warehouse

**HTTP Calls:**
```
GET /wp-json/wc/v3/products/{product_id}
POST /wp-json/wc/v3/orders
Body: {
  "parent_id": {original_order_id},
  "line_items": [{warehouse_specific_items}],
  "meta_data": [{"key": "_warehouse", "value": "warehouse_a"}]
}
```

---

## CATEGORY 2: INVENTORY MANAGEMENT

### Recipe 2.1: Low Stock Daily Alert

**Trigger:** Schedule (daily at 9:00)

**Logic:**
1. Get all products with manage_stock=true
2. Filter stock_quantity < threshold (default 10)
3. Group by category
4. Send alert

**HTTP Calls:**
```
GET /wp-json/wc/v3/products?per_page=100&stock_status=instock
```

**Alert Template:**
```
📦 Low Stock Alert - {{date}}

Critical (< 5):
{{#each critical}}
- [{{this.sku}}] {{this.name}}: {{this.stock}} left
{{/each}}

Warning (< 10):
{{#each warning}}
- [{{this.sku}}] {{this.name}}: {{this.stock}} left
{{/each}}

Total products low: {{total}}
```

---

### Recipe 2.2: Automatic Reorder Point Notification

**Trigger:** Product Updated Webhook (stock change)

**Condition:** stock_quantity <= reorder_point (from meta)

**Actions:**
1. Create purchase order draft
2. Email supplier with order details
3. Update product meta with PO reference
4. Log to inventory system

**HTTP Calls:**
```
GET /wp-json/wc/v3/products/{id}
PUT /wp-json/wc/v3/products/{id}
Body: {
  "meta_data": [
    {"key": "_last_po_date", "value": "2024-01-15"},
    {"key": "_last_po_number", "value": "PO-2024-001"}
  ]
}
```

---

### Recipe 2.3: Multi-Store Stock Sync

**Trigger:** Product Updated Webhook from any store

**Actions:**
1. Extract SKU and new stock level
2. For each other store:
   - Find product by SKU
   - Update stock to match
3. Update master inventory (Airtable)

**HTTP Calls (per store):**
```
GET /wp-json/wc/v3/products?sku={sku}
PUT /wp-json/wc/v3/products/{id}
Body: {"stock_quantity": {new_stock}}
```

---

### Recipe 2.4: Dead Stock Identification

**Trigger:** Schedule (weekly)

**Logic:**
1. Get all products
2. For each: Check last sale date from orders
3. If no sale > 90 days: Flag as dead stock
4. Generate report with recommendations

**HTTP Calls:**
```
GET /wp-json/wc/v3/products?per_page=100
GET /wp-json/wc/v3/orders?product={product_id}&per_page=1&orderby=date&order=desc
```

**Report Format:**
```
📊 Dead Stock Report - Week {{week_number}}

No Sales > 90 Days:
| SKU | Product | Stock | Value | Last Sale |
|-----|---------|-------|-------|-----------|
| SKU001 | Product A | 50 | €500 | 120 days |

Recommendations:
- Run clearance sale
- Bundle with popular items
- Consider donation for tax benefits

Total dead stock value: €5,000
```

---

### Recipe 2.5: Stock Forecast & Auto-Replenish

**Trigger:** Schedule (weekly)

**Logic:**
1. Calculate average daily sales (last 30 days)
2. Calculate days until stockout
3. If < 14 days: Trigger reorder

**Calculation:**
```javascript
const avgDailySales = totalSalesLast30Days / 30;
const daysUntilStockout = currentStock / avgDailySales;
const reorderQuantity = avgDailySales * leadTimeDays * safetyMultiplier;
```

---

## CATEGORY 3: CUSTOMER MANAGEMENT

### Recipe 3.1: New Customer Welcome Sequence

**Trigger:** Order Completed (first order)

**Actions:**
1. Day 0: Welcome email with discount code
2. Day 3: Product care tips
3. Day 7: Ask for review
4. Day 14: Cross-sell related products

**HTTP Calls:**
```
POST /wp-json/wc/v3/coupons
Body: {
  "code": "WELCOME10-{customer_id}",
  "discount_type": "percent",
  "amount": "10",
  "individual_use": true,
  "usage_limit": 1,
  "usage_limit_per_user": 1,
  "email_restrictions": ["{customer_email}"]
}
```

---

### Recipe 3.2: Customer Winback Campaign

**Trigger:** Schedule (weekly)

**Target:** Customers with last order > 60 days

**Actions:**
1. Segment by value (high/medium/low)
2. Create personalized coupon
3. Send winback email with coupon

**HTTP Calls:**
```
GET /wp-json/wc/v3/customers?per_page=100
GET /wp-json/wc/v3/orders?customer={id}&per_page=1&orderby=date&order=desc
POST /wp-json/wc/v3/coupons
```

**Segmentation:**
```javascript
const segments = {
  high: { threshold: 500, discount: 20 },
  medium: { threshold: 200, discount: 15 },
  low: { threshold: 0, discount: 10 }
};
```

---

### Recipe 3.3: Birthday/Anniversary Rewards

**Trigger:** Schedule (daily)

**Logic:**
1. Check customer meta for birthday
2. If birthday within 7 days: Send reward

**HTTP Calls:**
```
GET /wp-json/wc/v3/customers?per_page=100
PUT /wp-json/wc/v3/customers/{id}
Body: {
  "meta_data": [{"key": "_birthday_reward_sent_2024", "value": "true"}]
}
```

---

### Recipe 3.4: VIP Customer Identification & Perks

**Trigger:** Order Completed Webhook

**Logic:**
1. Calculate customer lifetime value
2. If LTV > €1000: Upgrade to VIP
3. Apply VIP benefits

**VIP Benefits:**
- Free shipping on all orders
- Priority support
- Early access to sales
- Exclusive discounts

**HTTP Calls:**
```
PUT /wp-json/wc/v3/customers/{id}
Body: {
  "meta_data": [
    {"key": "_customer_tier", "value": "vip"},
    {"key": "_vip_since", "value": "2024-01-15"}
  ]
}
```

---

### Recipe 3.5: Customer Review Request

**Trigger:** Order status → Completed + 7 days

**Actions:**
1. Check if customer already reviewed
2. If not: Send review request
3. If reviewed: Send thank you + share request

**HTTP Calls:**
```
GET /wp-json/wc/v3/products/reviews?reviewer_email={email}
PUT /wp-json/wc/v3/orders/{id}
Body: {
  "meta_data": [{"key": "_review_request_sent", "value": "true"}]
}
```

---

## CATEGORY 4: PRICING & PROMOTIONS

### Recipe 4.1: Dynamic Pricing Based on Stock

**Trigger:** Schedule (daily) or Stock Update Webhook

**Logic:**
- If stock > 100: Normal price
- If stock < 20: Increase by 10%
- If stock < 5: Increase by 20%
- If dead stock: Decrease by 30%

**HTTP Calls:**
```
GET /wp-json/wc/v3/products?per_page=100
PUT /wp-json/wc/v3/products/{id}
Body: {"regular_price": "{new_price}"}
```

---

### Recipe 4.2: Competitor Price Monitoring

**Trigger:** Schedule (daily)

**Actions:**
1. For each product with competitor URL:
   - Scrape competitor price
   - Compare with our price
   - Adjust if needed (beat by X%)
2. Generate price change report

**Price Rules:**
```javascript
const rules = {
  beatBy: 0.05, // Beat competitor by 5%
  minMargin: 0.20, // Never go below 20% margin
  maxDiscount: 0.30 // Max 30% below normal price
};
```

---

### Recipe 4.3: Flash Sale Automation

**Trigger:** Schedule or Manual

**Actions:**
1. Select products for sale
2. Store original prices
3. Apply sale prices
4. Schedule end (restore prices)
5. Send promotional emails/SMS

**HTTP Calls:**
```
POST /wp-json/wc/v3/products/batch
Body: {
  "update": [
    {
      "id": 123,
      "sale_price": "29.99",
      "date_on_sale_from": "2024-01-15T00:00:00",
      "date_on_sale_to": "2024-01-16T23:59:59"
    }
  ]
}
```

---

### Recipe 4.4: Automatic Coupon Expiry Warning

**Trigger:** Schedule (daily)

**Actions:**
1. Get coupons expiring within 3 days
2. For each: Get customers who have it
3. Send reminder email

**HTTP Calls:**
```
GET /wp-json/wc/v3/coupons?per_page=100
# Filter in code: date_expires within 3 days
```

---

### Recipe 4.5: Bundle Pricing Auto-Update

**Trigger:** Component price change

**Logic:**
- When a product price changes
- Find all bundles containing it
- Recalculate bundle price
- Apply discount (e.g., 15% off components total)

---

## CATEGORY 5: SHIPPING & FULFILLMENT

### Recipe 5.1: Shipping Label Auto-Generation

**Trigger:** Order status → Processing

**Actions:**
1. Get order shipping details
2. Call shipping provider API
3. Generate label
4. Add tracking to order
5. Update status to Shipped

**HTTP Calls:**
```
PUT /wp-json/wc/v3/orders/{id}
Body: {
  "meta_data": [
    {"key": "_tracking_number", "value": "1Z999AA10123456784"},
    {"key": "_tracking_provider", "value": "UPS"},
    {"key": "_tracking_url", "value": "https://..."}
  ]
}

POST /wp-json/wc/v3/orders/{id}/notes
Body: {
  "note": "Shipped via UPS. Tracking: 1Z999AA10123456784",
  "customer_note": true
}
```

---

### Recipe 5.2: Delivery Delay Alert

**Trigger:** Schedule (daily)

**Logic:**
1. Get orders shipped > 5 days ago
2. Check tracking status (via carrier API)
3. If not delivered: Alert customer & admin

---

### Recipe 5.3: Local Pickup Notification

**Trigger:** Order status → Processing

**Condition:** Shipping method = local_pickup

**Actions:**
1. Send pickup ready email
2. Generate pickup code
3. Schedule reminder if not picked up

---

### Recipe 5.4: International Order Customs Handler

**Trigger:** Order Created with international shipping

**Actions:**
1. Calculate customs value
2. Generate customs forms
3. Add HS codes to items
4. Include commercial invoice

**HTTP Calls:**
```
GET /wp-json/wc/v3/products/{id}
# Get customs meta: _hs_code, _country_of_origin, _customs_description
```

---

## CATEGORY 6: ANALYTICS & REPORTING

### Recipe 6.1: Daily KPI Dashboard

**Trigger:** Schedule (daily at 8:00)

**Metrics:**
- Total revenue
- Number of orders
- Average order value
- Conversion rate (if available)
- Top products
- Low stock alerts

**HTTP Calls:**
```
GET /wp-json/wc/v3/reports/sales?period=day
GET /wp-json/wc/v3/reports/top_sellers?period=day
GET /wp-json/wc/v3/reports/orders/totals
GET /wp-json/wc/v3/reports/customers/totals
```

---

### Recipe 6.2: Weekly Performance Report

**Trigger:** Schedule (Monday 9:00)

**Sections:**
1. Sales summary (vs last week)
2. Product performance
3. Customer acquisition
4. Inventory status
5. Pending issues

**Output:** PDF report → Email to management

---

### Recipe 6.3: Real-Time Revenue Tracker

**Trigger:** Order Completed Webhook

**Actions:**
1. Update daily total
2. Check against target
3. If milestone reached: Celebrate! 🎉
4. Update live dashboard

---

### Recipe 6.4: Product Performance Analysis

**Trigger:** Schedule (weekly)

**Analysis:**
- Sales velocity (units/day)
- Revenue per product
- Profit margin
- Return rate
- Review score

**Output:** Ranked product list with recommendations

---

### Recipe 6.5: Customer Cohort Analysis

**Trigger:** Schedule (monthly)

**Analysis:**
- New vs returning customers
- Retention rate by month
- LTV by acquisition channel
- Repeat purchase rate

---

## CATEGORY 7: INTEGRATIONS

### Recipe 7.1: Accounting Sync (e.g., QuickBooks)

**Trigger:** Order Completed

**Actions:**
1. Create invoice in accounting
2. Record payment
3. Update inventory
4. Categorize expenses

---

### Recipe 7.2: CRM Sync (e.g., HubSpot)

**Trigger:** New Customer / Order

**Actions:**
1. Create/update contact
2. Add order as deal
3. Update lifecycle stage
4. Trigger workflows

---

### Recipe 7.3: Email Marketing Sync (e.g., Mailchimp)

**Trigger:** Customer events

**Actions:**
1. Add to list on purchase
2. Tag based on purchases
3. Update custom fields
4. Trigger automations

---

### Recipe 7.4: Helpdesk Integration (e.g., Zendesk)

**Trigger:** Order issues

**Actions:**
1. Create ticket on refund request
2. Attach order details
3. Link customer profile
4. Priority based on value

---

### Recipe 7.5: Google Sheets Live Dashboard

**Trigger:** Various events

**Sheets:**
1. Orders log
2. Daily sales summary
3. Inventory levels
4. Customer database
5. KPI tracker

---

## CATEGORY 8: AI-POWERED AUTOMATIONS

### Recipe 8.1: AI Product Descriptions

**Trigger:** Product Created (without description)

**Actions:**
1. Gather product info
2. Call AI (OpenAI/Claude)
3. Generate SEO-optimized content
4. Update product

**Prompt Template:**
```
Create product content for:
Name: {{name}}
Category: {{category}}
Features: {{attributes}}

Generate:
1. SEO title (60 chars)
2. Meta description (155 chars)
3. Short description (2 sentences)
4. Full description (300 words)
```

---

### Recipe 8.2: AI Customer Support Bot

**Trigger:** Chat/Email inquiry

**Capabilities:**
- Order status lookup
- Return initiation
- Product recommendations
- FAQ answers

**Escalation:** Complex issues → Human agent

---

### Recipe 8.3: AI Review Response

**Trigger:** New Review Received

**Actions:**
1. Analyze sentiment
2. Generate appropriate response
3. Queue for approval
4. Post response

---

### Recipe 8.4: AI Sales Forecasting

**Trigger:** Schedule (weekly)

**Input:**
- Historical sales
- Seasonality patterns
- Marketing calendar
- External factors

**Output:**
- Next week forecast
- Inventory recommendations
- Staffing suggestions

---

### Recipe 8.5: AI Fraud Detection

**Trigger:** Order Created

**Analysis:**
- Behavior patterns
- Device fingerprint
- Historical data
- Risk scoring

**Action:** Flag suspicious orders for review

---

## CATEGORY 9: SUBSCRIPTION SPECIFIC

### Recipe 9.1: Subscription Renewal Reminder

**Trigger:** 7 days before renewal

**Actions:**
1. Send reminder email
2. Include upcoming charge amount
3. Option to pause/cancel
4. Update payment method link

---

### Recipe 9.2: Failed Payment Recovery

**Trigger:** Renewal payment failed

**Sequence:**
1. Immediate: Notify customer
2. Day 2: Second attempt + reminder
3. Day 5: Final notice
4. Day 7: Suspend subscription

---

### Recipe 9.3: Subscription Upgrade Prompt

**Trigger:** Customer uses X% of plan limits

**Actions:**
1. Send upgrade suggestion
2. Show benefits of higher tier
3. Offer limited-time discount

---

### Recipe 9.4: Churn Prevention

**Trigger:** Cancel request received

**Actions:**
1. Show exit survey
2. Offer retention deal
3. If churned: Schedule winback

---

## CATEGORY 10: SEASONAL & EVENT-BASED

### Recipe 10.1: Black Friday Preparation

**T-7 Days:**
- Increase stock alerts
- Prepare email campaigns
- Test checkout capacity

**D-Day:**
- Enable flash sales
- Increase monitoring
- Real-time alerts

---

### Recipe 10.2: Holiday Season Automation

**Actions:**
- Extended shipping deadlines
- Gift wrapping options
- Holiday messaging
- Post-holiday returns handling

---

### Recipe 10.3: Product Launch Sequence

**Pre-Launch:**
1. Tease campaign
2. Waitlist collection
3. VIP early access

**Launch:**
1. Enable product
2. Send launch emails
3. Social media posts
4. Monitor inventory

---

This completes the automation recipes. Each recipe can be implemented as a standalone n8n workflow or combined for comprehensive automation.
