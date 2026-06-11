---
name: invoice-generator
description: Generate professional invoice images from JSON data using HTML Canvas. Creates pixel-perfect invoice layouts with line items, discounts, GST calculations, and exports to PNG format.
version: 1.0.0
---

# Invoice Generator Skill

## Overview
Generate professional invoice images from JSON data using HTML Canvas. Creates pixel-perfect invoice layouts that can be downloaded as PNG images.

## ⚠️ CRITICAL IMPLEMENTATION NOTES

### 1. CSS Must Be Embedded Inline
**ALWAYS embed CSS inline!** The template uses `<link rel="stylesheet" href="invoice-styles.css">` which will NOT work because the CSS file is in the `templates/` subdirectory. You MUST:
1. Read both `templates/invoice-template.html` AND `templates/invoice-styles.css`
2. Replace the `<link>` tag with `<style>` tags containing the CSS content
3. This ensures proper centering and styling of the invoice

**Without embedded CSS, the invoice will not be centered and styling will be broken.**

### 2. File Naming Convention
**Output filename MUST be the invoice ID only** (e.g., `INV-20260611-7401.html`), NOT `invoice-INV-20260611-7401.html`.
- ✅ Correct: `INV-20260611-7401.html`
- ❌ Wrong: `invoice-INV-20260611-7401.html`

## When to Use This Skill
Activate this skill when the user needs to:
- Generate invoice images from JSON data
- Create printable invoices programmatically
- Convert invoice data to image format
- Produce professional invoice documents
- Batch generate multiple invoices

**Trigger phrases:**
- "generate invoice image"
- "create invoice from JSON"
- "invoice generator"
- "make invoice image"
- "convert invoice to image"
- "invoice to PNG"
- "create invoice document"

## Capabilities
- Generate invoice images from structured JSON data
- Support for customer information and contact details
- Line item tables with product details, quantities, and prices
- Automatic calculation display (subtotal, discounts, GST, total)
- Professional formatting with borders and styling
- Export to PNG format
- Customizable colors and fonts
- Responsive canvas sizing

## JSON Data Structure

The skill expects invoice data in this format:

```json
{
  "invoiceId": "INV-2001",
  "customer": {
    "name": "ABC Corp",
    "email": "billing@abccorp.com"
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
  "generatedDate": "2026-06-02T11:54:09+05:30"
}
```

### Required Fields
- `invoiceId`: Unique invoice identifier
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
- `gstPercent`: GST/tax percentage
- `gstAmount`: Calculated GST amount
- `totalAmount`: Final total amount
- `generatedDate`: Invoice generation date (ISO 8601 format)

## How It Works

### Step 1: Validate JSON Data
Ensure all required fields are present and properly formatted:
- Invoice ID is a string
- Customer has name and email
- Items array is not empty
- All numeric fields are valid numbers
- Date is in ISO 8601 format

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
Replace the placeholder JSON in the template with user's data:
```javascript
const invoiceData = JSON.stringify(userData, null, 2);
html = html.replace('{{INVOICE_DATA}}', invoiceData);
```

### Step 5: Generate Image
The template automatically:
- Renders the invoice on HTML Canvas
- Formats currency values (₹ symbol for Indian Rupees)
- Displays all line items in a table
- Shows discount and GST calculations
- Provides download button
- Exports as PNG

### Step 6: Deliver Result
Provide the user with:
- Generated HTML file (can be opened in browser)
- Instructions to download the image
- Option to customize further

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
   This ensures proper styling and centering since the CSS file is in a subdirectory.

3. **Prepare invoice data:**
   ```javascript
   const invoiceData = {
     invoiceId: "INV-2001",
     customer: { name: "ABC Corp", email: "billing@abccorp.com" },
     // ... rest of data
   };
   ```

4. **Inject data into template:**
   ```javascript
   html = html.replace('{{INVOICE_DATA}}', JSON.stringify(invoiceData, null, 2));
   ```

5. **Write output file with correct naming:**
   ```javascript
   // Use invoice ID as filename (NOT "invoice-" prefix)
   const filename = `${invoiceData.invoiceId}.html`;
   writeFile(filename, html);
   ```
   **Example:** For invoice ID "INV-20260611-7401", the filename should be `INV-20260611-7401.html`

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
- Indian Rupee (₹) currency formatting
- GST calculation display
- **NOTE:** Uses external CSS link that MUST be replaced with inline CSS

### CSS Styles
Location: `templates/invoice-styles.css`
- Modern minimalist design
- Flexbox centering for proper layout
- Responsive breakpoints
- Print-friendly styles
- **MUST be embedded inline in the HTML file for proper rendering**

### Example Data
Location: `examples/sample-invoice.json`
- Reference JSON structure
- All supported fields
- Formatting examples

## Customization Options

The template supports customization of:
- **Colors**: Header background, text colors, borders
- **Fonts**: Font family, sizes, weights
- **Layout**: Canvas dimensions, spacing, margins
- **Currency**: Default is ₹ (INR), can be changed
- **Company Info**: Add company name, logo, address
- **Styling**: Border styles, table formatting

## Best Practices

1. **CSS Embedding**: ALWAYS embed CSS inline by replacing the `<link>` tag with `<style>` tags
2. **File Naming**: Use invoice ID ONLY as filename (e.g., `INV-2001.html`), NOT `invoice-INV-2001.html`
3. **Data Validation**: Always validate JSON structure before generation
4. **Number Formatting**: Ensure amounts are properly formatted (2 decimal places)
5. **Currency Display**: Use ₹ symbol for Indian Rupees
6. **Date Formatting**: Convert ISO date to readable format (DD/MM/YYYY)
7. **Text Overflow**: Long product names may need truncation or wrapping
8. **Canvas Size**: Default 900x1200px, adjustable
9. **PNG Download**: The downloaded PNG will use the invoice ID as filename (e.g., `invoice-INV-2001.png`)

## Example Usage

**User Request:**
"Generate an invoice image from this JSON data: [provides JSON]"

**Agent Response:**
1. Validate the JSON structure
2. Read the invoice template from `templates/invoice-template.html`
3. Read the CSS file from `templates/invoice-styles.css`
4. **Replace the external CSS link with embedded inline CSS** (critical for proper centering)
5. Inject the invoice data into the template
6. Write the output HTML file using invoice ID as filename (e.g., `INV-2001.html`)
7. Provide instructions:
   - "I've created `INV-2001.html`" (use invoice ID, not "invoice-" prefix)
   - "Open it in your browser"
   - "Click 'Download Invoice' to save as PNG"

## Output Format
The skill generates:
- **HTML file**: Can be opened in any browser
- **PNG image**: Downloaded via browser (800x1100px default)
- **High quality**: Suitable for printing and digital use
- **Professional layout**: Clean, organized invoice format

## Troubleshooting

### Issue: Invoice Not Centered / Styling Missing
**Solution**: The CSS file must be embedded inline. Read `templates/invoice-styles.css` and replace the `<link>` tag with `<style>` tags containing the CSS content. This is the most common issue.

### Issue: Text Overlapping
**Solution**: Adjust line height or font size in template

### Issue: Items Not Showing
**Solution**: Verify items array structure matches expected format

### Issue: Currency Symbol Wrong
**Solution**: Template uses ₹ by default; modify template for other currencies

### Issue: Date Format Incorrect
**Solution**: Ensure date is in ISO 8601 format; template converts to DD/MM/YYYY

### Issue: Download Not Working
**Solution**: Check browser permissions for file downloads

## Advanced Features

### Batch Generation
For multiple invoices:
1. Loop through array of invoice JSON objects
2. Generate each invoice sequentially
3. Name files using invoice IDs
4. Collect all generated files

### Custom Branding
To add company information:
1. Modify template to include company section
2. Add company data to JSON structure
3. Template will render company details in header

### PDF Export
For PDF output:
1. Use the generated PNG
2. Convert using libraries like jsPDF
3. Or use browser print-to-PDF functionality

## Limitations
- Requires browser to render and download
- Canvas size limited by browser memory
- No automatic PDF generation (PNG only)
- Single-page invoices only
- No interactive elements in final image

## Related Skills
- **pdf**: For PDF manipulation and generation
- **docx**: For Word document invoice templates
- **xlsx**: For spreadsheet-based invoicing

## Version
1.0.0 - Initial release