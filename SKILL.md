---
name: invoice-generator
description: Generate and retrieve professional invoice documents from order data or order IDs using HTML Canvas. Use when the user asks to generate, create, produce, make, show, get, fetch, view, or display an invoice — for any order number, order ID, customer, or JSON data. Also use for converting invoices to PNG or image format, creating printable invoices, or batch generating multiple invoices. Trigger phrases include "generate invoice", "create invoice", "invoice for order", "get invoice", "show invoice", "make invoice", "invoice image", "invoice to PNG", "invoice from JSON", "produce invoice", "invoice document", "invoice for customer", "invoice number", "fetch invoice".
version: 1.1.0
---

# Invoice Generator Skill

## Overview
Generate professional invoice documents from order data or order IDs using HTML Canvas. Reads the workspace template, embeds CSS inline, injects the invoice data, and writes a ready-to-open HTML file that renders and downloads the invoice as a PNG. When an order ID is provided instead of raw JSON, fetch the invoice data from your configured backend or MCP before proceeding.

## ⚠️ CRITICAL IMPLEMENTATION NOTES

### 1. CSS Must Be Embedded Inline
**ALWAYS embed CSS inline!** The template uses `<link rel="stylesheet" href="invoice-styles.css">` which will NOT work because the CSS file is in the `templates/` subdirectory. You MUST:
1. Read both `templates/invoice-template.html` AND `templates/invoice-styles.css`
2. Replace the `<link>` tag with `<style>` tags containing the CSS content
3. This ensures proper centering and styling of the invoice

**Without embedded CSS, the invoice will not be centered and styling will be broken.**

### 2. File Naming Convention
**Output filename MUST be the order ID only** (e.g., `order10000.html`)
- ✅ Correct: `order10000.html`
- ❌ Wrong: `invoice-INV-20260611-7401.html`

### 3. For Getting Calculated JSON Data
- If this is a **new order** (raw line-item data provided), call your backend's order-creation/enrichment endpoint to compute totals, apply discounts, and calculate taxes before generating the invoice.
- Save the enriched JSON to your orders store if persistence is required.
- The input JSON must follow the structure described in the **JSON Data Structure** section below — no external API reference file is needed.

### 4. If Only Order ID Is Provided
- Fetch the invoice JSON from your configured backend or MCP tool (e.g., a `getInvoiceByOrderId` operation).
- Pass the order ID as required by your backend (commonly as a path or query parameter).
- The response may return the invoice JSON as an escaped string inside a wrapper field (e.g., `message`). Parse and normalise it before injecting it into the template.

**Example MCP call pattern:**
```json
{
  "path": {
    "orderId": 10002
  }
}
```

### 5. Seller / Billing Sections Must Always Render
The invoice must include both:
- a **FROM** section using seller details
- a **BILL TO** section using customer details

If seller data is not provided in the invoice JSON, use defaults:
- Seller name: `Example Enterprises Ltd`
- Address line 1: `123 Business Street`
- Address line 2: `City, Country`
- Email: `billing@example.com`

## When to Use This Skill
Activate this skill when the user needs to:
- Generate or retrieve invoice documents from an order ID or order number
- Generate invoice images from JSON data
- Create printable invoices programmatically
- Convert invoice data to image format
- Produce professional invoice documents
- Batch generate multiple invoices
- Show, fetch, view, or display an invoice for a customer or order

**Trigger phrases:**
- "generate invoice"
- "generate invoice for order"
- "invoice for order [number]"
- "get invoice"
- "show invoice"
- "fetch invoice"
- "view invoice"
- "create invoice"
- "produce invoice"
- "make invoice"
- "generate invoice image"
- "create invoice from JSON"
- "invoice generator"
- "make invoice image"
- "convert invoice to image"
- "invoice to PNG"
- "create invoice document"
- "invoice for customer"

## Capabilities
- Generate invoice images from structured JSON data
- Support for customer information and contact details
- Support seller/from details with sensible defaults
- Line item tables with product details, quantities, and prices
- Automatic calculation display (subtotal, discounts, tax, total)
- Professional formatting with borders and styling
- Export to PNG format
- Customizable colors and fonts
- Responsive canvas sizing
- Currency-aware formatting

## JSON Data Structure

The skill expects invoice data in this format:

```json
{
  "invoiceId": "INV-2001",
  "orderId": 10002,
  "customer": {
    "name": "ABC Corp",
    "email": "billing@abccorp.com"
  },
  "seller": {
    "name": "Example Enterprises Ltd",
    "addressLine1": "123 Business Street",
    "addressLine2": "City, Country",
    "email": "billing@example.com"
  },
  "items": [
    {
      "productId": "P101",
      "productName": "Enterprise Software License",
      "quantity": 2,
      "unitPrice": 45000,
      "lineTotal": 90000
    },
    {
      "productId": "P102",
      "productName": "Professional Services Package",
      "quantity": 1,
      "unitPrice": 25000,
      "lineTotal": 25000
    }
  ],
  "subtotal": 115000,
  "invoiceDiscountPercent": 10,
  "invoiceDiscountAmount": 11500,
  "discountedAmount": 103500,
  "gstPercent": 18,
  "gstAmount": 18630,
  "totalAmount": 122130,
  "generatedDate": "2026-06-02T11:54:09+05:30",
  "currency": "USD",
  "locale": "en-US",
}
```

### Required Fields
- `customer`: Object with `name` and `email`
- `items`: Array of line items with:
  - `productId`: Product identifier
  - `productName`: Product description
  - `quantity`: Number of units
  - `unitPrice`: Price per unit
  - `lineTotal`: Total for this line item
- `subtotal`: Sum of all line items
- `invoiceDiscountPercent`: Discount percentage applied
- `invoiceDiscountAmount`: Calculated discount amount
- `discountedAmount`: Subtotal after discount
- `gstPercent`: Tax percentage
- `gstAmount`: Calculated tax amount
- `totalAmount`: Final total amount
- `generatedDate`: Invoice generation date (ISO 8601 format)

### Optional Fields
- `invoiceId`: Unique invoice identifier
- `orderId`: Order identifier used as fallback display/filename reference
- `seller`: Seller/from block details
- `currency`: Currency code such as `USD` or `INR`
- `locale`: Locale such as `en-US` or `en-IN`
- `currencySymbol`: Currency symbol override such as `$` or `₹`

## How It Works

### Step 1: Validate JSON Data
Ensure all required fields are present and properly formatted:
- Customer has name and email
- Items array is not empty
- All numeric fields are valid numbers
- Date is in ISO 8601 format
- Currency metadata is present when non-default formatting is needed

### Step 2: Load Template
Read the invoice template from `templates/invoice-template.html`

**IMPORTANT:** The template uses an external CSS link (`<link rel="stylesheet" href="invoice-styles.css">`). You MUST replace this with embedded inline CSS from `templates/invoice-styles.css` to ensure proper styling and centering. The CSS file is in a subdirectory and won't load correctly with a relative path.

### Step 3: Embed CSS
Read the CSS file and embed it inline:
```javascript
const css = readFile('templates/invoice-styles.css');
html = html.replace('<link rel="stylesheet" href="invoice-styles.css">', `<style>${css}</style>`);
```

### Step 4: Inject Data
Replace the placeholder JSON in the template with escaped JSON text:
```javascript
const invoiceData = JSON.stringify(userData)
  .replace(/\\/g, '\\\\')
  .replace(/`/g, '\\`');
html = html.replace('{{INVOICE_DATA}}', invoiceData);
```

The template parses it with:
```javascript
const invoiceData = JSON.parse(`{{INVOICE_DATA}}`);
```

### Step 5: Generate Image
The template automatically:
- Renders the invoice on HTML Canvas
- Formats currency values using invoice currency metadata
- Displays seller and billing sections
- Displays all line items in a table
- Shows discount and tax calculations
- Provides download button
- Exports as PNG

### Step 6: Deliver Result
Provide the user with:
- Generated HTML file (can be opened in browser)
- Instructions to download the image

## Implementation Guide

### Using the Template

1. **Read the template and CSS files:**
   ```javascript
   const template = readFile('templates/invoice-template.html');
   const css = readFile('templates/invoice-styles.css');
   ```

2. **Embed CSS inline (CRITICAL STEP):**
   ```javascript
   let html = template.replace('<link rel="stylesheet" href="invoice-styles.css">', `<style>${css}</style>`);
   ```

3. **Prepare invoice data:**
   ```javascript
   const invoiceData = {
     orderId: 10002,
     customer: { name: "Global Enterprises", email: "payments@globalent.com" },
     currency: "USD",
     locale: "en-US",
     currencySymbol: "$"
   };
   ```

4. **Inject data into template safely:**
   ```javascript
   const serialized = JSON.stringify(invoiceData)
     .replace(/\\/g, '\\\\')
     .replace(/`/g, '\\`');
   html = html.replace('{{INVOICE_DATA}}', serialized);
   ```

5. **Write output file with correct naming:**
   ```javascript
   const filename = `order${invoiceData.orderId}.html`;
   writeFile(filename, html);
   ```

6. **Instruct user:**
   - Open the HTML file in a browser
   - Click "Download Invoice" button
   - Save the PNG image

## Code Reference

### Main Template
Location: `templates/invoice-template.html`
- Complete HTML Canvas implementation
- Automatic rendering on page load
- Download functionality included
- Responsive design
- Currency-aware formatting
- Seller/from and bill-to sections
- Uses fallback invoice number from order ID
- Uses external CSS link that MUST be replaced with inline CSS

### CSS Styles
Location: `templates/invoice-styles.css`
- Modern minimalist design
- Proper centering for canvas and metadata panels
- Responsive breakpoints
- Print-friendly styles
- MUST be embedded inline in the HTML file for proper rendering

### Example Data
Location: `examples/sample-invoice.json`
- Reference JSON structure
- All supported fields
- Formatting examples

## Customization Options

The template supports customization of:
- **Colors**: Header text, borders, table fills
- **Fonts**: Font family, sizes, weights
- **Layout**: Canvas dimensions, spacing, margins
- **Currency**: Controlled by invoice JSON
- **Company Info**: Seller block can be overridden per invoice
- **Styling**: Border styles, table formatting

## Best Practices

1. **CSS Embedding**: Always embed CSS inline by replacing the `<link>` tag with `<style>` tags
2. **File Naming**: Use order ID only as filename when the request is order-based, e.g. `order10002.html`
3. **Data Validation**: Always validate JSON structure before generation
4. **Number Formatting**: Ensure amounts are properly formatted with 2 decimal places
5. **Currency Display**: Use invoice-provided currency metadata, do not assume INR
6. **Date Formatting**: Convert ISO date to readable format
7. **Text Overflow**: Wrap or truncate long product names in the canvas table
8. **Canvas Size**: Default 900x1200px, adjustable
9. **Billing Blocks**: Always render both FROM and BILL TO sections

## Example Usage

**User Request:**
"hey give me invoice for order 10002"

**Agent Response Flow:**
1. Call your backend/MCP `getInvoiceByOrderId` with the order ID
2. Parse the returned invoice JSON (unwrap from any wrapper field if needed)
3. Add currency metadata if missing
4. Read `templates/invoice-template.html`
5. Read `templates/invoice-styles.css`
6. Replace the external CSS link with embedded inline CSS
7. Inject the invoice data into the template
8. Write the output HTML file using order ID as filename, e.g. `order10002.html`
9. Tell the user to open the file and download the PNG if needed

## Output Format
The skill generates:
- **HTML file**: Can be opened in any browser
- **PNG image**: Downloaded via browser
- **High quality**: Suitable for printing and digital use
- **Professional layout**: Clean, organized invoice format

## Troubleshooting

### Issue: Invoice Not Centered / Styling Missing
**Solution:** The CSS file must be embedded inline. Read `templates/invoice-styles.css` and replace the `<link>` tag with `<style>` tags containing the CSS content.

### Issue: Text Overlapping
**Solution:** Wrap long product names and rebalance column widths in the canvas layout.

### Issue: Items Not Showing
**Solution:** Verify items array structure matches expected format.

### Issue: Currency Symbol Wrong
**Solution:** Set `currency`, `locale`, and `currencySymbol` in the invoice JSON.

### Issue: Invoice Number Missing
**Solution:** Fall back to `ORDER-{orderId}` when `invoiceId` is absent.

### Issue: Seller Details Missing
**Solution:** Use default seller values when `seller` is not provided.

### Issue: Download Not Working
**Solution:** Check browser permissions for file downloads.

## Advanced Features

### Batch Generation
For multiple invoices:
1. Loop through array of invoice JSON objects
2. Generate each invoice sequentially
3. Name files using order IDs or invoice IDs based on request context
4. Collect all generated files

### Custom Branding
To add company information:
1. Add a `seller` object to the invoice JSON
2. Override default seller values
3. Template will render company details in header section

### PDF Export
For PDF output:
1. Use the generated PNG
2. Convert using libraries like jsPDF
3. Or use browser print-to-PDF functionality

## Limitations
- Requires browser to render and download
- Canvas size limited by browser memory
- No automatic PDF generation
- Single-page invoices only
- No interactive elements in final image

## Related Skills
- **pdf**: For PDF manipulation and generation
- **docx**: For Word document invoice templates
- **xlsx**: For spreadsheet-based invoicing

## Version
1.1.0 - Added currency-aware formatting, seller defaults, order-based retrieval guidance, and improved layout rules