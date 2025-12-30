# Code Snippets & Integration Examples

## Overview

Αυτό το αρχείο περιέχει έτοιμα code snippets για n8n Code nodes και integration examples.

---

## SECTION 1: N8N CODE NODE SNIPPETS

### 1.1 Data Transformation

#### Transform Order for External System
```javascript
const orders = $input.all();

return orders.map(order => {
  const o = order.json;
  return {
    json: {
      external_id: `WC-${o.id}`,
      customer: {
        name: `${o.billing.first_name} ${o.billing.last_name}`,
        email: o.billing.email,
        phone: o.billing.phone,
        address: {
          line1: o.billing.address_1,
          line2: o.billing.address_2,
          city: o.billing.city,
          state: o.billing.state,
          postal: o.billing.postcode,
          country: o.billing.country
        }
      },
      items: o.line_items.map(item => ({
        sku: item.sku,
        name: item.name,
        quantity: item.quantity,
        price: parseFloat(item.total) / item.quantity,
        total: parseFloat(item.total)
      })),
      totals: {
        subtotal: parseFloat(o.total) - parseFloat(o.shipping_total) - parseFloat(o.total_tax),
        shipping: parseFloat(o.shipping_total),
        tax: parseFloat(o.total_tax),
        discount: parseFloat(o.discount_total),
        total: parseFloat(o.total)
      },
      status: o.status,
      created_at: o.date_created,
      payment_method: o.payment_method
    }
  };
});
```

#### Flatten Nested Product Data
```javascript
const products = $input.all();

return products.map(p => {
  const prod = p.json;
  return {
    json: {
      id: prod.id,
      sku: prod.sku,
      name: prod.name,
      price: prod.price,
      regular_price: prod.regular_price,
      sale_price: prod.sale_price,
      stock: prod.stock_quantity,
      stock_status: prod.stock_status,
      category: prod.categories?.[0]?.name || 'Uncategorized',
      category_id: prod.categories?.[0]?.id || 0,
      image: prod.images?.[0]?.src || '',
      weight: prod.weight,
      length: prod.dimensions?.length,
      width: prod.dimensions?.width,
      height: prod.dimensions?.height,
      attributes: prod.attributes?.map(a => `${a.name}: ${a.options.join(', ')}`).join(' | ')
    }
  };
});
```

---

### 1.2 Filtering & Sorting

#### Filter Products by Criteria
```javascript
const products = $input.all();
const criteria = {
  minStock: 10,
  maxPrice: 100,
  category: 'electronics',
  onSale: true
};

const filtered = products.filter(p => {
  const prod = p.json;

  if (criteria.minStock && prod.stock_quantity < criteria.minStock) return false;
  if (criteria.maxPrice && parseFloat(prod.price) > criteria.maxPrice) return false;
  if (criteria.category && !prod.categories?.some(c => c.slug === criteria.category)) return false;
  if (criteria.onSale && !prod.on_sale) return false;

  return true;
});

return filtered;
```

#### Sort Orders by Priority
```javascript
const orders = $input.all();

const priorityScore = (order) => {
  let score = 0;
  const o = order.json;

  // High value orders
  if (parseFloat(o.total) > 200) score += 50;
  if (parseFloat(o.total) > 500) score += 50;

  // VIP customers
  if (o.meta_data?.some(m => m.key === '_customer_tier' && m.value === 'vip')) score += 100;

  // Express shipping
  if (o.shipping_lines?.some(s => s.method_id === 'express')) score += 30;

  // Older orders first (FIFO)
  const age = Date.now() - new Date(o.date_created).getTime();
  score += Math.min(age / (1000 * 60 * 60), 24); // Max 24 points for age

  return score;
};

return orders.sort((a, b) => priorityScore(b) - priorityScore(a));
```

---

### 1.3 Aggregation & Statistics

#### Calculate Order Statistics
```javascript
const orders = $input.all();

const stats = {
  totalOrders: orders.length,
  totalRevenue: 0,
  totalItems: 0,
  avgOrderValue: 0,
  statusBreakdown: {},
  topProducts: {},
  revenueByDay: {}
};

orders.forEach(order => {
  const o = order.json;
  const total = parseFloat(o.total);
  const date = o.date_created.split('T')[0];

  stats.totalRevenue += total;
  stats.totalItems += o.line_items.reduce((sum, item) => sum + item.quantity, 0);

  // Status breakdown
  stats.statusBreakdown[o.status] = (stats.statusBreakdown[o.status] || 0) + 1;

  // Revenue by day
  stats.revenueByDay[date] = (stats.revenueByDay[date] || 0) + total;

  // Top products
  o.line_items.forEach(item => {
    stats.topProducts[item.product_id] = stats.topProducts[item.product_id] || {
      name: item.name,
      quantity: 0,
      revenue: 0
    };
    stats.topProducts[item.product_id].quantity += item.quantity;
    stats.topProducts[item.product_id].revenue += parseFloat(item.total);
  });
});

stats.avgOrderValue = stats.totalOrders > 0
  ? (stats.totalRevenue / stats.totalOrders).toFixed(2)
  : 0;

// Sort top products
stats.topProductsList = Object.entries(stats.topProducts)
  .map(([id, data]) => ({ id, ...data }))
  .sort((a, b) => b.revenue - a.revenue)
  .slice(0, 10);

return [{ json: stats }];
```

#### Customer Segmentation Analysis
```javascript
const customers = $input.all();
const now = new Date();

const segments = {
  vip: [],
  regular: [],
  atRisk: [],
  churned: [],
  new: []
};

customers.forEach(c => {
  const customer = c.json;
  const totalSpent = parseFloat(customer.total_spent) || 0;
  const orderCount = customer.orders_count || 0;
  const daysSinceRegistration = customer.date_created
    ? Math.floor((now - new Date(customer.date_created)) / (1000 * 60 * 60 * 24))
    : 999;

  // Get days since last order from meta or calculate
  const lastOrderDate = customer.meta_data?.find(m => m.key === '_last_order_date')?.value;
  const daysSinceOrder = lastOrderDate
    ? Math.floor((now - new Date(lastOrderDate)) / (1000 * 60 * 60 * 24))
    : 999;

  const customerData = {
    id: customer.id,
    email: customer.email,
    name: `${customer.first_name} ${customer.last_name}`,
    totalSpent,
    orderCount,
    daysSinceOrder,
    daysSinceRegistration
  };

  // Segmentation logic
  if (totalSpent >= 1000 || orderCount >= 10) {
    segments.vip.push(customerData);
  } else if (daysSinceOrder > 90 && orderCount > 0) {
    segments.churned.push(customerData);
  } else if (daysSinceOrder > 60 && orderCount > 0) {
    segments.atRisk.push(customerData);
  } else if (daysSinceRegistration < 30 || orderCount <= 1) {
    segments.new.push(customerData);
  } else {
    segments.regular.push(customerData);
  }
});

return [{
  json: {
    summary: {
      vip: segments.vip.length,
      regular: segments.regular.length,
      atRisk: segments.atRisk.length,
      churned: segments.churned.length,
      new: segments.new.length
    },
    segments
  }
}];
```

---

### 1.4 Date & Time Operations

#### Date Range Calculations
```javascript
// Helper functions
const dateHelpers = {
  now: () => new Date(),

  startOfDay: (date = new Date()) => {
    const d = new Date(date);
    d.setHours(0, 0, 0, 0);
    return d;
  },

  endOfDay: (date = new Date()) => {
    const d = new Date(date);
    d.setHours(23, 59, 59, 999);
    return d;
  },

  daysAgo: (days) => {
    const d = new Date();
    d.setDate(d.getDate() - days);
    return d;
  },

  hoursAgo: (hours) => {
    return new Date(Date.now() - hours * 60 * 60 * 1000);
  },

  startOfWeek: () => {
    const d = new Date();
    const day = d.getDay();
    const diff = d.getDate() - day + (day === 0 ? -6 : 1);
    d.setDate(diff);
    d.setHours(0, 0, 0, 0);
    return d;
  },

  startOfMonth: () => {
    const d = new Date();
    d.setDate(1);
    d.setHours(0, 0, 0, 0);
    return d;
  },

  toISO: (date) => date.toISOString(),

  toWooDate: (date) => date.toISOString().split('.')[0]
};

// Use in workflow
const ranges = {
  today: {
    after: dateHelpers.toWooDate(dateHelpers.startOfDay()),
    before: dateHelpers.toWooDate(dateHelpers.endOfDay())
  },
  yesterday: {
    after: dateHelpers.toWooDate(dateHelpers.startOfDay(dateHelpers.daysAgo(1))),
    before: dateHelpers.toWooDate(dateHelpers.endOfDay(dateHelpers.daysAgo(1)))
  },
  thisWeek: {
    after: dateHelpers.toWooDate(dateHelpers.startOfWeek()),
    before: dateHelpers.toWooDate(dateHelpers.now())
  },
  thisMonth: {
    after: dateHelpers.toWooDate(dateHelpers.startOfMonth()),
    before: dateHelpers.toWooDate(dateHelpers.now())
  },
  last30Days: {
    after: dateHelpers.toWooDate(dateHelpers.daysAgo(30)),
    before: dateHelpers.toWooDate(dateHelpers.now())
  }
};

return [{ json: ranges }];
```

---

### 1.5 String Formatting

#### Generate Order Summary Email
```javascript
const order = $input.first().json;

const formatCurrency = (amount) => `€${parseFloat(amount).toFixed(2)}`;

const itemsList = order.line_items.map(item =>
  `• ${item.name} × ${item.quantity} - ${formatCurrency(item.total)}`
).join('\n');

const emailBody = `
Dear ${order.billing.first_name},

Thank you for your order #${order.number}!

ORDER SUMMARY
─────────────────────────────
${itemsList}

─────────────────────────────
Subtotal: ${formatCurrency(parseFloat(order.total) - parseFloat(order.shipping_total) - parseFloat(order.total_tax))}
Shipping: ${formatCurrency(order.shipping_total)}
Tax: ${formatCurrency(order.total_tax)}
TOTAL: ${formatCurrency(order.total)}

SHIPPING ADDRESS
${order.shipping.first_name} ${order.shipping.last_name}
${order.shipping.address_1}
${order.shipping.address_2 ? order.shipping.address_2 + '\n' : ''}${order.shipping.city}, ${order.shipping.postcode}
${order.shipping.country}

${order.customer_note ? `\nYOUR NOTE: ${order.customer_note}\n` : ''}

We'll notify you when your order ships!

Best regards,
Your Store Team
`.trim();

return [{
  json: {
    to: order.billing.email,
    subject: `Order Confirmation #${order.number}`,
    body: emailBody
  }
}];
```

#### Generate Slack Report Block
```javascript
const stats = $input.first().json;

const slackBlocks = [
  {
    type: "header",
    text: {
      type: "plain_text",
      text: `📊 Daily Sales Report - ${new Date().toLocaleDateString()}`
    }
  },
  {
    type: "section",
    fields: [
      {
        type: "mrkdwn",
        text: `*💰 Revenue*\n€${stats.totalRevenue.toFixed(2)}`
      },
      {
        type: "mrkdwn",
        text: `*📦 Orders*\n${stats.totalOrders}`
      },
      {
        type: "mrkdwn",
        text: `*📈 Avg Order*\n€${stats.avgOrderValue}`
      },
      {
        type: "mrkdwn",
        text: `*🛒 Items Sold*\n${stats.totalItems}`
      }
    ]
  },
  {
    type: "divider"
  },
  {
    type: "section",
    text: {
      type: "mrkdwn",
      text: `*🏆 Top Products*\n${stats.topProductsList.slice(0, 5).map((p, i) =>
        `${i + 1}. ${p.name} (${p.quantity} sold - €${p.revenue.toFixed(2)})`
      ).join('\n')}`
    }
  },
  {
    type: "context",
    elements: [
      {
        type: "mrkdwn",
        text: `Generated at ${new Date().toLocaleTimeString()}`
      }
    ]
  }
];

return [{ json: { blocks: slackBlocks } }];
```

---

### 1.6 Validation & Error Handling

#### Validate Order Data
```javascript
const orders = $input.all();
const errors = [];
const valid = [];

orders.forEach((order, index) => {
  const o = order.json;
  const orderErrors = [];

  // Required fields
  if (!o.billing?.email) orderErrors.push('Missing billing email');
  if (!o.billing?.first_name) orderErrors.push('Missing billing first name');
  if (!o.billing?.last_name) orderErrors.push('Missing billing last name');
  if (!o.billing?.address_1) orderErrors.push('Missing billing address');
  if (!o.billing?.city) orderErrors.push('Missing billing city');
  if (!o.billing?.postcode) orderErrors.push('Missing billing postcode');
  if (!o.billing?.country) orderErrors.push('Missing billing country');

  // Email format
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (o.billing?.email && !emailRegex.test(o.billing.email)) {
    orderErrors.push('Invalid email format');
  }

  // Phone format (basic)
  if (o.billing?.phone && o.billing.phone.length < 8) {
    orderErrors.push('Phone number too short');
  }

  // Line items
  if (!o.line_items || o.line_items.length === 0) {
    orderErrors.push('No line items');
  }

  // Total validation
  if (parseFloat(o.total) <= 0) {
    orderErrors.push('Invalid order total');
  }

  if (orderErrors.length > 0) {
    errors.push({
      orderId: o.id,
      orderNumber: o.number,
      errors: orderErrors
    });
  } else {
    valid.push(order);
  }
});

// Output both valid orders and error report
return {
  valid: valid,
  errors: errors.length > 0 ? [{ json: { errors, count: errors.length } }] : []
};
```

#### Safe Property Access
```javascript
// Helper for safe property access
const get = (obj, path, defaultValue = null) => {
  const keys = path.split('.');
  let result = obj;

  for (const key of keys) {
    if (result === null || result === undefined) {
      return defaultValue;
    }
    result = result[key];
  }

  return result ?? defaultValue;
};

// Usage
const order = $input.first().json;

return [{
  json: {
    id: get(order, 'id'),
    customerName: `${get(order, 'billing.first_name', '')} ${get(order, 'billing.last_name', '')}`.trim() || 'Unknown',
    email: get(order, 'billing.email', 'no-email@example.com'),
    total: parseFloat(get(order, 'total', '0')),
    firstProductName: get(order, 'line_items.0.name', 'No products'),
    vipStatus: get(order, 'meta_data', []).find(m => m.key === '_vip')?.value || 'standard'
  }
}];
```

---

## SECTION 2: API REQUEST EXAMPLES

### 2.1 WooCommerce Product Operations

#### Create Variable Product with Variations
```javascript
// Step 1: Create parent product
const parentProduct = {
  name: "T-Shirt",
  type: "variable",
  description: "Premium cotton t-shirt",
  short_description: "Comfortable everyday t-shirt",
  categories: [{ id: 15 }],
  images: [
    { src: "https://example.com/tshirt.jpg" }
  ],
  attributes: [
    {
      id: 1, // Color attribute ID
      name: "Color",
      visible: true,
      variation: true,
      options: ["Red", "Blue", "Black"]
    },
    {
      id: 2, // Size attribute ID
      name: "Size",
      visible: true,
      variation: true,
      options: ["S", "M", "L", "XL"]
    }
  ]
};

// HTTP Request to create parent
// POST /wp-json/wc/v3/products
// Body: parentProduct

// Step 2: Create variations
const variations = [
  {
    regular_price: "29.99",
    attributes: [
      { id: 1, option: "Red" },
      { id: 2, option: "S" }
    ],
    stock_quantity: 10,
    sku: "TSHIRT-RED-S"
  },
  {
    regular_price: "29.99",
    attributes: [
      { id: 1, option: "Red" },
      { id: 2, option: "M" }
    ],
    stock_quantity: 15,
    sku: "TSHIRT-RED-M"
  }
  // ... more variations
];

// HTTP Request for each variation
// POST /wp-json/wc/v3/products/{parent_id}/variations
// Body: variation

return [{ json: { parentProduct, variations } }];
```

#### Bulk Price Update
```javascript
const priceUpdates = $input.all();

// Build batch request
const batchBody = {
  update: priceUpdates.map(p => ({
    id: p.json.id,
    regular_price: p.json.newPrice.toString(),
    sale_price: p.json.newSalePrice ? p.json.newSalePrice.toString() : ""
  }))
};

// HTTP Request
// POST /wp-json/wc/v3/products/batch
// Body: batchBody

return [{ json: batchBody }];
```

---

### 2.2 WooCommerce Order Operations

#### Create Order Programmatically
```javascript
const orderData = {
  payment_method: "bacs",
  payment_method_title: "Bank Transfer",
  set_paid: false,
  billing: {
    first_name: "John",
    last_name: "Doe",
    address_1: "123 Main St",
    city: "Athens",
    state: "AT",
    postcode: "10431",
    country: "GR",
    email: "john@example.com",
    phone: "2101234567"
  },
  shipping: {
    first_name: "John",
    last_name: "Doe",
    address_1: "123 Main St",
    city: "Athens",
    state: "AT",
    postcode: "10431",
    country: "GR"
  },
  line_items: [
    {
      product_id: 123,
      quantity: 2
    },
    {
      product_id: 456,
      variation_id: 789,
      quantity: 1
    }
  ],
  shipping_lines: [
    {
      method_id: "flat_rate",
      method_title: "Flat Rate",
      total: "5.00"
    }
  ],
  coupon_lines: [
    {
      code: "DISCOUNT10"
    }
  ],
  meta_data: [
    {
      key: "_source",
      value: "n8n_automation"
    }
  ]
};

// HTTP Request
// POST /wp-json/wc/v3/orders
// Body: orderData

return [{ json: orderData }];
```

#### Process Refund
```javascript
const order = $input.first().json;

const refundData = {
  amount: order.refundAmount || order.total,
  reason: order.refundReason || "Customer request",
  refunded_by: 1, // Admin user ID
  meta_data: [
    {
      key: "_refund_processed_by",
      value: "n8n_automation"
    }
  ],
  line_items: order.refundItems?.map(item => ({
    id: item.lineItemId,
    quantity: item.quantity,
    refund_total: item.total
  })) || []
};

// HTTP Request
// POST /wp-json/wc/v3/orders/{order.id}/refunds
// Body: refundData

return [{ json: refundData }];
```

---

### 2.3 WooCommerce Customer Operations

#### Update Customer with Calculated Fields
```javascript
const customer = $input.first().json;
const orders = $input.item(1)?.json || [];

// Calculate customer metrics
const metrics = {
  totalSpent: orders.reduce((sum, o) => sum + parseFloat(o.total), 0),
  orderCount: orders.length,
  avgOrderValue: orders.length > 0
    ? orders.reduce((sum, o) => sum + parseFloat(o.total), 0) / orders.length
    : 0,
  lastOrderDate: orders.length > 0
    ? orders.sort((a, b) => new Date(b.date_created) - new Date(a.date_created))[0].date_created
    : null,
  favoriteCategory: calculateFavoriteCategory(orders)
};

function calculateFavoriteCategory(orders) {
  const categories = {};
  orders.forEach(o => {
    o.line_items?.forEach(item => {
      item.categories?.forEach(cat => {
        categories[cat.name] = (categories[cat.name] || 0) + item.quantity;
      });
    });
  });
  const sorted = Object.entries(categories).sort((a, b) => b[1] - a[1]);
  return sorted[0]?.[0] || 'Unknown';
}

// Determine customer tier
let tier = 'bronze';
if (metrics.totalSpent >= 1000) tier = 'gold';
else if (metrics.totalSpent >= 500) tier = 'silver';

const updateData = {
  meta_data: [
    { key: '_total_spent_calculated', value: metrics.totalSpent.toFixed(2) },
    { key: '_order_count', value: metrics.orderCount.toString() },
    { key: '_avg_order_value', value: metrics.avgOrderValue.toFixed(2) },
    { key: '_last_order_date', value: metrics.lastOrderDate },
    { key: '_favorite_category', value: metrics.favoriteCategory },
    { key: '_customer_tier', value: tier },
    { key: '_metrics_updated', value: new Date().toISOString() }
  ]
};

// HTTP Request
// PUT /wp-json/wc/v3/customers/{customer.id}
// Body: updateData

return [{ json: { customerId: customer.id, updateData, metrics } }];
```

---

### 2.4 WordPress Content Operations

#### Create Blog Post with Featured Image
```javascript
// Step 1: Upload image
const imageUpload = {
  // This would be done with binary data
  // POST /wp-json/wp/v2/media
  // Content-Type: image/jpeg
  // Content-Disposition: attachment; filename="image.jpg"
};

// Step 2: Create post
const postData = {
  title: "New Product Announcement",
  content: `
    <p>We're excited to announce our latest product!</p>
    <h2>Features</h2>
    <ul>
      <li>Feature 1</li>
      <li>Feature 2</li>
      <li>Feature 3</li>
    </ul>
    <p>Available now in our store.</p>
  `,
  excerpt: "Check out our exciting new product with amazing features.",
  status: "publish", // or "draft"
  categories: [1, 5], // Category IDs
  tags: [10, 15], // Tag IDs
  featured_media: 123, // Media ID from step 1
  meta: {
    _yoast_wpseo_title: "New Product Announcement | Your Store",
    _yoast_wpseo_metadesc: "Discover our latest product with amazing features. Shop now!"
  }
};

// HTTP Request
// POST /wp-json/wp/v2/posts
// Body: postData

return [{ json: postData }];
```

---

## SECTION 3: INTEGRATION PATTERNS

### 3.1 Google Sheets Sync

#### Export Orders to Sheets
```javascript
const orders = $input.all();

// Format for Google Sheets
const rows = orders.map(o => {
  const order = o.json;
  return {
    order_id: order.id,
    order_number: order.number,
    date: order.date_created,
    status: order.status,
    customer_name: `${order.billing.first_name} ${order.billing.last_name}`,
    customer_email: order.billing.email,
    customer_phone: order.billing.phone,
    shipping_city: order.shipping.city,
    shipping_country: order.shipping.country,
    items_count: order.line_items.reduce((sum, i) => sum + i.quantity, 0),
    subtotal: parseFloat(order.total) - parseFloat(order.shipping_total) - parseFloat(order.total_tax),
    shipping: parseFloat(order.shipping_total),
    tax: parseFloat(order.total_tax),
    discount: parseFloat(order.discount_total),
    total: parseFloat(order.total),
    payment_method: order.payment_method_title,
    items_list: order.line_items.map(i => `${i.name} x${i.quantity}`).join('; ')
  };
});

return rows.map(row => ({ json: row }));
```

---

### 3.2 Slack Notifications

#### Order Alert with Actions
```javascript
const order = $input.first().json;

const slackMessage = {
  text: `New Order #${order.number}`,
  blocks: [
    {
      type: "section",
      text: {
        type: "mrkdwn",
        text: `*New Order #${order.number}*\n` +
              `Customer: ${order.billing.first_name} ${order.billing.last_name}\n` +
              `Total: €${order.total}`
      },
      accessory: {
        type: "button",
        text: {
          type: "plain_text",
          text: "View Order"
        },
        url: `https://yourstore.com/wp-admin/post.php?post=${order.id}&action=edit`
      }
    },
    {
      type: "section",
      fields: [
        {
          type: "mrkdwn",
          text: `*Items:*\n${order.line_items.map(i => `• ${i.name} ×${i.quantity}`).join('\n')}`
        },
        {
          type: "mrkdwn",
          text: `*Shipping:*\n${order.shipping.city}, ${order.shipping.country}`
        }
      ]
    },
    {
      type: "actions",
      elements: [
        {
          type: "button",
          text: { type: "plain_text", text: "Process" },
          style: "primary",
          action_id: `process_${order.id}`
        },
        {
          type: "button",
          text: { type: "plain_text", text: "Hold" },
          action_id: `hold_${order.id}`
        }
      ]
    }
  ]
};

return [{ json: slackMessage }];
```

---

### 3.3 Email Templates

#### HTML Order Confirmation Email
```javascript
const order = $input.first().json;

const htmlEmail = `
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
    .container { max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: #4a90d9; color: white; padding: 20px; text-align: center; }
    .content { padding: 20px; background: #f9f9f9; }
    .order-table { width: 100%; border-collapse: collapse; margin: 20px 0; }
    .order-table th, .order-table td { padding: 10px; border-bottom: 1px solid #ddd; text-align: left; }
    .order-table th { background: #f0f0f0; }
    .totals { text-align: right; }
    .footer { text-align: center; padding: 20px; font-size: 12px; color: #666; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>Order Confirmation</h1>
      <p>Order #${order.number}</p>
    </div>

    <div class="content">
      <p>Hi ${order.billing.first_name},</p>
      <p>Thank you for your order! Here's a summary:</p>

      <table class="order-table">
        <thead>
          <tr>
            <th>Product</th>
            <th>Qty</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>
          ${order.line_items.map(item => `
            <tr>
              <td>${item.name}</td>
              <td>${item.quantity}</td>
              <td>€${parseFloat(item.total).toFixed(2)}</td>
            </tr>
          `).join('')}
        </tbody>
      </table>

      <div class="totals">
        <p><strong>Subtotal:</strong> €${(parseFloat(order.total) - parseFloat(order.shipping_total) - parseFloat(order.total_tax)).toFixed(2)}</p>
        <p><strong>Shipping:</strong> €${parseFloat(order.shipping_total).toFixed(2)}</p>
        <p><strong>Tax:</strong> €${parseFloat(order.total_tax).toFixed(2)}</p>
        <p style="font-size: 18px;"><strong>Total:</strong> €${parseFloat(order.total).toFixed(2)}</p>
      </div>

      <h3>Shipping Address</h3>
      <p>
        ${order.shipping.first_name} ${order.shipping.last_name}<br>
        ${order.shipping.address_1}<br>
        ${order.shipping.address_2 ? order.shipping.address_2 + '<br>' : ''}
        ${order.shipping.city}, ${order.shipping.postcode}<br>
        ${order.shipping.country}
      </p>
    </div>

    <div class="footer">
      <p>Questions? Contact us at support@yourstore.com</p>
      <p>© ${new Date().getFullYear()} Your Store. All rights reserved.</p>
    </div>
  </div>
</body>
</html>
`;

return [{
  json: {
    to: order.billing.email,
    subject: `Order Confirmation #${order.number}`,
    html: htmlEmail
  }
}];
```

---

## SECTION 4: UTILITY CODE

### 4.1 Common Helpers Module

```javascript
// This can be stored as a Code node and referenced
const helpers = {
  // Currency formatting
  formatCurrency: (amount, currency = 'EUR') => {
    return new Intl.NumberFormat('el-GR', {
      style: 'currency',
      currency: currency
    }).format(amount);
  },

  // Date formatting
  formatDate: (dateString, locale = 'el-GR') => {
    return new Date(dateString).toLocaleDateString(locale, {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  },

  // Slugify
  slugify: (text) => {
    return text
      .toLowerCase()
      .replace(/[^\w\s-]/g, '')
      .replace(/[\s_-]+/g, '-')
      .replace(/^-+|-+$/g, '');
  },

  // Generate unique ID
  generateId: (prefix = '') => {
    return `${prefix}${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  },

  // Deep clone
  deepClone: (obj) => JSON.parse(JSON.stringify(obj)),

  // Chunk array
  chunk: (array, size) => {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  },

  // Remove duplicates
  unique: (array, key) => {
    if (key) {
      const seen = new Set();
      return array.filter(item => {
        const k = item[key];
        return seen.has(k) ? false : seen.add(k);
      });
    }
    return [...new Set(array)];
  },

  // Sort by key
  sortBy: (array, key, order = 'asc') => {
    return [...array].sort((a, b) => {
      if (order === 'asc') return a[key] > b[key] ? 1 : -1;
      return a[key] < b[key] ? 1 : -1;
    });
  },

  // Group by key
  groupBy: (array, key) => {
    return array.reduce((groups, item) => {
      const group = item[key];
      groups[group] = groups[group] || [];
      groups[group].push(item);
      return groups;
    }, {});
  },

  // Calculate percentage
  percentage: (value, total, decimals = 2) => {
    if (total === 0) return 0;
    return ((value / total) * 100).toFixed(decimals);
  },

  // Retry with exponential backoff
  sleep: (ms) => new Promise(resolve => setTimeout(resolve, ms)),

  // Parse WooCommerce meta
  getMeta: (metaArray, key, defaultValue = null) => {
    const meta = metaArray?.find(m => m.key === key);
    return meta?.value ?? defaultValue;
  },

  // Set WooCommerce meta
  setMeta: (metaArray, key, value) => {
    const index = metaArray?.findIndex(m => m.key === key);
    if (index > -1) {
      metaArray[index].value = value;
    } else {
      metaArray?.push({ key, value });
    }
    return metaArray;
  }
};

// Export for use
return [{ json: { helpers: 'available' } }];
```

---

This completes the code snippets and integration examples documentation.
