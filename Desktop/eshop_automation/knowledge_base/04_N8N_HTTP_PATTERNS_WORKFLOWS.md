# n8n HTTP Patterns & Workflows - Complete Guide

## Overview

Αυτό το αρχείο περιέχει patterns, best practices και έτοιμα workflow templates για n8n με WooCommerce/WordPress HTTP requests.

---

## ΜΕΡΟΣ 1: HTTP REQUEST NODE CONFIGURATION

### Basic Setup για WooCommerce

```json
{
  "node": "HTTP Request",
  "parameters": {
    "method": "GET|POST|PUT|DELETE",
    "url": "https://yourstore.com/wp-json/wc/v3/endpoint",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpBasicAuth",
    "options": {
      "timeout": 30000,
      "response": {
        "response": {
          "fullResponse": false
        }
      }
    }
  }
}
```

### Credentials Configuration

```json
{
  "httpBasicAuth": {
    "user": "ck_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "password": "cs_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

### Headers Configuration

```json
{
  "headerParameters": {
    "parameter": [
      {
        "name": "Content-Type",
        "value": "application/json"
      }
    ]
  }
}
```

---

## ΜΕΡΟΣ 2: COMMON HTTP PATTERNS

### Pattern 1: Paginated Fetch (Get All Items)

```
[Start]
  → [Set Variables: page=1, allItems=[]]
  → [Loop]
      → [HTTP: GET /products?page={{page}}&per_page=100]
      → [Merge items to allItems]
      → [IF: hasMorePages?]
          → [YES: page++, goto Loop]
          → [NO: Output allItems]
```

**n8n Implementation:**

```json
{
  "nodes": [
    {
      "name": "Set Initial",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "values": {
          "number": [{"name": "page", "value": 1}],
          "string": [{"name": "allProducts", "value": "[]"}]
        }
      }
    },
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "={{$json.storeUrl}}/wp-json/wc/v3/products",
        "qs": {
          "page": "={{$json.page}}",
          "per_page": 100
        }
      }
    },
    {
      "name": "Check More Pages",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [{
            "value1": "={{$json.length}}",
            "operation": "equal",
            "value2": 100
          }]
        }
      }
    }
  ]
}
```

---

### Pattern 2: Batch Processing (Bulk Updates)

```
[Input: Array of items to update]
  → [Split in Batches: 100 items]
  → [For Each Batch:]
      → [Build batch request body]
      → [HTTP: POST /products/batch]
      → [Collect results]
  → [Merge all results]
  → [Output summary]
```

**Request Body Structure:**

```json
{
  "update": [
    {"id": 1, "regular_price": "29.99"},
    {"id": 2, "regular_price": "39.99"}
  ]
}
```

---

### Pattern 3: Webhook Receiver + Action

```
[Webhook Trigger: /webhook/order-created]
  → [Parse webhook payload]
  → [Validate signature (optional)]
  → [Process order data]
  → [HTTP: GET /orders/{{orderId}} for full details]
  → [Perform actions]
  → [Respond 200 OK]
```

**Webhook Configuration:**

```json
{
  "node": "Webhook",
  "parameters": {
    "path": "woocommerce/order-created",
    "httpMethod": "POST",
    "responseMode": "responseNode",
    "options": {}
  }
}
```

---

### Pattern 4: Conditional Processing

```
[Trigger]
  → [HTTP: GET resource]
  → [Switch by condition:]
      → [Case A: Do action A]
      → [Case B: Do action B]
      → [Default: Log and skip]
```

**n8n Switch Node:**

```json
{
  "node": "Switch",
  "parameters": {
    "dataType": "string",
    "value1": "={{$json.status}}",
    "rules": {
      "rules": [
        {"value2": "processing", "output": 0},
        {"value2": "on-hold", "output": 1},
        {"value2": "completed", "output": 2}
      ]
    }
  }
}
```

---

### Pattern 5: Error Handling & Retry

```
[HTTP Request]
  → [On Error:]
      → [Log error]
      → [Wait 5 seconds]
      → [Retry (max 3 times)]
      → [If still fails: Alert admin]
```

**n8n Error Workflow:**

```json
{
  "node": "HTTP Request",
  "continueOnFail": true,
  "onError": "continueErrorOutput"
}
```

Μετά:
```json
{
  "node": "IF",
  "parameters": {
    "conditions": {
      "boolean": [{
        "value1": "={{$json.error !== undefined}}",
        "value2": true
      }]
    }
  }
}
```

---

### Pattern 6: Scheduled Sync

```
[Schedule Trigger: Every hour]
  → [Get last sync timestamp]
  → [HTTP: GET /orders?after={{lastSync}}]
  → [Process new/updated orders]
  → [Update last sync timestamp]
  → [Log results]
```

---

### Pattern 7: Multi-Store Operations

```
[Trigger]
  → [Get list of stores from config]
  → [For Each Store:]
      → [Set store credentials]
      → [HTTP: Perform operation]
      → [Collect results]
  → [Aggregate results]
  → [Output unified report]
```

**Store Configuration:**

```json
{
  "stores": [
    {
      "name": "Store A",
      "url": "https://store-a.com",
      "key": "ck_xxx",
      "secret": "cs_xxx"
    },
    {
      "name": "Store B",
      "url": "https://store-b.com",
      "key": "ck_yyy",
      "secret": "cs_yyy"
    }
  ]
}
```

---

## ΜΕΡΟΣ 3: COMPLETE WORKFLOW TEMPLATES

### Workflow 1: Daily Sales Report

```yaml
Name: Daily Sales Report
Trigger: Schedule (every day at 8:00)
Steps:
  1. HTTP GET /reports/sales?period=day
  2. HTTP GET /reports/top_sellers?period=day
  3. HTTP GET /orders?status=processing
  4. Calculate KPIs (total, avg order, items)
  5. Format report
  6. Send to Slack/Email
  7. Save to Google Sheets
```

**Full Implementation:**

```json
{
  "name": "Daily Sales Report",
  "nodes": [
    {
      "name": "Schedule",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {"interval": [{"field": "hours", "hoursInterval": 24}]}
      },
      "position": [0, 0]
    },
    {
      "name": "Get Sales Report",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://store.com/wp-json/wc/v3/reports/sales",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpBasicAuth",
        "qs": {"period": "day"}
      },
      "position": [200, 0]
    },
    {
      "name": "Get Top Sellers",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://store.com/wp-json/wc/v3/reports/top_sellers",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpBasicAuth",
        "qs": {"period": "day"}
      },
      "position": [200, 200]
    },
    {
      "name": "Get Pending Orders",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://store.com/wp-json/wc/v3/orders",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpBasicAuth",
        "qs": {"status": "processing", "per_page": 100}
      },
      "position": [200, 400]
    },
    {
      "name": "Merge Data",
      "type": "n8n-nodes-base.merge",
      "parameters": {"mode": "append"},
      "position": [400, 200]
    },
    {
      "name": "Calculate KPIs",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "jsCode": "const sales = $input.first().json;\nconst topSellers = $input.item(1).json;\nconst orders = $input.item(2).json;\n\nreturn [{\n  json: {\n    total_sales: sales.total_sales,\n    total_orders: sales.total_orders,\n    avg_order_value: (parseFloat(sales.total_sales) / sales.total_orders).toFixed(2),\n    pending_orders: orders.length,\n    top_product: topSellers[0]?.title || 'N/A'\n  }\n}];"
      },
      "position": [600, 200]
    },
    {
      "name": "Send Slack",
      "type": "n8n-nodes-base.slack",
      "parameters": {
        "channel": "#sales-reports",
        "text": "📊 Daily Sales Report\n\n💰 Total: €{{$json.total_sales}}\n📦 Orders: {{$json.total_orders}}\n📈 Avg Order: €{{$json.avg_order_value}}\n⏳ Pending: {{$json.pending_orders}}\n🏆 Top: {{$json.top_product}}"
      },
      "position": [800, 200]
    }
  ]
}
```

---

### Workflow 2: Low Stock Alert

```yaml
Name: Low Stock Alert
Trigger: Schedule (every day at 9:00)
Steps:
  1. HTTP GET /products?stock_status=instock&per_page=100
  2. Paginate to get all products
  3. Filter: stock_quantity < threshold
  4. Group by category
  5. Generate report
  6. Send alert (Slack + Email)
  7. Optional: Auto-create PO draft
```

**Filter Expression:**

```javascript
// Code node to filter low stock
const threshold = 10;
const products = $input.all();

const lowStock = products.filter(p =>
  p.json.manage_stock &&
  p.json.stock_quantity < threshold
);

return lowStock.map(p => ({
  json: {
    id: p.json.id,
    name: p.json.name,
    sku: p.json.sku,
    stock: p.json.stock_quantity,
    category: p.json.categories[0]?.name || 'Uncategorized'
  }
}));
```

---

### Workflow 3: Abandoned Cart Recovery

```yaml
Name: Abandoned Cart Recovery
Trigger: Schedule (every hour)
Steps:
  1. HTTP GET /orders?status=pending&after=1_hour_ago&before=now
  2. Filter: orders without reminder_sent meta
  3. For each order:
     a. Get customer email
     b. Generate recovery link
     c. Send email
     d. HTTP PUT /orders/{id} - add meta reminder_sent
  4. Log to spreadsheet
```

**Get Pending Orders:**

```json
{
  "node": "HTTP Request",
  "parameters": {
    "url": "https://store.com/wp-json/wc/v3/orders",
    "qs": {
      "status": "pending",
      "after": "={{DateTime.now().minus({hours: 24}).toISO()}}",
      "before": "={{DateTime.now().minus({hours: 1}).toISO()}}",
      "per_page": 100
    }
  }
}
```

**Mark as Reminded:**

```json
{
  "node": "HTTP Request",
  "parameters": {
    "method": "PUT",
    "url": "https://store.com/wp-json/wc/v3/orders/{{$json.id}}",
    "body": {
      "meta_data": [
        {
          "key": "_cart_recovery_sent",
          "value": "={{DateTime.now().toISO()}}"
        }
      ]
    }
  }
}
```

---

### Workflow 4: Order Status Automation

```yaml
Name: Order Processing Pipeline
Trigger: Webhook (order.created)
Steps:
  1. Receive order webhook
  2. HTTP GET /orders/{id} for full data
  3. Validate order (fraud check)
  4. Check stock availability
  5. If valid:
     a. HTTP PUT /orders/{id} status=processing
     b. Send confirmation email
     c. Create invoice
     d. Notify warehouse
  6. If invalid:
     a. HTTP PUT /orders/{id} status=on-hold
     b. Create admin note
     c. Alert admin
```

**Fraud Check Code:**

```javascript
const order = $input.first().json;

const riskFactors = [];
let riskScore = 0;

// Check billing/shipping mismatch
if (order.billing.country !== order.shipping.country) {
  riskFactors.push('Different billing/shipping countries');
  riskScore += 30;
}

// High value order
if (parseFloat(order.total) > 500) {
  riskFactors.push('High value order');
  riskScore += 20;
}

// New customer with high value
if (order.customer_id === 0 && parseFloat(order.total) > 200) {
  riskFactors.push('Guest checkout high value');
  riskScore += 25;
}

// Free email domain
const freeEmails = ['gmail.com', 'yahoo.com', 'hotmail.com'];
const emailDomain = order.billing.email.split('@')[1];
if (freeEmails.includes(emailDomain) && parseFloat(order.total) > 300) {
  riskFactors.push('Free email + high value');
  riskScore += 15;
}

return [{
  json: {
    ...order,
    risk_score: riskScore,
    risk_factors: riskFactors,
    needs_review: riskScore > 50
  }
}];
```

---

### Workflow 5: Multi-Store Inventory Sync

```yaml
Name: Cross-Store Inventory Sync
Trigger: Webhook (product.updated) from any store
Steps:
  1. Receive product update webhook
  2. Extract SKU and new stock
  3. For each OTHER store:
     a. HTTP GET /products?sku={sku}
     b. If exists: HTTP PUT /products/{id} stock_quantity
  4. Update master inventory (Airtable/Sheets)
  5. Log sync operation
```

**Sync to Other Stores:**

```javascript
// Code to prepare sync requests
const sourceProduct = $input.first().json;
const stores = $env.stores; // Array of store configs

const syncRequests = [];

for (const store of stores) {
  // Skip source store
  if (store.url === sourceProduct.source_store) continue;

  syncRequests.push({
    store: store.name,
    url: store.url,
    sku: sourceProduct.sku,
    stock_quantity: sourceProduct.stock_quantity,
    regular_price: sourceProduct.regular_price
  });
}

return syncRequests.map(r => ({json: r}));
```

---

### Workflow 6: Customer Segmentation

```yaml
Name: Customer Value Segmentation
Trigger: Schedule (weekly)
Steps:
  1. HTTP GET /customers (paginated)
  2. For each customer:
     a. Calculate total spent
     b. Calculate order frequency
     c. Calculate days since last order
  3. Segment:
     - VIP: spent > €1000
     - At Risk: no order 60+ days
     - New: registered < 30 days
  4. Update customer meta with segment
  5. Sync segments to email platform
```

**Segmentation Logic:**

```javascript
const customers = $input.all();
const now = new Date();

return customers.map(c => {
  const customer = c.json;
  const lastOrder = customer.last_order ? new Date(customer.last_order.date_created) : null;
  const daysSinceOrder = lastOrder ? Math.floor((now - lastOrder) / (1000 * 60 * 60 * 24)) : 999;
  const totalSpent = parseFloat(customer.total_spent) || 0;

  let segment;
  if (totalSpent >= 1000) {
    segment = 'VIP';
  } else if (daysSinceOrder > 60 && totalSpent > 0) {
    segment = 'At Risk';
  } else if (daysSinceOrder === 999) {
    segment = 'Never Purchased';
  } else if (customer.orders_count <= 1) {
    segment = 'New';
  } else {
    segment = 'Regular';
  }

  return {
    json: {
      id: customer.id,
      email: customer.email,
      name: `${customer.first_name} ${customer.last_name}`,
      total_spent: totalSpent,
      orders_count: customer.orders_count,
      days_since_order: daysSinceOrder,
      segment: segment
    }
  };
});
```

---

### Workflow 7: AI Product Enrichment

```yaml
Name: AI Product Description Generator
Trigger: Webhook (product.created) OR Manual
Steps:
  1. HTTP GET /products/{id}
  2. Extract: name, categories, attributes
  3. Call OpenAI/Claude:
     - Generate SEO title
     - Generate meta description
     - Generate long description
     - Generate short description
  4. HTTP PUT /products/{id} with AI content
  5. Update Yoast meta
  6. Log changes
```

**OpenAI Prompt Template:**

```javascript
const product = $input.first().json;

const prompt = `You are an expert e-commerce copywriter. Create compelling product content for:

Product: ${product.name}
Category: ${product.categories.map(c => c.name).join(', ')}
Attributes: ${product.attributes.map(a => `${a.name}: ${a.options.join(', ')}`).join('; ')}
Current Price: €${product.price}

Generate:
1. SEO Title (max 60 chars)
2. Meta Description (max 155 chars)
3. Short Description (2-3 sentences, highlights key benefits)
4. Full Description (300-500 words, persuasive, includes features and benefits)

Format as JSON:
{
  "seo_title": "",
  "meta_description": "",
  "short_description": "",
  "full_description": ""
}`;

return [{json: {prompt}}];
```

---

### Workflow 8: Dynamic Pricing

```yaml
Name: Competitor Price Monitor & Auto-Adjust
Trigger: Schedule (daily at 6:00)
Steps:
  1. Get products to monitor
  2. For each product:
     a. Scrape/API competitor prices
     b. Calculate optimal price
     c. If change > 5%: Queue for update
  3. Batch update prices
  4. Generate price change report
  5. Send to admin
```

**Price Calculation Logic:**

```javascript
const product = $input.first().json;
const competitorPrice = product.competitor_price;
const ourCost = product.cost || product.regular_price * 0.6; // Assume 40% margin if no cost
const minMargin = 0.2; // 20% minimum margin

let newPrice;
let reason;

// Calculate based on competitor
if (competitorPrice) {
  const competitorAdjusted = competitorPrice * 0.95; // Beat by 5%
  const minPrice = ourCost * (1 + minMargin);

  if (competitorAdjusted > minPrice) {
    newPrice = competitorAdjusted;
    reason = 'Beat competitor by 5%';
  } else {
    newPrice = minPrice;
    reason = 'Maintain minimum margin';
  }
} else {
  newPrice = product.regular_price;
  reason = 'No competitor data';
}

const priceChange = ((newPrice - product.regular_price) / product.regular_price * 100).toFixed(2);

return [{
  json: {
    id: product.id,
    sku: product.sku,
    name: product.name,
    current_price: product.regular_price,
    new_price: newPrice.toFixed(2),
    price_change_percent: priceChange,
    reason: reason,
    should_update: Math.abs(parseFloat(priceChange)) > 5
  }
}];
```

---

## ΜΕΡΟΣ 4: UTILITY FUNCTIONS

### Date/Time Helpers

```javascript
// Get ISO date for X hours ago
const hoursAgo = (hours) => {
  return new Date(Date.now() - hours * 60 * 60 * 1000).toISOString();
};

// Get ISO date for X days ago
const daysAgo = (days) => {
  return new Date(Date.now() - days * 24 * 60 * 60 * 1000).toISOString();
};

// Start of today
const todayStart = () => {
  const d = new Date();
  d.setHours(0, 0, 0, 0);
  return d.toISOString();
};

// End of today
const todayEnd = () => {
  const d = new Date();
  d.setHours(23, 59, 59, 999);
  return d.toISOString();
};
```

### Pagination Helper

```javascript
// Get all pages of a paginated endpoint
async function getAllPages(baseUrl, credentials, params = {}) {
  let page = 1;
  let allItems = [];
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`${baseUrl}?${new URLSearchParams({
      ...params,
      page: page,
      per_page: 100
    })}`, {
      headers: {
        'Authorization': `Basic ${btoa(credentials.key + ':' + credentials.secret)}`
      }
    });

    const items = await response.json();
    allItems = allItems.concat(items);

    hasMore = items.length === 100;
    page++;
  }

  return allItems;
}
```

### Batch Request Builder

```javascript
// Split array into batches
function splitIntoBatches(items, batchSize = 100) {
  const batches = [];
  for (let i = 0; i < items.length; i += batchSize) {
    batches.push(items.slice(i, i + batchSize));
  }
  return batches;
}

// Build WooCommerce batch request body
function buildBatchBody(items, operation = 'update') {
  return {
    [operation]: items
  };
}
```

### Error Handler

```javascript
// Standard error handler for HTTP requests
function handleHttpError(error, context) {
  const errorInfo = {
    timestamp: new Date().toISOString(),
    context: context,
    message: error.message,
    statusCode: error.statusCode || 'Unknown',
    response: error.response?.body || null
  };

  // Log to console
  console.error('HTTP Error:', JSON.stringify(errorInfo));

  // Determine if retryable
  const retryableCodes = [408, 429, 500, 502, 503, 504];
  errorInfo.retryable = retryableCodes.includes(error.statusCode);

  return errorInfo;
}
```

---

## ΜΕΡΟΣ 5: BEST PRACTICES

### 1. Rate Limiting

```javascript
// Add delay between requests
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Use in loop
for (const item of items) {
  await processItem(item);
  await delay(100); // 100ms between requests
}
```

### 2. Credential Security

- Ποτέ hardcode credentials στο workflow
- Χρήση n8n Credentials για αποθήκευση
- Χρήση environment variables για sensitive data

### 3. Logging Best Practices

```javascript
// Structured logging
const logEntry = {
  timestamp: new Date().toISOString(),
  workflow: 'Order Processing',
  action: 'Update Order Status',
  orderId: order.id,
  oldStatus: order.status,
  newStatus: 'processing',
  success: true,
  duration_ms: endTime - startTime
};
```

### 4. Idempotency

- Χρήση unique identifiers για operations
- Check before create (avoid duplicates)
- Αποθήκευση processed item IDs

### 5. Error Recovery

- Implement retry logic με exponential backoff
- Dead letter queue για failed items
- Alert on repeated failures

### 6. Performance Optimization

- Use batch operations όπου δυνατόν
- Parallel execution για independent operations
- Cache frequently accessed data
- Use webhooks αντί για polling

---

## ΜΕΡΟΣ 6: WEBHOOK SECURITY

### Verify WooCommerce Webhook Signature

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const hash = crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('base64');

  return hash === signature;
}

// In webhook node
const signature = $request.headers['x-wc-webhook-signature'];
const isValid = verifyWebhookSignature(
  JSON.stringify($json),
  signature,
  $env.WEBHOOK_SECRET
);

if (!isValid) {
  throw new Error('Invalid webhook signature');
}
```

---

## ΜΕΡΟΣ 7: EXPRESSIONS CHEATSHEET

### Common n8n Expressions

```javascript
// Current item data
{{$json.fieldName}}

// Previous node data
{{$node["NodeName"].json.fieldName}}

// All items from node
{{$node["NodeName"].all()}}

// Item at index
{{$node["NodeName"].item(0).json.fieldName}}

// Environment variable
{{$env.VARIABLE_NAME}}

// Current timestamp
{{DateTime.now().toISO()}}

// Date math
{{DateTime.now().minus({days: 7}).toISO()}}

// Conditional
{{$json.status === 'active' ? 'Yes' : 'No'}}

// Array length
{{$json.items.length}}

// Sum array
{{$json.items.reduce((a, b) => a + b.price, 0)}}

// Filter array
{{$json.items.filter(i => i.stock > 0)}}

// Map array
{{$json.items.map(i => i.id)}}

// Join array
{{$json.tags.join(', ')}}

// Parse JSON string
{{JSON.parse($json.meta_data)}}

// Stringify object
{{JSON.stringify($json)}}
```

---

## ΜΕΡΟΣ 8: DEBUGGING TIPS

### 1. Use Console Log

```javascript
// In Code node
console.log('Debug:', JSON.stringify($json, null, 2));
```

### 2. Inspect HTTP Responses

Enable "Full Response" in HTTP Request node to see headers and status.

### 3. Test Webhooks Locally

```bash
# Use ngrok for local testing
ngrok http 5678
```

### 4. Validate API Calls

```bash
# Test WooCommerce API directly
curl -X GET "https://store.com/wp-json/wc/v3/products" \
  -u "ck_xxx:cs_xxx" \
  -H "Content-Type: application/json"
```

### 5. Check n8n Execution Log

- Κάθε execution αποθηκεύεται
- Δες input/output κάθε node
- Check για errors στο error output
