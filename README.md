# quickinvoicegen-api

## PDF Invoice Generation API

A simple API to generate PDF invoices. At this time the API supports only one basic, but customizable template.  The API is called via a POST request and supports both JSON as well as form encoded requests. 

**Supported languages:** English, Russian

**Supported currencies:**  USD,  CAD,  AUD,  GBP and EUR.

### Request endpoint

The API is accessed via send a POST request to the  [https://quickinvoicegen.com/pdf](https://quickinvoicegen.com/pdf) endpoint. Client can send an unlimited number of requests, however requests are rate limited. The endpoint will return `403` HTTP error if the rate limit is breached.

The API doesn't store any information related to the request on the server. 


### Request parameters

Mandatory fields are denoted by the exclamation mark (!) at the end of a field or a block name.
```json
{
  "from!" : {
    "name!": "The name of the invoice issuer",
    "details": "Address and other details of the invoice issuer. Lines are separated by the new line (\n) character",
  },
  "bill_to!" : {
    "name!": "The name of the invoice recepient",
    "details": "Address and other details of the invoice recepient. Lines are separated by the new line (\n) character",
  },
  "ship_to" : {
    "name!": "The name of good recepient",
    "details": "Address and other details of the good recepient. Lines are separated by the new line (\n) character",
	},
  "invoice_number!": "Alphanumeric representation of the invoice number",
  "invoice_date": "Date of the invoice in the YYYY-MM-DD format. Default: current date",
  "due_date": "Date when the invoice is due in the YYYY-MM-DD format. Default: no default",
  "due_days": "Number of days to use in automatic calculation of the invoice due date. Default: no default",
  "locale": "Language of the invoice. Default: en. Valid values: en, ru",
  "currency": "Currency of the invoice. Default: USD. Valid values: USD, CAD, AUD, EUR, GBP",
  "logo": "URL of the logo to use in the invoice",
  "items!": [
    {
      "description!": "Description of the item being invoiced.",
      "quantity!": "Number of units",
      "price!": "Price of a single unit",
      "taxes": "Overrides for taxes for the item. See the Taxes section"
    }
  ],

  "taxes": [
    {
      "name!": "Short name of the tax (i.e. VAT, GST, etc)",
      "rate!": "Tax rate",
      "invoice_text": "Name of the tax in the totals section of the invoice"
    }
  ],
  "custom_fields": [
    {
      "title!": "Title of the custom field",
      "value!": "Value of the custom field"
    }
  ],
  "notes": "Invoice notes",
  "gross_price": "Prices are gross prices (tax inclusive). Default: false. Valid values: true, false",
  "net_columns": "If prices are gross prices, enable net amount and tax colums in the invoice. Default: false. Valid values: true, false",
  "page_size": "Page size to use for the invoice. Default: Letter. Valid values: Letter, A4",
  "text": 
    [
     "List of overrides for text labels in the template. Described in the Custom Labels section below"
    ]
}
```
### Basic Example

```
curl \
-X POST "https://quickinvoicegen.com/pdf" \
-d from[name]="NASA" \
-d logo="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e5/NASA_logo.svg/1200px-NASA_logo.svg.png" \
-d from[details]="300 E St SW,%0AWashington, DC%0A20546, United States" \
-d bill_to[name]="White House" \
-d invoice_number="202010-1" \
-d items[0][description]="Useless but fun stuff from Mars" \
-d items[0][quantity]=1 \
-d items[0][price]=5000000 \
-o basic.pdf
```

### Taxes

The template supports multiple taxes. The list of taxes is provided using the  `taxes` parameter which is an array of objects describing individual taxes:

```json
{
  "name": "VAT",
  "rate": 20
}
```

If the name of the tax needs to be overridden in the totals section of the invoice, the `invoice_text` parameter can be used:

```json
{
  "name": "VAT",
  "rate": 20,
  "invoice_text": "VAT (UK1234567890)",
}
```

#### Example of an invoice with a tax

```
curl \
-X POST "https://quickinvoicegen.com/pdf" \
-d currency="CAD" \
-d from[name]="A Canadian Company" \
-d bill_to[name]="A Canadian Client" \
-d invoice_number="202010-1" \
-d items[0][description]="Subscription to a service" \
-d items[0][quantity]=1 \
-d items[0][price]=11.99 \
-d taxes[0][name]="HST" \
-d taxes[0][rate]=13 \
-o simple_taxes.pdf
```

In addition every single tax can be either overridden or disabled by the `taxes` parameter inside the each individual `item` block. The parameter is a hash where the key is a name of the tax found in the main `taxes` block, and the value is either `false` or the rate that applies to the item:

```json
{
  "taxes": [ 
	{
	  "name": "GST",
	  "rate": 6
	},
	{
	  "name": "PST",
	  "rate": 8
	}
  ],
  "items": [
     {
       "description": "An item without PST",
       "taxes": { "PST": "false"}
     }
  ]
}
```

#### Example of an invoice with two taxes where only one tax is charged to one of the items
```
curl \
-X POST "https://quickinvoicegen.com/pdf" \
-d currency="CAD" \
-d from[name]="A Canadian Company" \
-d bill_to[name]="A Canadian Client" \
-d invoice_number="202010-1" \
-d items[0][description]="An item with both GST and PST" \
-d items[0][quantity]=1 \
-d items[0][price]=10 \
-d items[1][description]="An item with just a GST" \
-d items[1][quantity]=1 \
-d items[1][price]=20 \
-d items[1][taxes][PST]=false \
-d taxes[0][name]="GST" \
-d taxes[0][rate]=5 \
-d taxes[1][name]="PST" \
-d taxes[1][rate]=8 \
-o multiple_taxes2.pdf
```

### Custom Fields

It is possible to provide a list of  additional fields that will be displayed in the rightmost column of the template in the `custom_fields` parameter.

```
 ...
 "custom_fields"
 ...
```

### Custom Labels

All the labels in the template can be customized via the `text` parameter. The `text` is hash where the key is the label being customized and value is the new text for the label. The valid labels are:

- invoice
- bill_to
- ship_to
- invoice_number
- invoice_date
- due_date
- description 
- quantity
- price
- amount
- amount_net
- total
- tax
- subtotal
- discount
- shipping

#### Example
Override the `description` label:

```
curl \
-X POST "https://quickinvoicegen.com/pdf" \
-d from[name]="NASA" \
-d logo="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e5/NASA_logo.svg/1200px-NASA_logo.svg.png" \
-d from[details]="300 E St SW,%0AWashington, DC%0A20546, United States" \
-d bill_to[name]="Quick Invoice Generator Developers" \
-d invoice_number="202010-1" \
-d items[0][description]="One way ticket to the Moon" \
-d items[0][quantity]=1 \
-d items[0][price]=0.99 \
-d text[invoice]="The Invoice" \
-o invoice_label_override.pdf

```

[https://github.com/srsem/quickinvoicegen-api](https://github.com/ssemenyuk-dev/quickinvoicegen-api)
