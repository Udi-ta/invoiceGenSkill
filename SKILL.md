---
name: invoice-generator
description: Generate professional invoice images from JSON data using HTML Canvas. Creates pixel-perfect invoice layouts with line items, discounts, GST calculations, and exports to PNG format.
version: 1.0.0
---

# Invoice Generator Skill

## Overview
Generate professional invoice images from JSON data using HTML Canvas. Creates pixel-perfect invoice layouts that can be downloaded as PNG images.

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

### Step 3: Inject Data
Replace the placeholder JSON in the template with user's data:
```javascript
const invoiceData = JSON.stringify(userData, null, 2);
html = html.replace('{{INVOICE_DATA}}', invoiceData);
```

### Step 4: Generate Image
The template automatically:
- Renders the invoice on HTML Canvas
- Formats currency values (₹ symbol for Indian Rupees)
- Displays all line items in a table
- Shows discount and GST calculations
- Provides download button
- Exports as PNG

### Step 5: Deliver Result
Provide the user with:
- Generated HTML file (can be opened in browser)
- Instructions to download the image
- Option to customize further

## Implementation Guide

### Using the Template

1. **Read the template file:**
   ```javascript
   const template = readFile('templates/invoice-template.html');
   ```

2. **Prepare invoice data:**
   ```javascript
   const invoiceData = {
     invoiceId: "INV-2001",
     customer: { name: "ABC Corp", email: "billing@abccorp.com" },
     // ... rest of data
   };
   ```

3. **Inject data into template:**
   ```javascript
   const html = template.replace('{{INVOICE_DATA}}', JSON.stringify(invoiceData, null, 2));
   ```

4. **Write output file:**
   ```javascript
   writeFile('invoice-output.html', html);
   ```

5. **Instruct user:**
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

1. **Data Validation**: Always validate JSON structure before generation
2. **Number Formatting**: Ensure amounts are properly formatted (2 decimal places)
3. **Currency Display**: Use ₹ symbol for Indian Rupees
4. **Date Formatting**: Convert ISO date to readable format (DD/MM/YYYY)
5. **Text Overflow**: Long product names may need truncation or wrapping
6. **Canvas Size**: Default 800x1100px (A4 ratio), adjustable
7. **File Naming**: Use invoice ID in filename (e.g., `invoice-INV-2001.png`)

## Example Usage

**User Request:**
"Generate an invoice image from this JSON data: [provides JSON]"

**Agent Response:**
1. Validate the JSON structure
2. Read the invoice template from `templates/invoice-template.html`
3. Inject the invoice data into the template
4. Write the output HTML file
5. Provide instructions:
   - "I've created `invoice-INV-2001.html`"
   - "Open it in your browser"
   - "Click 'Download Invoice' to save as PNG"

## Output Format
The skill generates:
- **HTML file**: Can be opened in any browser
- **PNG image**: Downloaded via browser (800x1100px default)
- **High quality**: Suitable for printing and digital use
- **Professional layout**: Clean, organized invoice format

## Troubleshooting

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