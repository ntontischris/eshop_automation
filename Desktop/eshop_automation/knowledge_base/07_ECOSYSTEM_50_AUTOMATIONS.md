# 50 Αυτοματισμοί - Πλήρες Οικοσύστημα WooCommerce

## Overview

Αυτό το αρχείο περιέχει **50 αυτοματισμούς** που μαζί δημιουργούν ένα **πλήρες οικοσύστημα** αυτοματοποίησης για WooCommerce eshops.

Κάθε αυτοματισμός περιλαμβάνει:
- **Τίτλο & Περιγραφή**
- **Prompt για το Skill** (copy-paste ready)
- **Σύνδεση με άλλους αυτοματισμούς**

---

## ΑΡΧΙΤΕΚΤΟΝΙΚΗ ΟΙΚΟΣΥΣΤΗΜΑΤΟΣ

```
┌─────────────────────────────────────────────────────────────────┐
│                    WOOCOMMERCE ECOSYSTEM                        │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: ORDER MANAGEMENT (1-10)                               │
│  ├── New Order Pipeline                                         │
│  ├── Status Automation                                          │
│  ├── Fraud Detection                                            │
│  └── Fulfillment Integration                                    │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: INVENTORY (11-18)                                     │
│  ├── Stock Monitoring                                           │
│  ├── Multi-Store Sync                                           │
│  ├── Supplier Integration                                       │
│  └── Forecasting                                                │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: CUSTOMERS (19-28)                                     │
│  ├── Segmentation                                               │
│  ├── Lifecycle Automation                                       │
│  ├── Loyalty Program                                            │
│  └── Communication                                              │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 4: MARKETING & PRICING (29-38)                           │
│  ├── Dynamic Pricing                                            │
│  ├── Promotions                                                 │
│  ├── Email Marketing                                            │
│  └── Reviews & UGC                                              │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 5: ANALYTICS & AI (39-50)                                │
│  ├── Reporting                                                  │
│  ├── Dashboards                                                 │
│  ├── AI Content                                                 │
│  └── Predictions                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

# LAYER 1: ORDER MANAGEMENT (1-10)

## Αυτοματισμός #1: New Order Notification Hub

**Περιγραφή:** Κεντρικό σύστημα ειδοποιήσεων για κάθε νέα παραγγελία.

**Prompt για Skill:**
```
Φτιάξε μου ένα n8n workflow που όταν δημιουργείται νέα παραγγελία στο WooCommerce:
1. Στέλνει Slack notification στο κανάλι #orders με όλες τις λεπτομέρειες
2. Στέλνει SMS στην ομάδα warehouse
3. Δημιουργεί row στο Google Sheet "Orders Log"
4. Αν η παραγγελία είναι >200€, στέλνει επιπλέον alert στο #vip-orders

Δώσε μου τα πλήρη HTTP requests, το Slack message format, και τον κώδικα για το Google Sheets.
```

**Συνδέεται με:** #2, #3, #5, #39

---

## Αυτοματισμός #2: Order Status Auto-Progression

**Περιγραφή:** Αυτόματη προώθηση status παραγγελιών με βάση events.

**Prompt για Skill:**
```
Φτιάξε workflow που διαχειρίζεται αυτόματα τα order statuses:

1. Όταν γίνεται πληρωμή (payment complete) → status: processing
2. Όταν προστίθεται tracking number (meta update) → status: shipped (custom)
3. Όταν περάσουν 5 μέρες από shipped → status: completed
4. Όταν γίνεται refund → status: refunded + notification

Χρειάζομαι τα webhooks που πρέπει να στήσω, τα HTTP PUT requests για κάθε αλλαγή,
και τη λογική για τον έλεγχο του χρόνου.
```

**Συνδέεται με:** #1, #6, #8, #40

---

## Αυτοματισμός #3: Fraud Detection System

**Περιγραφή:** Αυτόματος έλεγχος παραγγελιών για fraud indicators.

**Prompt για Skill:**
```
Θέλω ένα fraud detection workflow για νέες παραγγελίες που ελέγχει:

Risk Factors:
- Διαφορετική χώρα billing/shipping (+30 score)
- Παραγγελία >500€ (+25 score)
- Guest checkout με υψηλή αξία (+20 score)
- Πολλαπλές αποτυχημένες παραγγελίες από ίδιο email (+40 score)
- Disposable email domain (+15 score)

Actions:
- Score >50: Βάλε on-hold + admin note + Slack alert
- Score >70: On-hold + SMS σε manager
- Score <30: Proceed normally

Δώσε μου τον κώδικα για τον υπολογισμό, τα API calls, και τα notification templates.
```

**Συνδέεται με:** #1, #2, #45

---

## Αυτοματισμός #4: Failed Order Recovery

**Περιγραφή:** Αυτόματη ανάκτηση αποτυχημένων παραγγελιών.

**Prompt για Skill:**
```
Φτιάξε abandoned/failed order recovery workflow:

Timeline:
- 0h: Order fails/pending
- 1h: Πρώτο email reminder με payment link
- 24h: Δεύτερο email με 5% discount code
- 48h: Τρίτο email "Last chance" με 10% discount
- 72h: Final SMS (αν έχουμε phone)

Θέλω:
1. Schedule trigger κάθε ώρα
2. Query για pending orders
3. Έλεγχος meta για reminders που έχουν ήδη σταλεί
4. Δημιουργία personalized coupons μέσω API
5. Email templates για κάθε στάδιο
6. Tracking σε Google Sheet

Δώσε μου όλα τα HTTP requests και τον κώδικα.
```

**Συνδέεται με:** #1, #23, #30

---

## Αυτοματισμός #5: High-Value Order VIP Treatment

**Περιγραφή:** Ειδική μεταχείριση για παραγγελίες υψηλής αξίας.

**Prompt για Skill:**
```
Δημιούργησε VIP order workflow για παραγγελίες >300€:

Actions:
1. Άμεσο Slack notification στο #vip-orders
2. Προσθήκη "_priority: high" meta στην παραγγελία
3. Αυτόματο email στον πελάτη με personal thank you
4. Δημιουργία 15% discount coupon για επόμενη αγορά
5. Προσθήκη στη VIP λίστα αν δεν είναι ήδη
6. Assignment σε senior team member

Θέλω τα πλήρη API calls, τον κώδικα για το coupon generation,
και το email template.
```

**Συνδέεται με:** #1, #19, #22

---

## Αυτοματισμός #6: Shipping Label Auto-Generation

**Περιγραφή:** Αυτόματη δημιουργία shipping labels και tracking.

**Prompt για Skill:**
```
Φτιάξε workflow για αυτόματη δημιουργία shipping labels:

Trigger: Order status → processing

Steps:
1. Fetch full order details με shipping address
2. Κλήση σε shipping provider API (πχ ACS, Speedex, ή generic)
3. Λήψη tracking number και label PDF
4. Update order meta με:
   - _tracking_number
   - _tracking_provider
   - _label_url
5. Προσθήκη customer-visible note με tracking link
6. Email στον πελάτη με tracking info

Δώσε μου τη δομή για integration με shipping API και τα WooCommerce updates.
```

**Συνδέεται με:** #2, #7, #8

---

## Αυτοματισμός #7: Delivery Status Tracking

**Περιγραφή:** Παρακολούθηση delivery status και alerts.

**Prompt για Skill:**
```
Θέλω workflow για tracking delivery status:

Schedule: Κάθε 4 ώρες

Logic:
1. Get all orders με status "shipped" και tracking number
2. Για κάθε order: Query carrier API για delivery status
3. Actions based on status:
   - "Delivered": Update order → completed + thank you email
   - "Exception/Delayed": Alert admin + notify customer
   - "Returned to sender": Alert + investigation flag
4. Log all status changes

Χρειάζομαι τη λογική για carrier API integration και τα update flows.
```

**Συνδέεται με:** #2, #6, #24

---

## Αυτοματισμός #8: Order Notes Automation

**Περιγραφή:** Αυτόματες σημειώσεις σε orders για audit trail.

**Prompt για Skill:**
```
Δημιούργησε automatic order notes workflow:

Προσθήκη note σε κάθε event:
- Δημιουργία: "Order created via [checkout/admin/api]"
- Payment: "Payment received via [method] - Transaction: [id]"
- Status change: "Status changed from [old] to [new] by [system/user]"
- Tracking: "Tracking added: [number] via [carrier]"
- Customer contact: "Email sent: [type]"
- Modification: "Order modified: [details]"

Θέλω να είναι όλα customer_note: false (internal only).
Δώσε μου τα API calls για order notes creation.
```

**Συνδέεται με:** #1, #2, #6

---

## Αυτοματισμός #9: Refund Processing Pipeline

**Περιγραφή:** Αυτοματοποιημένη διαδικασία refunds.

**Prompt για Skill:**
```
Φτιάξε refund processing workflow:

Trigger: Custom webhook ή manual trigger

Steps:
1. Validate refund request (order exists, not already refunded)
2. Calculate refund amount (full or partial)
3. Create refund via WooCommerce API
4. If partial: Restock items automatically
5. Update customer record με refund history
6. Send refund confirmation email
7. Notify accounting system
8. Log to "Refunds" Google Sheet

Include error handling για failed refunds.
```

**Συνδέεται με:** #2, #8, #40

---

## Αυτοματισμός #10: Order Split for Multiple Warehouses

**Περιγραφή:** Διαχωρισμός παραγγελιών ανά warehouse.

**Prompt για Skill:**
```
Θέλω workflow που χωρίζει orders σε sub-orders ανά warehouse:

Logic:
1. New order trigger
2. Για κάθε line item: Check product meta "_warehouse"
3. Group items by warehouse
4. If multiple warehouses:
   - Create child orders για κάθε warehouse
   - Link με parent_id
   - Add meta "_warehouse" σε κάθε child
   - Notify respective warehouse team
5. Original order: Add note με child order IDs

Δώσε μου τον κώδικα για grouping και τα API calls για order creation.
```

**Συνδέεται με:** #1, #6, #11

---

# LAYER 2: INVENTORY MANAGEMENT (11-18)

## Αυτοματισμός #11: Real-Time Stock Monitor

**Περιγραφή:** 24/7 παρακολούθηση αποθέματος.

**Prompt για Skill:**
```
Φτιάξε real-time stock monitoring system:

Schedule: Κάθε 2 ώρες (ή webhook on stock change)

Levels:
- Critical (stock < 5): Immediate Slack alert + SMS
- Warning (stock < 15): Daily digest email
- Low (stock < 30): Weekly report

Output:
1. Slack message με critical items
2. Daily email digest με warning items
3. Google Sheet "Stock Levels" update
4. Dashboard data push

Θέλω τα queries για products, τη λογική filtering, και τα notification templates.
```

**Συνδέεται με:** #12, #13, #14, #41

---

## Αυτοματισμός #12: Automatic Reorder System

**Περιγραφή:** Αυτόματες παραγγελίες προς suppliers.

**Prompt για Skill:**
```
Δημιούργησε automatic reorder workflow:

Logic:
1. Daily check: stock_quantity <= reorder_point (από product meta)
2. Για κάθε product που χρειάζεται reorder:
   - Calculate reorder quantity (meta: _reorder_qty ή default)
   - Get supplier info (meta: _supplier_email, _supplier_sku)
3. Group by supplier
4. Generate PO (Purchase Order):
   - PO number: PO-{date}-{sequence}
   - Items list με quantities
5. Email PO σε κάθε supplier
6. Create PO record σε Google Sheet
7. Update product meta: _last_po_date, _last_po_number

Δώσε μου τα templates και τον κώδικα.
```

**Συνδέεται με:** #11, #13, #16

---

## Αυτοματισμός #13: Multi-Store Inventory Sync

**Περιγραφή:** Συγχρονισμός stock μεταξύ πολλαπλών stores.

**Prompt για Skill:**
```
Φτιάξε multi-store inventory sync:

Stores Configuration:
- Store A (Primary): https://store-a.com
- Store B: https://store-b.com
- Store C: https://store-c.com

Trigger: Product updated webhook (stock change) από οποιοδήποτε store

Logic:
1. Receive stock update από source store
2. Extract SKU και new stock level
3. For each OTHER store:
   - GET /products?sku={sku}
   - If exists: PUT /products/{id} με νέο stock
4. Update master inventory (Airtable ή Google Sheet)
5. Log sync operation

Handle conflicts: Source store always wins.
Δώσε μου τη δομή για credentials management και τα sync calls.
```

**Συνδέεται με:** #11, #14, #15

---

## Αυτοματισμός #14: Stock Forecast & Prediction

**Περιγραφή:** Πρόβλεψη αποθέματος με βάση ιστορικά δεδομένα.

**Prompt για Skill:**
```
Δημιούργησε stock forecasting workflow:

Schedule: Weekly (Δευτέρα 6:00)

Analysis:
1. Get sales data τελευταίων 90 ημερών per product
2. Calculate:
   - Average daily sales
   - Sales trend (increasing/decreasing)
   - Seasonality factor (if applicable)
3. Predict:
   - Days until stockout
   - Recommended reorder date
   - Suggested order quantity

Output:
1. "Stock Forecast" Google Sheet με predictions
2. Alert για products με <14 days until stockout
3. Weekly forecast report email

Δώσε μου τον κώδικα για calculations και τα API queries.
```

**Συνδέεται με:** #11, #12, #41

---

## Αυτοματισμός #15: Dead Stock Identifier

**Περιγραφή:** Εντοπισμός προϊόντων χωρίς πωλήσεις.

**Prompt για Skill:**
```
Φτιάξε dead stock identification workflow:

Schedule: Monthly

Criteria:
- No sales in 90+ days
- Has stock > 0

Analysis per product:
1. Get all products με stock
2. For each: Query orders για last sale date
3. Calculate days since last sale
4. Classify:
   - Slow moving: 60-90 days
   - Dead stock: 90+ days
   - Never sold: No sales ever

Actions:
1. Generate report με dead stock value
2. Recommendations:
   - Discount suggestions
   - Bundle opportunities
   - Clearance candidates
3. Email report to management

Δώσε μου queries και calculations.
```

**Συνδέεται με:** #11, #14, #32

---

## Αυτοματισμός #16: Supplier Performance Tracking

**Περιγραφή:** Παρακολούθηση απόδοσης suppliers.

**Prompt για Skill:**
```
Δημιούργησε supplier tracking system:

Data Collection (on PO fulfillment):
1. Log delivery date vs expected date
2. Log quality issues (manual input via form)
3. Log quantity discrepancies

Metrics per Supplier:
- On-time delivery rate
- Quality score
- Order accuracy
- Average lead time

Schedule: Monthly report

Output:
1. Supplier scorecard Google Sheet
2. Alert if supplier score drops below threshold
3. Monthly performance email to procurement

Δώσε μου τη δομή data και calculations.
```

**Συνδέεται με:** #12, #17

---

## Αυτοματισμός #17: Inventory Value Report

**Περιγραφή:** Υπολογισμός αξίας αποθέματος.

**Prompt για Skill:**
```
Φτιάξε inventory valuation workflow:

Schedule: Weekly

Calculations:
1. Get all products με stock
2. For each product:
   - Stock value = stock_quantity × cost (meta: _cost ή regular_price × 0.6)
   - Retail value = stock_quantity × regular_price
3. Group by category
4. Calculate totals

Output:
1. Summary:
   - Total cost value
   - Total retail value
   - Potential profit margin
2. Breakdown by category
3. Top 10 highest value items
4. Google Sheet "Inventory Value" update

Δώσε μου queries και calculations code.
```

**Συνδέεται με:** #11, #14, #41

---

## Αυτοματισμός #18: Bundle Stock Manager

**Περιγραφή:** Διαχείριση stock για bundled products.

**Prompt για Skill:**
```
Θέλω bundle stock management:

Logic:
1. Define bundles via product meta:
   - _bundle_components: [{product_id, quantity}, ...]
2. On component stock change:
   - Calculate max possible bundles
   - Update bundle stock accordingly
3. On bundle sale:
   - Decrease component stock

Trigger: Any component product stock update

Workflow:
1. Get bundle products
2. For each bundle: Calculate available qty based on components
3. Update bundle stock if changed
4. Alert if bundle becomes unavailable

Δώσε μου τη λογική και τα API calls.
```

**Συνδέεται με:** #11, #13

---

# LAYER 3: CUSTOMER MANAGEMENT (19-28)

## Αυτοματισμός #19: Customer Segmentation Engine

**Περιγραφή:** Αυτόματη κατηγοριοποίηση πελατών.

**Prompt για Skill:**
```
Δημιούργησε customer segmentation workflow:

Schedule: Daily (για νέους) + Weekly (full recalculation)

Segments:
- VIP: total_spent >= €1000 OR orders >= 10
- Gold: total_spent >= €500
- Silver: total_spent >= €200
- Bronze: total_spent >= €50
- New: First order < 30 days
- At-Risk: No order 60+ days (but has history)
- Churned: No order 90+ days
- Inactive: Registered but never ordered

Actions:
1. Calculate segment για κάθε customer
2. Update customer meta: _segment, _segment_updated
3. Sync segments to email marketing platform
4. Generate segment movement report

Δώσε μου τον κώδικα segmentation και τα API calls.
```

**Συνδέεται με:** #5, #20, #22, #23

---

## Αυτοματισμός #20: New Customer Welcome Series

**Περιγραφή:** Welcome sequence για νέους πελάτες.

**Prompt για Skill:**
```
Φτιάξε new customer welcome workflow:

Trigger: First order completed

Sequence:
- Day 0: Welcome email + 10% discount code για επόμενη αγορά
- Day 3: "How to get the most from your purchase" email
- Day 7: Review request email
- Day 14: Product recommendations based on purchase
- Day 21: "We miss you" αν δεν έχει κάνει 2η αγορά

Requirements:
1. Create unique coupon per customer
2. Track email sends στο customer meta
3. Stop sequence αν γίνει 2η αγορά
4. Personalized content βάσει purchased category

Δώσε μου τα API calls, coupon creation, και email triggers.
```

**Συνδέεται με:** #19, #23, #24

---

## Αυτοματισμός #21: Customer Birthday Rewards

**Περιγραφή:** Αυτόματα birthday rewards.

**Prompt για Skill:**
```
Δημιούργησε birthday rewards workflow:

Setup:
- Birthday stored in customer meta: _birthday (YYYY-MM-DD)

Schedule: Daily at 9:00

Logic:
1. Find customers με birthday = today (ή within 3 days)
2. Check if reward already sent this year
3. Generate personalized:
   - Birthday email
   - Special coupon (15% off, valid 7 days)
4. Update meta: _birthday_reward_{year}

Extra:
- VIP customers get 20% instead of 15%
- Include personalized product recommendations

Δώσε μου τα date calculations, coupon API, και email template.
```

**Συνδέεται με:** #19, #30

---

## Αυτοματισμός #22: VIP Customer Management

**Περιγραφή:** Ειδική διαχείριση VIP πελατών.

**Prompt για Skill:**
```
Φτιάξε VIP management system:

VIP Criteria:
- Total spent >= €1000 OR
- Orders >= 10 OR
- Manual VIP flag

VIP Benefits (automatic):
1. Free shipping on all orders
2. Early access to sales (24h before)
3. Dedicated support email
4. Birthday double discount
5. Exclusive VIP-only products visibility

Workflows needed:
1. VIP qualification check (on order complete)
2. Apply free shipping automatically
3. VIP notification για new products
4. Quarterly VIP appreciation email

Δώσε μου τα API calls για κάθε benefit implementation.
```

**Συνδέεται με:** #5, #19, #21

---

## Αυτοματισμός #23: Winback Campaign Automation

**Περιγραφή:** Αυτόματες καμπάνιες επαναπροσέλκυσης.

**Prompt για Skill:**
```
Δημιούργησε winback campaign workflow:

Trigger: Customer segment changes to "At-Risk" ή "Churned"

Sequences:

At-Risk (60-90 days):
- Email 1: "We miss you" + 10% off
- +7 days: Product recommendations
- +14 days: 15% off "last chance"

Churned (90+ days):
- Email 1: "Come back" + 20% off
- +7 days: Survey "Why did you leave?"
- +14 days: Final offer 25% off

Personalization:
- Include last purchased products
- Recommend similar items
- Show what's new since last visit

Track: Opens, clicks, conversions, revenue recovered

Δώσε μου τα workflows, coupons, και email templates.
```

**Συνδέεται με:** #19, #20, #30

---

## Αυτοματισμός #24: Post-Purchase Follow-up

**Περιγραφή:** Follow-up emails μετά την αγορά.

**Prompt για Skill:**
```
Φτιάξε post-purchase follow-up sequence:

Trigger: Order status → completed

Timeline:
- +1 day: Delivery confirmation + care instructions
- +5 days: "How's your purchase?" check-in
- +10 days: Review request (με incentive)
- +20 days: Cross-sell recommendations
- +30 days: Replenishment reminder (για consumables)

Smart features:
1. Skip review request αν already reviewed
2. Different timing για different product types
3. Include product-specific care tips
4. Personalized recommendations based on purchase

Δώσε μου τη λογική, τα API calls, και templates.
```

**Συνδέεται με:** #7, #20, #35

---

## Αυτοματισμός #25: Customer Lifetime Value Calculator

**Περιγραφή:** Υπολογισμός και tracking CLV.

**Prompt για Skill:**
```
Δημιούργησε CLV calculation workflow:

Schedule: Weekly

Metrics per Customer:
1. Historical CLV = Sum of all order totals
2. Average Order Value = Historical / Order count
3. Purchase Frequency = Orders / months as customer
4. Predicted CLV = AOV × Frequency × Expected lifetime

Store in customer meta:
- _clv_historical
- _clv_predicted
- _aov
- _purchase_frequency
- _clv_updated

Actions:
1. Update all customer CLV values
2. Identify high-potential customers (high predicted, low historical)
3. Alert για CLV drops
4. Weekly CLV report

Δώσε μου calculations και API updates.
```

**Συνδέεται με:** #19, #22, #42

---

## Αυτοματισμός #26: Customer Feedback Collection

**Περιγραφή:** Συλλογή feedback και NPS.

**Prompt για Skill:**
```
Φτιάξε feedback collection system:

Triggers:
1. After 3rd order: NPS survey
2. After support ticket closed: CSAT survey
3. After return/refund: Experience survey
4. Quarterly: General feedback

NPS Flow:
1. Send NPS email (1-10 scale)
2. Capture response via webhook
3. Categorize:
   - Promoters (9-10): Thank you + referral program invite
   - Passives (7-8): Ask for improvement suggestions
   - Detractors (0-6): Immediate alert + personal follow-up
4. Store score in customer meta
5. Calculate overall NPS

Δώσε μου τα survey triggers, response handling, και calculations.
```

**Συνδέεται με:** #24, #27, #43

---

## Αυτοματισμός #27: Referral Program Automation

**Περιγραφή:** Αυτοματοποιημένο referral program.

**Prompt για Skill:**
```
Δημιούργησε referral program workflow:

Program Structure:
- Referrer gets: €10 credit per successful referral
- Referee gets: 15% off first order

Workflow:
1. Generate unique referral code per customer
2. Track referral link clicks
3. On referee's first order:
   - Validate referral code
   - Apply referee discount
   - Credit referrer account
   - Notify referrer
4. Monthly referral leaderboard

Storage:
- Customer meta: _referral_code, _referrals_count, _referral_credit
- Tracking: Referrals Google Sheet

Δώσε μου code generation, tracking, και credit system.
```

**Συνδέεται με:** #22, #26, #30

---

## Αυτοματισμός #28: Customer Data Enrichment

**Περιγραφή:** Εμπλουτισμός customer profiles.

**Prompt για Skill:**
```
Φτιάξε customer data enrichment workflow:

Trigger: New customer OR weekly batch

Data to Calculate/Enrich:
1. Purchase patterns:
   - Favorite category
   - Preferred price range
   - Average days between orders
2. Engagement:
   - Email open rate
   - Last website visit
   - Wishlist items
3. Predictions:
   - Next likely purchase category
   - Churn probability
   - Upsell potential

Store in customer meta:
- _favorite_category
- _price_sensitivity (low/medium/high)
- _purchase_cycle_days
- _churn_risk (low/medium/high)
- _enriched_date

Δώσε μου calculations και data structure.
```

**Συνδέεται με:** #19, #25, #46

---

# LAYER 4: MARKETING & PRICING (29-38)

## Αυτοματισμός #29: Dynamic Pricing Engine

**Περιγραφή:** Αυτόματη προσαρμογή τιμών.

**Prompt για Skill:**
```
Δημιούργησε dynamic pricing workflow:

Schedule: Daily at 6:00

Rules:
1. Stock-based pricing:
   - Stock > 100: Normal price
   - Stock < 20: +10% (scarcity)
   - Stock < 5: +15%
2. Demand-based:
   - Sales velocity high: +5%
   - Sales velocity low: -5%
3. Competition-based (if competitor price available):
   - Beat competitor by 3%
4. Minimum margin protection: Never below 20% margin

Output:
1. Update product prices via API
2. Log all price changes
3. Daily price change report
4. Alert για significant changes (>10%)

Δώσε μου calculations, rules engine, και API calls.
```

**Συνδέεται με:** #11, #14, #32

---

## Αυτοματισμός #30: Coupon Management System

**Περιγραφή:** Κεντρικό σύστημα διαχείρισης κουπονιών.

**Prompt για Skill:**
```
Φτιάξε coupon management workflow:

Features:
1. Auto-generate coupons για:
   - Welcome (new customer)
   - Birthday
   - Winback
   - Referral
   - VIP exclusive
2. Expiry management:
   - Send reminder 3 days before expiry
   - Auto-delete expired coupons
3. Performance tracking:
   - Usage rate
   - Revenue generated
   - ROI per coupon type
4. Fraud prevention:
   - One-time use enforcement
   - Customer-specific coupons

Weekly report: Coupon performance summary

Δώσε μου generation logic, tracking, και cleanup workflows.
```

**Συνδέεται με:** #4, #20, #21, #23

---

## Αυτοματισμός #31: Flash Sale Automation

**Περιγραφή:** Αυτοματοποίηση flash sales.

**Prompt για Skill:**
```
Δημιούργησε flash sale workflow:

Setup (via Google Sheet ή form):
- Products to include
- Discount percentage
- Start datetime
- End datetime

Workflow:
1. Pre-sale (1 hour before):
   - Email subscribers
   - Social media posts via Zapier/webhook
2. Sale start:
   - Batch update prices με sale_price
   - Set date_on_sale_from/to
   - Enable "Sale" badge
3. During sale:
   - Real-time sales tracking
   - Stock alerts
   - Hourly Slack updates
4. Sale end:
   - Remove sale prices
   - Performance report
   - Return products to normal

Δώσε μου τα batch updates, scheduling, και reporting.
```

**Συνδέεται με:** #29, #39, #40

---

## Αυτοματισμός #32: Clearance & Discount Manager

**Περιγραφή:** Διαχείριση εκπτώσεων για dead stock.

**Prompt για Skill:**
```
Φτιάξε automatic clearance workflow:

Trigger: Weekly analysis

Rules:
1. Products με no sales >60 days:
   - Apply 20% discount
   - Add "Clearance" tag
2. Products με no sales >90 days:
   - Increase to 30% discount
   - Feature in "Deals" section
3. Products με no sales >120 days:
   - Apply 50% discount
   - Alert for manual review (donate/dispose?)

Notifications:
- Weekly clearance report
- Alert when product enters each tier
- Monthly clearance revenue report

Δώσε μου τα date calculations, price updates, και tagging.
```

**Συνδέεται με:** #15, #29

---

## Αυτοματισμός #33: Email Marketing Sync

**Περιγραφή:** Συγχρονισμός με email marketing platform.

**Prompt για Skill:**
```
Δημιούργησε email marketing sync workflow:

Platform: Mailchimp/Klaviyo/generic webhook

Sync Events:
1. New customer → Add to "Customers" list
2. Segment change → Update tags
3. Order completed → Trigger post-purchase flow
4. Cart abandoned → Trigger recovery flow
5. VIP status → Add to "VIP" segment

Data to Sync:
- Email, name
- Total spent, order count
- Segment
- Last order date
- Favorite category
- Custom properties

Schedule: Real-time (webhooks) + daily full sync

Δώσε μου webhook handlers και sync API calls.
```

**Συνδέεται με:** #19, #20, #23

---

## Αυτοματισμός #34: Social Proof Automation

**Περιγραφή:** Αυτόματη συλλογή και display social proof.

**Prompt για Skill:**
```
Φτιάξε social proof automation:

Features:
1. Recent purchases notification:
   - "[Name] from [City] just bought [Product]"
   - Webhook για frontend display
2. Review highlights:
   - Auto-select best reviews per product
   - Feature in product meta
3. Sales counters:
   - Update "X sold this week" badges
   - Track trending products
4. User-generated content:
   - Collect Instagram posts με hashtag
   - Request permission για display

Updates: Every 30 minutes

Δώσε μου data collection, storage, και webhook output.
```

**Συνδέεται με:** #35, #39

---

## Αυτοματισμός #35: Review Management System

**Περιγραφή:** Πλήρης διαχείριση reviews.

**Prompt για Skill:**
```
Δημιούργησε review management workflow:

Collection:
1. Request review (post-purchase workflow)
2. Incentivize: 5% off coupon για review

Moderation:
1. New review alert
2. Sentiment analysis (AI):
   - Positive: Auto-approve
   - Neutral: Auto-approve
   - Negative: Flag για review
3. Spam detection

Response:
1. Negative reviews (<3 stars):
   - Immediate admin alert
   - Draft response suggestion (AI)
   - Follow-up tracking
2. Positive reviews:
   - Thank you response
   - Request για social share

Analytics:
- Average rating per product
- Review sentiment trends
- Response rate

Δώσε μου τα API calls, sentiment logic, και workflows.
```

**Συνδέεται με:** #24, #26, #47

---

## Αυτοματισμός #36: Wishlist & Back-in-Stock

**Περιγραφή:** Wishlist tracking και back-in-stock alerts.

**Prompt για Skill:**
```
Φτιάξε wishlist & back-in-stock system:

Wishlist:
1. Track wishlist additions (via plugin webhook ή custom)
2. Store wishlist items per customer
3. Wishlist-based recommendations
4. Price drop alerts για wishlist items

Back-in-Stock:
1. Collect email για out-of-stock products
2. On stock update (quantity > 0):
   - Find all subscribers για αυτό το product
   - Send "Back in Stock" email
   - Include quick-buy link
3. Priority access για VIP customers

Δώσε μου data storage, triggers, και email templates.
```

**Συνδέεται με:** #11, #22, #24

---

## Αυτοματισμός #37: Cross-sell & Upsell Engine

**Περιγραφή:** Αυτόματες προτάσεις cross-sell/upsell.

**Prompt για Skill:**
```
Δημιούργησε cross-sell/upsell workflow:

Data Collection:
1. Track "frequently bought together"
2. Analyze order patterns
3. Calculate product affinity scores

Recommendations:
1. On product view: Related products (webhook για frontend)
2. In cart: Upsell suggestions
3. Post-purchase email: Cross-sell

Logic:
1. Get product
2. Find products frequently bought with it
3. Calculate affinity score
4. Store as product meta: _related_products

Update: Weekly recalculation

Δώσε μου τον algorithm, data structure, και API updates.
```

**Συνδέεται με:** #24, #28, #46

---

## Αυτοματισμός #38: Seasonal Campaign Manager

**Περιγραφή:** Διαχείριση seasonal campaigns.

**Prompt για Skill:**
```
Φτιάξε seasonal campaign automation:

Calendar (configurable):
- Valentine's Day (Feb 1-14)
- Easter (dynamic dates)
- Summer Sale (Jun 15 - Jul 15)
- Black Friday (Nov last week)
- Christmas (Dec 1-24)
- New Year (Dec 26 - Jan 7)

Auto-actions per season:
1. Apply seasonal tags to products
2. Enable seasonal banners (via meta/webhook)
3. Activate seasonal coupons
4. Adjust email templates
5. Update social media content

Schedule: Check daily, activate/deactivate automatically

Δώσε μου calendar management, actions, και cleanup.
```

**Συνδέεται με:** #31, #33, #34

---

# LAYER 5: ANALYTICS & AI (39-50)

## Αυτοματισμός #39: Daily KPI Dashboard

**Περιγραφή:** Real-time KPI tracking.

**Prompt για Skill:**
```
Δημιούργησε daily KPI dashboard workflow:

Schedule: Every 15 minutes (real-time) + daily summary

KPIs:
1. Sales:
   - Revenue today/MTD/YTD
   - Orders today/MTD
   - Average order value
   - Conversion rate (if traffic data available)
2. Products:
   - Top 10 sellers today
   - Low stock alerts
   - Out-of-stock count
3. Customers:
   - New customers today
   - Repeat customers
   - Active cart count
4. Operations:
   - Orders pending processing
   - Orders shipped
   - Refund rate

Output:
1. Google Sheet "Live Dashboard"
2. Slack #metrics channel updates
3. Daily email digest

Δώσε μου τα API calls, calculations, και output formatting.
```

**Συνδέεται με:** #1, #11, #40

---

## Αυτοματισμός #40: Weekly Performance Report

**Περιγραφή:** Εβδομαδιαίο performance report.

**Prompt για Skill:**
```
Φτιάξε weekly report workflow:

Schedule: Monday 8:00

Sections:
1. Executive Summary:
   - Revenue vs last week vs same week last year
   - Orders comparison
   - Key highlights
2. Sales Analysis:
   - Revenue by day
   - Revenue by category
   - Top products
   - Worst performers
3. Customer Metrics:
   - New vs returning
   - Customer acquisition cost (if ad data)
   - Segment movements
4. Inventory:
   - Stock value
   - Stock turn rate
   - Reorder alerts
5. Marketing:
   - Coupon performance
   - Email metrics
   - Campaign results
6. Operations:
   - Fulfillment time
   - Refund rate
   - Support tickets

Output: PDF report + Email

Δώσε μου data collection, calculations, και report generation.
```

**Συνδέεται με:** #39, #41, #43

---

## Αυτοματισμός #41: Revenue & Profit Analytics

**Περιγραφή:** Βαθιά analysis revenue και profit.

**Prompt για Skill:**
```
Δημιούργησε revenue analytics workflow:

Schedule: Daily

Calculations:
1. Gross Revenue: Sum of order totals
2. Net Revenue: Gross - Refunds - Discounts
3. COGS: Sum of (product cost × quantity sold)
4. Gross Profit: Net Revenue - COGS
5. Gross Margin %: Gross Profit / Net Revenue

Breakdowns:
- By product
- By category
- By customer segment
- By traffic source (if available)

Output:
1. "Revenue Analytics" Google Sheet
2. Profit margin alerts (if below threshold)
3. Monthly trend analysis

Δώσε μου calculations με product cost handling.
```

**Συνδέεται με:** #17, #39, #40

---

## Αυτοματισμός #42: Customer Analytics Dashboard

**Περιγραφή:** Customer-focused analytics.

**Prompt για Skill:**
```
Φτιάξε customer analytics workflow:

Schedule: Daily

Metrics:
1. Customer Acquisition:
   - New customers today/week/month
   - Acquisition cost (if ad spend data)
   - First purchase value
2. Retention:
   - Repeat purchase rate
   - Time to second purchase
   - Churn rate
3. Value:
   - Average CLV
   - CLV by segment
   - Revenue per customer
4. Engagement:
   - Active customers (30/60/90 days)
   - Email engagement
   - Review rate

Cohort Analysis:
- Monthly cohort retention

Output: Google Sheet + Visualizations data

Δώσε μου calculations και cohort logic.
```

**Συνδέεται με:** #19, #25, #40

---

## Αυτοματισμός #43: NPS & Satisfaction Tracking

**Περιγραφή:** Tracking NPS και customer satisfaction.

**Prompt για Skill:**
```
Δημιούργησε NPS tracking workflow:

Collection (from #26):
- Store NPS responses with timestamp

Analysis:
1. Overall NPS calculation
2. NPS by segment
3. NPS trend over time
4. Response themes (from comments)

Alerts:
- NPS drops below threshold
- Increase in detractors
- Trending negative comments

Reporting:
- Monthly NPS report
- Segment comparison
- Improvement tracking

Actions:
- Auto-escalate detractor responses
- Celebrate promoters
- Track resolution of issues

Δώσε μου calculations, trend analysis, και reporting.
```

**Συνδέεται με:** #26, #40, #42

---

## Αυτοματισμός #44: Product Performance Analytics

**Περιγραφή:** Βαθιά ανάλυση απόδοσης προϊόντων.

**Prompt για Skill:**
```
Φτιάξε product analytics workflow:

Schedule: Weekly

Per Product Metrics:
1. Sales:
   - Units sold
   - Revenue generated
   - Revenue trend
2. Profitability:
   - Profit margin
   - ROI
3. Performance:
   - Views to purchase rate
   - Cart abandonment rate
   - Return rate
4. Inventory:
   - Stock turn rate
   - Days of inventory

Rankings:
- Top performers
- Underperformers
- Rising stars (trend up)
- Declining (trend down)

Output:
1. Product scorecard
2. Action recommendations
3. Google Sheet update

Δώσε μου calculations και scoring logic.
```

**Συνδέεται με:** #14, #15, #41

---

## Αυτοματισμός #45: Anomaly Detection System

**Περιγραφή:** Αυτόματη ανίχνευση ανωμαλιών.

**Prompt για Skill:**
```
Δημιούργησε anomaly detection workflow:

Schedule: Every hour

Monitor for:
1. Sales anomalies:
   - Sudden spike (>200% of average)
   - Sudden drop (>50% below average)
2. Order anomalies:
   - Unusual order patterns
   - Multiple orders same customer quickly
   - Unusual AOV
3. Inventory anomalies:
   - Stock discrepancies
   - Unusual stock movements
4. System anomalies:
   - High error rates
   - Slow API responses
   - Payment failures spike

Alerts:
- Slack immediate για critical
- Email digest για warnings

Δώσε μου baseline calculations, deviation detection, και alert logic.
```

**Συνδέεται με:** #3, #39, #41

---

## Αυτοματισμός #46: AI Product Recommendations

**Περιγραφή:** AI-powered recommendations.

**Prompt για Skill:**
```
Φτιάξε AI recommendation workflow:

Types:
1. Similar products (content-based)
2. Frequently bought together (collaborative)
3. Personalized (customer history)
4. Trending (popularity-based)

Implementation:
1. Collect purchase data
2. Calculate product similarities
3. Build customer profiles
4. Generate recommendations
5. Store in product/customer meta
6. Serve via API/webhook

Update frequency: Daily

Output:
- Product meta: _recommendations
- Customer meta: _personalized_recommendations
- API endpoint για real-time

Δώσε μου algorithms και data structures.
```

**Συνδέεται με:** #28, #37

---

## Αυτοματισμός #47: AI Content Generator

**Περιγραφή:** AI-generated product content.

**Prompt για Skill:**
```
Δημιούργησε AI content generation workflow:

Trigger: New product (missing description) OR manual

Generate:
1. SEO Title (60 chars)
2. Meta Description (155 chars)
3. Short Description (2-3 sentences)
4. Full Description (300-500 words)
5. Bullet points features
6. SEO keywords

Input:
- Product name
- Category
- Attributes
- Price

AI Prompt Structure:
- Context: E-commerce product copywriting
- Tone: Professional, persuasive
- Include: Benefits, features, call-to-action

Output: Update product via API

Δώσε μου AI prompts, API integration, και update workflow.
```

**Συνδέεται με:** #35, #48

---

## Αυτοματισμός #48: AI Review Response Generator

**Περιγραφή:** AI-generated review responses.

**Prompt για Skill:**
```
Φτιάξε AI review response workflow:

Trigger: New review received

Process:
1. Analyze review sentiment
2. Extract key points
3. Generate appropriate response:
   - Positive: Thank, encourage sharing
   - Neutral: Thank, ask for feedback
   - Negative: Apologize, offer solution, contact info
4. Queue για approval (optional)
5. Post response

Personalization:
- Include reviewer name
- Reference specific product
- Reference specific feedback points

Tone: Professional, empathetic, solution-oriented

Δώσε μου sentiment analysis, response templates, και posting workflow.
```

**Συνδέεται με:** #35, #47

---

## Αυτοματισμός #49: Predictive Analytics Engine

**Περιγραφή:** Predictions για sales και behavior.

**Prompt για Skill:**
```
Δημιούργησε predictive analytics workflow:

Schedule: Weekly

Predictions:
1. Sales Forecast:
   - Next week revenue (based on trends)
   - Next month revenue
   - Seasonality adjustments
2. Customer Predictions:
   - Churn probability per customer
   - Next purchase timing
   - Next purchase category
3. Inventory Predictions:
   - Demand forecast per product
   - Stockout probability
   - Optimal reorder timing
4. Marketing Predictions:
   - Best send time για emails
   - Optimal discount level

Output:
- Predictions stored in metas
- Weekly prediction report
- Accuracy tracking (actual vs predicted)

Δώσε μου forecasting models και storage.
```

**Συνδέεται με:** #14, #28, #42

---

## Αυτοματισμός #50: Ecosystem Health Monitor

**Περιγραφή:** Master monitoring όλου του οικοσυστήματος.

**Prompt για Skill:**
```
Φτιάξε ecosystem health monitor:

Schedule: Every 5 minutes

Check:
1. All workflow statuses (n8n API)
2. WooCommerce API health
3. External services status:
   - Email service
   - Payment gateways
   - Shipping providers
4. Data freshness:
   - Last successful sync times
   - Stale data alerts
5. Error rates:
   - Failed workflows
   - API errors
   - Webhook failures

Dashboard:
- Overall health score (0-100)
- Component statuses
- Recent errors
- Uptime metrics

Alerts:
- Immediate: Critical component down
- Warning: Degraded performance
- Info: Scheduled maintenance

Δώσε μου health checks, scoring, και alert logic.
```

**Συνδέεται με:** ALL (Master orchestrator)

---

# IMPLEMENTATION ROADMAP

## Phase 1: Foundation (Week 1-2)
1. #1 New Order Notification Hub
2. #2 Order Status Auto-Progression
3. #11 Real-Time Stock Monitor
4. #39 Daily KPI Dashboard
5. #50 Ecosystem Health Monitor

## Phase 2: Customer Core (Week 3-4)
6. #19 Customer Segmentation Engine
7. #20 New Customer Welcome Series
8. #24 Post-Purchase Follow-up
9. #33 Email Marketing Sync
10. #35 Review Management System

## Phase 3: Inventory & Operations (Week 5-6)
11. #3 Fraud Detection System
12. #6 Shipping Label Auto-Generation
13. #12 Automatic Reorder System
14. #13 Multi-Store Inventory Sync
15. #8 Order Notes Automation

## Phase 4: Revenue Optimization (Week 7-8)
16. #4 Failed Order Recovery
17. #5 High-Value Order VIP Treatment
18. #29 Dynamic Pricing Engine
19. #30 Coupon Management System
20. #37 Cross-sell & Upsell Engine

## Phase 5: Advanced Analytics (Week 9-10)
21. #40 Weekly Performance Report
22. #41 Revenue & Profit Analytics
23. #42 Customer Analytics Dashboard
24. #44 Product Performance Analytics
25. #45 Anomaly Detection System

## Phase 6: AI & Automation (Week 11-12)
26. #47 AI Content Generator
27. #48 AI Review Response Generator
28. #46 AI Product Recommendations
29. #49 Predictive Analytics Engine
30. #14 Stock Forecast & Prediction

## Phase 7: Complete Ecosystem (Week 13-14)
31-50. Remaining automations based on priority

---

# ESTIMATED RESULTS

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Manual work hours/week | 40 | 10 | -75% |
| Order processing time | 4 hours | 30 min | -87% |
| Stock-out incidents | 15/month | 2/month | -87% |
| Customer response time | 24 hours | 1 hour | -96% |
| Revenue from recovery | €0 | €2,000/month | New |
| Customer retention | 20% | 35% | +75% |
| Review collection rate | 5% | 25% | +400% |
| Data accuracy | 85% | 99% | +16% |

---

*Αυτό το οικοσύστημα δημιουργεί ένα πλήρως αυτοματοποιημένο WooCommerce store που τρέχει 24/7 με ελάχιστη χειροκίνητη παρέμβαση.*
