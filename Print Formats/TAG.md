```html
{% set company_name = doc.company or frappe.db.get_value("Global Defaults", None, "default_company") or "COMPANY" %}
{% set logo = frappe.get_all("File", filters={"attached_to_doctype": "Company", "attached_to_name": company_name}, fields=["file_url"], limit=1) %}
{% set barcode_value = (doc.barcodes and doc.barcodes[0].barcode) or "" %}
{% set show_size = doc.custom_size and (doc.item_group or "")|lower == "readymade suit" %}
{% set currency = doc.currency or frappe.db.get_value("Company", company_name, "default_currency") or "INR" %}
{% set mrp_price = doc.custom_50_of_ssr or 0 %}
{% set sale_price = doc.standard_rate or 0 %}

<div class="tag">

  <div class="tag-header{% if not (logo and logo|length > 0) %} no-logo{% endif %}">
    {% if logo and logo|length > 0 %}
      <img src="{{ logo[0].file_url }}" class="tag-logo" alt="{{ company_name }} Logo">
    {% endif %}
    <div class="tag-company">{{ company_name|upper }}</div>
  </div>

  {% if doc.brand %}
  <div class="tag-box tag-brand">{{ doc.brand|upper }}</div>
  {% endif %}

  <div class="tag-box tag-item">{{ (doc.item_name or "ITEM NAME")|upper }}</div>

  <div class="tag-row">
    {% if doc.custom_color %}
    <div class="tag-cell">
      <div class="cell-label">COLOR</div>
      <div class="cell-value">{{ doc.custom_color|upper }}</div>
    </div>
    {% endif %}

    {% if show_size %}
    <div class="tag-cell">
      <div class="cell-label">SIZE</div>
      <div class="cell-value">{{ doc.custom_size|upper }}</div>
    </div>
    {% endif %}
  </div>

  <div class="tag-row prices">
    <div class="tag-cell">
      <div class="cell-label">MRP</div>
      <div class="cell-value price-strike">{{ frappe.format_value(mrp_price, {"fieldtype":"Currency","options":currency}) }}</div>
    </div>
    <div class="tag-cell dark">
      <div class="cell-label">SALE 50%</div>
      <div class="cell-value">{{ frappe.format_value(sale_price, {"fieldtype":"Currency","options":currency}) }}</div>
    </div>
  </div>

  <div class="barcode">
    {% if barcode_value %}
      <svg id="tag-barcode" class="barcode-svg"></svg>
      <div class="barcode-text">{{ barcode_value }}</div>
    {% else %}
      <div class="no-barcode">NO BARCODE</div>
    {% endif %}
  </div>

  {% if doc.custom_encrypted_buying_price %}
  <div class="tag-footer">{{ doc.custom_encrypted_buying_price|upper }}</div>
  {% endif %}

</div>

<script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.12.1/dist/JsBarcode.all.min.js"></script>
<script>
(function(){
  if (typeof JsBarcode === "undefined") return;
  const value = "{{ barcode_value|e }}".trim();
  if (!value) return;
  const svg = document.getElementById("tag-barcode");
  if (!svg) return;

  JsBarcode(svg, value, {
    format: "CODE128",
    width: 1.4,
    height: 45,
    margin: 0,
    displayValue: false,
    background: "#FFFFFF",
    lineColor: "#000000"
  });

  svg.setAttribute("shape-rendering", "crispEdges");
  svg.style.width = "42mm";
  svg.style.height = "15mm";
})();
</script>
```

```css
@page {
  size: 50mm 100mm;
  margin: 0;
}

body {
  margin: 0;
  background: #ffffff;
  color: #000000;
  font-family: Arial, Helvetica, sans-serif;
  -webkit-print-color-adjust: exact;
  print-color-adjust: exact;
}

.tag {
  width: 50mm;
  height: 100mm;
  padding: 3mm 2mm;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  gap: 1.6mm;
}

/* Header */
.tag-header {
  border-bottom: 1px solid #000;
  padding: 0 0.6mm 1.1mm;
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 1.2mm;
  text-align: center;
}
.tag-logo {
  max-height: 9mm;
  max-width: 16mm;
  object-fit: contain;
  flex-shrink: 0;
}
.tag-company {
  font-weight: 900;
  font-size: 9.4pt;
  letter-spacing: 0.06em;
  flex: 1;
  text-align: center;
}
.tag-header.no-logo {
  justify-content: center;
}
.tag-header.no-logo .tag-company {
  flex: 0;
}

/* Generic boxes */
.tag-box {
  border: 1px solid #000;
  border-radius: 1mm;
  text-align: center;
  padding: 1.4mm;
  font-weight: 900;
  font-size: 8.6pt;
  letter-spacing: 0.04em;
}

.tag-brand {
  background: #000000;
  color: #ffffff;
}

.tag-item {
  font-size: 9.2pt;
  line-height: 1.18;
}

/* Attribute rows */
.tag-row {
  display: flex;
  gap: 1mm;
}

.tag-row.prices {
  margin-top: -0.2mm;
}

.tag-cell {
  flex: 1;
  border: 1px solid #000;
  border-radius: 1mm;
  padding: 1.1mm;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 0.7mm;
}

.cell-label {
  font-size: 5.6pt;
  font-weight: 800;
  letter-spacing: 0.24mm;
  color: #000000;
}

.cell-value {
  font-size: 8.6pt;
  font-weight: 900;
  line-height: 1.12;
}

.tag-cell.dark {
  background: #000000;
  color: #ffffff;
}

.tag-cell.dark .cell-label,
.tag-cell.dark .cell-value {
  color: #ffffff;
}

.price-strike {
  text-decoration: line-through;
  text-decoration-thickness: 0.35mm;
}

/* Barcode block */
.barcode {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding-top: 1.2mm;
  padding-bottom: 0.6mm;
  gap: 0.8mm;
}

.barcode-svg {
  width: 42mm;
  height: 15mm;
}

.barcode-text {
  font-size: 7.2pt;
  font-weight: 900;
  letter-spacing: 0.06em;
  color: #000000;
}

.no-barcode {
  font-size: 7.2pt;
  font-weight: 800;
  opacity: 0.55;
  color: #000000;
}

/* Footer */
.tag-footer {
  border-top: 1px dashed #000;
  padding: 1.2mm 1.2mm 1mm;
  margin-top: 1mm;
  background: #000000;
  color: #ffffff;
  font-size: 7.4pt;
  letter-spacing: 0.06em;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

@media print {
  .tag {
    page-break-inside: avoid;
  }
  * {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
}
```