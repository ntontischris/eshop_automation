# WooCommerce REST API v3 - Complete Reference

## Overview

Base URL: `https://yourstore.com/wp-json/wc/v3/`

Authentication: Basic Auth με Consumer Key & Consumer Secret

---

## 1. PRODUCTS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products` | List all products |
| GET | `/products/{id}` | Get single product |
| POST | `/products` | Create product |
| PUT | `/products/{id}` | Update product |
| DELETE | `/products/{id}` | Delete product |
| POST | `/products/batch` | Batch create/update/delete |

### Product Properties (Complete List)

```json
{
  "id": 123,
  "name": "Product Name",
  "slug": "product-name",
  "permalink": "https://store.com/product/product-name",
  "date_created": "2024-01-15T10:00:00",
  "date_modified": "2024-01-20T15:30:00",
  "type": "simple|grouped|external|variable",
  "status": "draft|pending|private|publish",
  "featured": false,
  "catalog_visibility": "visible|catalog|search|hidden",
  "description": "Full HTML description",
  "short_description": "Brief description",
  "sku": "SKU-001",
  "price": "29.99",
  "regular_price": "39.99",
  "sale_price": "29.99",
  "date_on_sale_from": "2024-01-01T00:00:00",
  "date_on_sale_to": "2024-01-31T23:59:59",
  "on_sale": true,
  "purchasable": true,
  "total_sales": 150,
  "virtual": false,
  "downloadable": false,
  "downloads": [],
  "download_limit": -1,
  "download_expiry": -1,
  "external_url": "",
  "button_text": "",
  "tax_status": "taxable|shipping|none",
  "tax_class": "",
  "manage_stock": true,
  "stock_quantity": 100,
  "stock_status": "instock|outofstock|onbackorder",
  "backorders": "no|notify|yes",
  "backorders_allowed": false,
  "backordered": false,
  "low_stock_amount": 10,
  "sold_individually": false,
  "weight": "0.5",
  "dimensions": {
    "length": "10",
    "width": "5",
    "height": "2"
  },
  "shipping_required": true,
  "shipping_taxable": true,
  "shipping_class": "",
  "shipping_class_id": 0,
  "reviews_allowed": true,
  "average_rating": "4.50",
  "rating_count": 25,
  "upsell_ids": [456, 789],
  "cross_sell_ids": [321, 654],
  "parent_id": 0,
  "purchase_note": "Thank you for your purchase!",
  "categories": [
    {"id": 15, "name": "Category Name", "slug": "category-name"}
  ],
  "tags": [
    {"id": 10, "name": "Tag Name", "slug": "tag-name"}
  ],
  "images": [
    {
      "id": 100,
      "src": "https://store.com/image.jpg",
      "name": "image-name",
      "alt": "Alt text"
    }
  ],
  "attributes": [
    {
      "id": 1,
      "name": "Color",
      "position": 0,
      "visible": true,
      "variation": true,
      "options": ["Red", "Blue", "Green"]
    }
  ],
  "default_attributes": [
    {"id": 1, "name": "Color", "option": "Red"}
  ],
  "variations": [111, 112, 113],
  "grouped_products": [],
  "menu_order": 0,
  "related_ids": [200, 201, 202],
  "meta_data": [
    {"id": 1, "key": "_custom_field", "value": "custom_value"}
  ]
}
```

### Query Parameters for GET /products

| Parameter | Type | Description |
|-----------|------|-------------|
| `context` | string | Scope: view, edit |
| `page` | integer | Current page (default: 1) |
| `per_page` | integer | Items per page (default: 10, max: 100) |
| `search` | string | Search term |
| `after` | string | Limit to after date (ISO8601) |
| `before` | string | Limit to before date (ISO8601) |
| `exclude` | array | Exclude specific IDs |
| `include` | array | Include specific IDs |
| `offset` | integer | Offset results |
| `order` | string | asc or desc |
| `orderby` | string | date, id, include, title, slug, price, popularity, rating |
| `parent` | array | Filter by parent ID |
| `parent_exclude` | array | Exclude by parent ID |
| `slug` | string | Filter by slug |
| `status` | string | Filter by status |
| `type` | string | Filter by type |
| `sku` | string | Filter by SKU |
| `featured` | boolean | Filter featured products |
| `category` | string | Filter by category ID |
| `tag` | string | Filter by tag ID |
| `shipping_class` | string | Filter by shipping class |
| `attribute` | string | Filter by attribute |
| `attribute_term` | string | Filter by attribute term |
| `tax_class` | string | Filter by tax class |
| `on_sale` | boolean | Filter on sale products |
| `min_price` | string | Filter by minimum price |
| `max_price` | string | Filter by maximum price |
| `stock_status` | string | Filter by stock status |

### Batch Operations

```json
POST /wp-json/wc/v3/products/batch

{
  "create": [
    {
      "name": "New Product 1",
      "type": "simple",
      "regular_price": "29.99",
      "description": "Description here",
      "categories": [{"id": 15}]
    }
  ],
  "update": [
    {
      "id": 123,
      "regular_price": "39.99",
      "stock_quantity": 50
    }
  ],
  "delete": [456, 789]
}
```

---

## 2. PRODUCT VARIATIONS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/{product_id}/variations` | List variations |
| GET | `/products/{product_id}/variations/{id}` | Get variation |
| POST | `/products/{product_id}/variations` | Create variation |
| PUT | `/products/{product_id}/variations/{id}` | Update variation |
| DELETE | `/products/{product_id}/variations/{id}` | Delete variation |
| POST | `/products/{product_id}/variations/batch` | Batch operations |

### Variation Properties

```json
{
  "id": 111,
  "date_created": "2024-01-15T10:00:00",
  "date_modified": "2024-01-20T15:30:00",
  "description": "Variation description",
  "permalink": "https://store.com/product/?attribute_color=red",
  "sku": "SKU-001-RED",
  "price": "29.99",
  "regular_price": "39.99",
  "sale_price": "29.99",
  "date_on_sale_from": null,
  "date_on_sale_to": null,
  "on_sale": true,
  "status": "publish",
  "purchasable": true,
  "virtual": false,
  "downloadable": false,
  "downloads": [],
  "download_limit": -1,
  "download_expiry": -1,
  "tax_status": "taxable",
  "tax_class": "",
  "manage_stock": true,
  "stock_quantity": 25,
  "stock_status": "instock",
  "backorders": "no",
  "backorders_allowed": false,
  "backordered": false,
  "low_stock_amount": 5,
  "weight": "0.5",
  "dimensions": {
    "length": "10",
    "width": "5",
    "height": "2"
  },
  "shipping_class": "",
  "shipping_class_id": 0,
  "image": {
    "id": 100,
    "src": "https://store.com/red-image.jpg",
    "name": "red-variation",
    "alt": "Red variation"
  },
  "attributes": [
    {
      "id": 1,
      "name": "Color",
      "option": "Red"
    }
  ],
  "menu_order": 0,
  "meta_data": []
}
```

---

## 3. PRODUCT CATEGORIES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/categories` | List categories |
| GET | `/products/categories/{id}` | Get category |
| POST | `/products/categories` | Create category |
| PUT | `/products/categories/{id}` | Update category |
| DELETE | `/products/categories/{id}` | Delete category |
| POST | `/products/categories/batch` | Batch operations |

### Category Properties

```json
{
  "id": 15,
  "name": "Category Name",
  "slug": "category-name",
  "parent": 0,
  "description": "Category description",
  "display": "default|products|subcategories|both",
  "image": {
    "id": 50,
    "src": "https://store.com/category-image.jpg",
    "name": "category-image",
    "alt": "Category Image"
  },
  "menu_order": 0,
  "count": 25
}
```

---

## 4. PRODUCT ATTRIBUTES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/attributes` | List attributes |
| GET | `/products/attributes/{id}` | Get attribute |
| POST | `/products/attributes` | Create attribute |
| PUT | `/products/attributes/{id}` | Update attribute |
| DELETE | `/products/attributes/{id}` | Delete attribute |
| POST | `/products/attributes/batch` | Batch operations |

### Attribute Properties

```json
{
  "id": 1,
  "name": "Color",
  "slug": "pa_color",
  "type": "select",
  "order_by": "menu_order|name|name_num|id",
  "has_archives": true
}
```

### Attribute Terms Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/attributes/{attr_id}/terms` | List terms |
| GET | `/products/attributes/{attr_id}/terms/{id}` | Get term |
| POST | `/products/attributes/{attr_id}/terms` | Create term |
| PUT | `/products/attributes/{attr_id}/terms/{id}` | Update term |
| DELETE | `/products/attributes/{attr_id}/terms/{id}` | Delete term |

---

## 5. PRODUCT TAGS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/tags` | List tags |
| GET | `/products/tags/{id}` | Get tag |
| POST | `/products/tags` | Create tag |
| PUT | `/products/tags/{id}` | Update tag |
| DELETE | `/products/tags/{id}` | Delete tag |
| POST | `/products/tags/batch` | Batch operations |

### Tag Properties

```json
{
  "id": 10,
  "name": "Tag Name",
  "slug": "tag-name",
  "description": "Tag description",
  "count": 15
}
```

---

## 6. PRODUCT REVIEWS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/reviews` | List reviews |
| GET | `/products/reviews/{id}` | Get review |
| POST | `/products/reviews` | Create review |
| PUT | `/products/reviews/{id}` | Update review |
| DELETE | `/products/reviews/{id}` | Delete review |
| POST | `/products/reviews/batch` | Batch operations |

### Review Properties

```json
{
  "id": 50,
  "date_created": "2024-01-15T10:00:00",
  "product_id": 123,
  "status": "approved|hold|spam|unspam|trash|untrash",
  "reviewer": "John Doe",
  "reviewer_email": "john@example.com",
  "review": "Great product!",
  "rating": 5,
  "verified": true,
  "reviewer_avatar_urls": {}
}
```

---

## 7. ORDERS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/orders` | List orders |
| GET | `/orders/{id}` | Get order |
| POST | `/orders` | Create order |
| PUT | `/orders/{id}` | Update order |
| DELETE | `/orders/{id}` | Delete order |
| POST | `/orders/batch` | Batch operations |

### Order Properties (Complete)

```json
{
  "id": 1000,
  "parent_id": 0,
  "status": "pending|processing|on-hold|completed|cancelled|refunded|failed|trash",
  "currency": "EUR",
  "version": "8.0.0",
  "prices_include_tax": false,
  "date_created": "2024-01-15T10:00:00",
  "date_modified": "2024-01-15T12:00:00",
  "discount_total": "10.00",
  "discount_tax": "2.40",
  "shipping_total": "5.00",
  "shipping_tax": "1.20",
  "cart_tax": "5.76",
  "total": "50.00",
  "total_tax": "9.36",
  "customer_id": 5,
  "order_key": "wc_order_xxxxx",
  "billing": {
    "first_name": "John",
    "last_name": "Doe",
    "company": "Company Inc",
    "address_1": "123 Main St",
    "address_2": "Apt 4",
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
    "address_2": "Apt 4",
    "city": "Athens",
    "state": "AT",
    "postcode": "10431",
    "country": "GR",
    "phone": "+30 210 1234567"
  },
  "payment_method": "stripe",
  "payment_method_title": "Credit Card (Stripe)",
  "transaction_id": "ch_xxxxx",
  "customer_ip_address": "192.168.1.1",
  "customer_user_agent": "Mozilla/5.0...",
  "created_via": "checkout|admin|rest-api",
  "customer_note": "Please gift wrap",
  "date_completed": "2024-01-16T10:00:00",
  "date_paid": "2024-01-15T10:05:00",
  "cart_hash": "xxxxx",
  "number": "1000",
  "line_items": [
    {
      "id": 1,
      "name": "Product Name",
      "product_id": 123,
      "variation_id": 0,
      "quantity": 2,
      "tax_class": "",
      "subtotal": "59.98",
      "subtotal_tax": "14.40",
      "total": "49.98",
      "total_tax": "12.00",
      "taxes": [
        {"id": 1, "total": "12.00", "subtotal": "14.40"}
      ],
      "meta_data": [],
      "sku": "SKU-001",
      "price": 24.99,
      "image": {"id": 100, "src": "https://..."},
      "parent_name": null
    }
  ],
  "tax_lines": [
    {
      "id": 1,
      "rate_code": "GR-VAT-1",
      "rate_id": 1,
      "label": "VAT",
      "compound": false,
      "tax_total": "9.36",
      "shipping_tax_total": "1.20",
      "rate_percent": 24,
      "meta_data": []
    }
  ],
  "shipping_lines": [
    {
      "id": 1,
      "method_title": "Flat rate",
      "method_id": "flat_rate",
      "instance_id": "1",
      "total": "5.00",
      "total_tax": "1.20",
      "taxes": [],
      "meta_data": []
    }
  ],
  "fee_lines": [
    {
      "id": 1,
      "name": "Processing Fee",
      "tax_class": "",
      "tax_status": "taxable",
      "amount": "2.00",
      "total": "2.00",
      "total_tax": "0.48",
      "taxes": [],
      "meta_data": []
    }
  ],
  "coupon_lines": [
    {
      "id": 1,
      "code": "DISCOUNT10",
      "discount": "10.00",
      "discount_tax": "2.40",
      "meta_data": []
    }
  ],
  "refunds": [
    {
      "id": 500,
      "reason": "Customer request",
      "total": "-25.00"
    }
  ],
  "payment_url": "https://store.com/checkout/order-pay/1000",
  "is_editable": false,
  "needs_payment": false,
  "needs_processing": true,
  "date_created_gmt": "2024-01-15T08:00:00",
  "date_modified_gmt": "2024-01-15T10:00:00",
  "date_completed_gmt": null,
  "date_paid_gmt": "2024-01-15T08:05:00",
  "currency_symbol": "€",
  "meta_data": [
    {"id": 1, "key": "_custom_field", "value": "value"}
  ]
}
```

### Order Status Values

| Status | Description |
|--------|-------------|
| `pending` | Awaiting payment |
| `processing` | Payment received, awaiting fulfillment |
| `on-hold` | Awaiting payment confirmation |
| `completed` | Order fulfilled |
| `cancelled` | Cancelled by admin or customer |
| `refunded` | Fully refunded |
| `failed` | Payment failed |
| `trash` | Trashed |

### Query Parameters for GET /orders

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
| `orderby` | string | date, id, include, title, slug |
| `parent` | array | Parent IDs |
| `parent_exclude` | array | Exclude parent IDs |
| `status` | string/array | Filter by status |
| `customer` | integer | Filter by customer ID |
| `product` | integer | Filter by product ID |
| `dp` | integer | Decimal points |

---

## 8. ORDER NOTES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/orders/{order_id}/notes` | List notes |
| GET | `/orders/{order_id}/notes/{id}` | Get note |
| POST | `/orders/{order_id}/notes` | Create note |
| DELETE | `/orders/{order_id}/notes/{id}` | Delete note |

### Note Properties

```json
{
  "id": 1,
  "author": "admin",
  "date_created": "2024-01-15T10:00:00",
  "note": "Order shipped via DHL",
  "customer_note": false
}
```

---

## 9. ORDER REFUNDS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/orders/{order_id}/refunds` | List refunds |
| GET | `/orders/{order_id}/refunds/{id}` | Get refund |
| POST | `/orders/{order_id}/refunds` | Create refund |
| DELETE | `/orders/{order_id}/refunds/{id}` | Delete refund |

### Refund Properties

```json
{
  "id": 500,
  "date_created": "2024-01-20T10:00:00",
  "amount": "25.00",
  "reason": "Product defective",
  "refunded_by": 1,
  "refunded_payment": true,
  "meta_data": [],
  "line_items": [
    {
      "id": 1,
      "name": "Product Name",
      "product_id": 123,
      "quantity": -1,
      "refund_total": -25.00
    }
  ]
}
```

---

## 10. CUSTOMERS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/customers` | List customers |
| GET | `/customers/{id}` | Get customer |
| POST | `/customers` | Create customer |
| PUT | `/customers/{id}` | Update customer |
| DELETE | `/customers/{id}` | Delete customer |
| POST | `/customers/batch` | Batch operations |

### Customer Properties

```json
{
  "id": 5,
  "date_created": "2024-01-01T10:00:00",
  "date_modified": "2024-01-15T12:00:00",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "role": "customer",
  "username": "johndoe",
  "password": "password123",
  "billing": {
    "first_name": "John",
    "last_name": "Doe",
    "company": "Company Inc",
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
    "country": "GR",
    "phone": "+30 210 1234567"
  },
  "is_paying_customer": true,
  "avatar_url": "https://...",
  "meta_data": [],
  "orders_count": 10,
  "total_spent": "500.00"
}
```

---

## 11. COUPONS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/coupons` | List coupons |
| GET | `/coupons/{id}` | Get coupon |
| POST | `/coupons` | Create coupon |
| PUT | `/coupons/{id}` | Update coupon |
| DELETE | `/coupons/{id}` | Delete coupon |
| POST | `/coupons/batch` | Batch operations |

### Coupon Properties

```json
{
  "id": 100,
  "code": "SUMMER20",
  "amount": "20.00",
  "date_created": "2024-01-01T00:00:00",
  "date_modified": "2024-01-15T10:00:00",
  "discount_type": "percent|fixed_cart|fixed_product",
  "description": "Summer sale 20% off",
  "date_expires": "2024-08-31T23:59:59",
  "usage_count": 150,
  "individual_use": true,
  "product_ids": [123, 456],
  "excluded_product_ids": [789],
  "usage_limit": 1000,
  "usage_limit_per_user": 1,
  "limit_usage_to_x_items": null,
  "free_shipping": false,
  "product_categories": [15, 16],
  "excluded_product_categories": [20],
  "exclude_sale_items": true,
  "minimum_amount": "50.00",
  "maximum_amount": "500.00",
  "email_restrictions": ["john@example.com"],
  "used_by": ["john@example.com"],
  "meta_data": []
}
```

### Discount Types

| Type | Description |
|------|-------------|
| `percent` | Percentage discount |
| `fixed_cart` | Fixed amount off cart |
| `fixed_product` | Fixed amount off per product |

---

## 12. REPORTS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/reports` | List available reports |
| GET | `/reports/sales` | Sales report |
| GET | `/reports/top_sellers` | Top sellers |
| GET | `/reports/coupons/totals` | Coupon usage totals |
| GET | `/reports/customers/totals` | Customer totals |
| GET | `/reports/orders/totals` | Order totals |
| GET | `/reports/products/totals` | Product totals |
| GET | `/reports/reviews/totals` | Review totals |

### Sales Report Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `context` | string | view |
| `period` | string | week, month, last_month, year |
| `date_min` | string | Start date (YYYY-MM-DD) |
| `date_max` | string | End date (YYYY-MM-DD) |

### Sales Report Response

```json
{
  "total_sales": "15000.00",
  "net_sales": "12500.00",
  "average_sales": "500.00",
  "total_orders": 150,
  "total_items": 450,
  "total_tax": "3000.00",
  "total_shipping": "750.00",
  "total_refunds": 5,
  "total_discount": "500.00",
  "totals_grouped_by": "day",
  "totals": {
    "2024-01-15": {
      "sales": "1500.00",
      "orders": 15,
      "items": 45,
      "tax": "300.00",
      "shipping": "75.00",
      "discount": "50.00",
      "customers": 12
    }
  }
}
```

### Top Sellers Response

```json
[
  {
    "title": "Best Selling Product",
    "product_id": 123,
    "quantity": 500
  }
]
```

---

## 13. TAX API

### Tax Rates Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/taxes` | List tax rates |
| GET | `/taxes/{id}` | Get tax rate |
| POST | `/taxes` | Create tax rate |
| PUT | `/taxes/{id}` | Update tax rate |
| DELETE | `/taxes/{id}` | Delete tax rate |
| POST | `/taxes/batch` | Batch operations |

### Tax Classes Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/taxes/classes` | List tax classes |
| POST | `/taxes/classes` | Create tax class |
| DELETE | `/taxes/classes/{slug}` | Delete tax class |

### Tax Rate Properties

```json
{
  "id": 1,
  "country": "GR",
  "state": "",
  "postcode": "",
  "city": "",
  "postcodes": [],
  "cities": [],
  "rate": "24.0000",
  "name": "VAT",
  "priority": 1,
  "compound": false,
  "shipping": true,
  "order": 0,
  "class": "standard"
}
```

---

## 14. SHIPPING API

### Shipping Zones Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/shipping/zones` | List zones |
| GET | `/shipping/zones/{id}` | Get zone |
| POST | `/shipping/zones` | Create zone |
| PUT | `/shipping/zones/{id}` | Update zone |
| DELETE | `/shipping/zones/{id}` | Delete zone |

### Shipping Zone Locations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/shipping/zones/{id}/locations` | List locations |
| PUT | `/shipping/zones/{id}/locations` | Update locations |

### Shipping Zone Methods

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/shipping/zones/{id}/methods` | List methods |
| GET | `/shipping/zones/{id}/methods/{method_id}` | Get method |
| POST | `/shipping/zones/{id}/methods` | Add method |
| PUT | `/shipping/zones/{id}/methods/{method_id}` | Update method |
| DELETE | `/shipping/zones/{id}/methods/{method_id}` | Delete method |

### Shipping Methods

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/shipping_methods` | List all methods |
| GET | `/shipping_methods/{id}` | Get method |

---

## 15. PAYMENT GATEWAYS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/payment_gateways` | List gateways |
| GET | `/payment_gateways/{id}` | Get gateway |
| PUT | `/payment_gateways/{id}` | Update gateway |

### Gateway Properties

```json
{
  "id": "stripe",
  "title": "Credit Card (Stripe)",
  "description": "Pay with your credit card via Stripe.",
  "order": 0,
  "enabled": true,
  "method_title": "Stripe",
  "method_description": "Stripe payment gateway",
  "method_supports": ["products", "refunds"],
  "settings": {
    "title": {
      "id": "title",
      "label": "Title",
      "type": "text",
      "value": "Credit Card (Stripe)"
    }
  }
}
```

---

## 16. SETTINGS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/settings` | List setting groups |
| GET | `/settings/{group}` | List settings in group |
| GET | `/settings/{group}/{id}` | Get setting |
| PUT | `/settings/{group}/{id}` | Update setting |
| POST | `/settings/{group}/batch` | Batch update |

### Setting Groups

- `general` - General settings
- `products` - Product settings
- `tax` - Tax settings
- `shipping` - Shipping settings
- `checkout` - Checkout settings
- `account` - Account settings
- `email` - Email settings
- `advanced` - Advanced settings

---

## 17. WEBHOOKS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/webhooks` | List webhooks |
| GET | `/webhooks/{id}` | Get webhook |
| POST | `/webhooks` | Create webhook |
| PUT | `/webhooks/{id}` | Update webhook |
| DELETE | `/webhooks/{id}` | Delete webhook |
| POST | `/webhooks/batch` | Batch operations |

### Webhook Properties

```json
{
  "id": 1,
  "name": "Order Created Hook",
  "status": "active|paused|disabled",
  "topic": "order.created",
  "resource": "order",
  "event": "created",
  "hooks": ["woocommerce_new_order"],
  "delivery_url": "https://example.com/webhook",
  "secret": "webhook_secret_key",
  "date_created": "2024-01-01T00:00:00",
  "date_modified": "2024-01-15T10:00:00"
}
```

### Available Webhook Topics

| Topic | Description |
|-------|-------------|
| `coupon.created` | Coupon created |
| `coupon.updated` | Coupon updated |
| `coupon.deleted` | Coupon deleted |
| `coupon.restored` | Coupon restored |
| `customer.created` | Customer created |
| `customer.updated` | Customer updated |
| `customer.deleted` | Customer deleted |
| `order.created` | Order created |
| `order.updated` | Order updated |
| `order.deleted` | Order deleted |
| `order.restored` | Order restored |
| `product.created` | Product created |
| `product.updated` | Product updated |
| `product.deleted` | Product deleted |
| `product.restored` | Product restored |

### Custom Action Topics

Για custom events χρησιμοποίησε `action.{action_name}`:

```
action.woocommerce_order_status_processing
action.woocommerce_order_status_completed
action.woocommerce_payment_complete
action.woocommerce_low_stock
action.woocommerce_no_stock
```

---

## 18. SYSTEM STATUS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/system_status` | Get system status |
| GET | `/system_status/tools` | List tools |
| GET | `/system_status/tools/{id}` | Get tool |
| PUT | `/system_status/tools/{id}` | Run tool |

### Available Tools

| Tool ID | Description |
|---------|-------------|
| `clear_transients` | Clear transients |
| `clear_expired_transients` | Clear expired transients |
| `delete_orphaned_variations` | Delete orphan variations |
| `recount_terms` | Recount terms |
| `reset_roles` | Reset user roles |
| `verify_db_tables` | Verify database tables |
| `clear_sessions` | Clear customer sessions |
| `install_pages` | Install WooCommerce pages |
| `delete_taxes` | Delete tax rates |
| `regenerate_thumbnails` | Regenerate thumbnails |
| `db_update_routine` | Update database |

---

## 19. DATA API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/data` | List data endpoints |
| GET | `/data/continents` | List continents |
| GET | `/data/continents/{code}` | Get continent |
| GET | `/data/countries` | List countries |
| GET | `/data/countries/{code}` | Get country |
| GET | `/data/currencies` | List currencies |
| GET | `/data/currencies/{code}` | Get currency |
| GET | `/data/currencies/current` | Get current currency |

---

## 20. AUTHENTICATION

### Basic Auth (Recommended for HTTPS)

```
Authorization: Basic base64(consumer_key:consumer_secret)
```

### Query String Auth (HTTP only - Not recommended)

```
https://store.com/wp-json/wc/v3/products?consumer_key=ck_xxx&consumer_secret=cs_xxx
```

### OAuth 1.0a (For HTTP)

Required parameters:
- `oauth_consumer_key`
- `oauth_timestamp`
- `oauth_nonce`
- `oauth_signature`
- `oauth_signature_method`

---

## 21. ERROR RESPONSES

### Standard Error Format

```json
{
  "code": "woocommerce_rest_product_invalid_id",
  "message": "Invalid ID.",
  "data": {
    "status": 404
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `woocommerce_rest_authentication_error` | 401 | Authentication failed |
| `woocommerce_rest_cannot_view` | 403 | No permission to view |
| `woocommerce_rest_cannot_create` | 403 | No permission to create |
| `woocommerce_rest_cannot_edit` | 403 | No permission to edit |
| `woocommerce_rest_cannot_delete` | 403 | No permission to delete |
| `woocommerce_rest_invalid_id` | 404 | Invalid resource ID |
| `woocommerce_rest_product_invalid_id` | 404 | Invalid product ID |
| `woocommerce_rest_order_invalid_id` | 404 | Invalid order ID |
| `woocommerce_rest_customer_invalid_id` | 404 | Invalid customer ID |

---

## 22. RATE LIMITING & PAGINATION

### Pagination Headers

```
X-WP-Total: 100
X-WP-TotalPages: 10
Link: <https://store.com/wp-json/wc/v3/products?page=2>; rel="next"
```

### Best Practices

1. Use `per_page=100` for maximum efficiency
2. Implement exponential backoff on 429 errors
3. Cache responses where appropriate
4. Use webhooks instead of polling when possible
