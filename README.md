# Shopify Checkout Events Pixel

## Overview
This repository provides a **custom pixel script** designed for **Shopify’s extensibility checkout**.  
The pixel listens to key lifecycle events — `checkout_started` and `checkout_completed` — and pushes them into **AB Tasty** for experimentation, personalization, attribution, and analytics workflows.  

By leveraging the AB Tasty cookie, the script ensures all payloads are tied to a **unique visitor ID (vid)** and their **active campaigns/variations**, enabling attribution of transactions back to experiments.

---

## Why This Pixel Matters
- Shopify’s checkout extensibility allows developers to capture **customer journey milestones** natively.  
- AB Tasty requires events to be passed in **batch ingestion format** to properly attribute experiments.  
- This pixel **bridges the gap**: listening to checkout events, enriching them with AB Tasty visitor/campaign context, and forwarding to the AB Tasty ingestion API.  

---

## How It Works
1. **Initialization**  
   - The script is wrapped in an IIFE to avoid polluting the global scope.  
   - Console logging is added for full observability in test and QA environments.  

2. **Cookie Extraction**  
   - Reads the `ABTasty` cookie.  
   - Extracts:
     - **Visitor ID (`vid`)** → AB Tasty unique visitor identifier.  
     - **Campaigns (`c`)** → Map of campaign IDs and variation IDs.  
   - Example extracted object:
     ```json
     {
       "vid": "visitor123",
       "campaigns": { "1001": "2001", "1002": "2002" }
     }
     ```

3. **Payload Construction**  
   - Builds a JSON payload conforming to AB Tasty’s **batch ingestion format**.  
   - Common fields:
     - `cid` → AB Tasty client/site ID  
     - `vid` → visitor ID  
     - `c` → campaigns map  
     - `dl` → current page URL  
     - `dr` → referrer URL  
     - `pt` → page title  
     - `cst` → timestamp  
   - Appends `EVENT` or `TRANSACTION` objects depending on the event type.  

4. **Payload Dispatch**  
   - Sends payloads to **`https://ariane.abtasty.com/`** using `XMLHttpRequest`.  
   - Includes status logging for success/failure.  

5. **Shopify Checkout Integration**  
   - Uses `analytics.subscribe` from Shopify Checkout Extensibility:  
     - `checkout_started` → triggers an **EVENT payload** with action `checkout_started`.  
     - `checkout_completed` → triggers a **TRANSACTION payload** with details like order ID, revenue, currency, item count, and shipping.  

---

## Code Flow

```mermaid
flowchart TD
    A[Shopify Checkout Events] --> B[Pixel Script]
    B --> C[Extract Visitor ID + Campaigns from ABTasty Cookie]
    C --> D[Construct AB Tasty Batch Payload]
    D --> E[Send Payload via XMLHttpRequest]
    E --> F[AB Tasty Ingestion Endpoint]
Payload Examples
checkout_started → EVENT
json
Copy code
{
  "cid": "647122547a691c3986656385348f326a",
  "vid": "visitor123",
  "c": { "1001": "2001" },
  "dl": "https://shop.com/checkout",
  "dr": "https://shop.com/cart",
  "pt": "Checkout",
  "cst": 1738210000000,
  "t": "BATCH",
  "h": [
    {
      "t": "EVENT",
      "ec": "Action Tracking",
      "ea": "checkout_started",
      "qt": 502
    }
  ]
}
checkout_completed → TRANSACTION
json
Copy code
{
  "cid": "647122547a691c3986656385348f326a",
  "vid": "visitor123",
  "c": { "1001": "2001" },
  "dl": "https://shop.com/checkout/thank_you",
  "dr": "https://shop.com/checkout",
  "pt": "Order Confirmation",
  "cst": 1738210050000,
  "t": "BATCH",
  "h": [
    {
      "t": "TRANSACTION",
      "tid": "order123",
      "ta": "Purchase",
      "tr": "99.99",
      "tc": "GBP",
      "ts": "4.99",
      "icn": 3,
      "qt": 503
    }
  ]
}
Key Functions Explained
Cookie Handling
safeGetCookie(name)
Retrieves cookies safely with error handling.

extractUidAndCampaignsFromCookie()
Parses the ABTasty cookie to extract vid and campaigns.

AB Tasty Data Fetcher
fetchABTastyData()
Wrapper that provides { vid, campaigns } or exits early if unavailable.

Payload Builder
constructPayload(vid, campaigns, eventType, eventDetails)
Creates a structured AB Tasty batch payload with contextual details.

Shopify Checkout Helpers
getTransactionId(checkout) → order ID

getTotalRevenue(checkout) → order total

getCurrency(checkout) → currency code

getItemCount(checkout) → number of items

getTransactionShipping(checkout) → shipping amount

Event Subscriptions
checkout_started → triggers an EVENT payload.

checkout_completed → triggers a TRANSACTION payload.

Installation
Clone the repository:

bash
Copy code
git clone https://github.com/<your-org>/shopify-checkout-events-pixel.git
cd shopify-checkout-events-pixel
Copy pixel.js into your Shopify app extension.

In Shopify Admin → Customer Events, register the pixel.

Deploy and validate by checking console logs and AB Tasty reporting.

Integration Notes
Requires AB Tasty cookie (ABTasty) to be present.

Payloads sent to https://ariane.abtasty.com/ in batch format.

Works with Shopify Checkout Extensibility only.

Designed to be extensible — more events can be added (cart updates, payment step, etc.).

Contributing
Contributions are welcome:

Open an issue for feature requests or bug reports.

PRs should include clear commit messages and test steps.

License
MIT License

yaml
Copy code

---

Would you like me to also add an **`examples/payloads.md` file** with multi
