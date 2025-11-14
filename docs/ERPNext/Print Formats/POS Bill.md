
bills

```html
<!-- ### FIXED 72mm THERMAL RECEIPT - HTML ONLY ### -->
{%- set company = frappe.get_doc("Company", doc.company) if doc.company else None -%}

<div class="receipt">
    <!-- === HEADER === -->
    <header class="receipt-header">
        {% if company and company.company_logo %}
            <img src="{{ company.company_logo }}" class="logo">
        {% endif %}
        <h2 class="company-name">{{ doc.company }}</h2>
        <p class="company-details">
            {% if company %}{{ company.get_formatted("address") }}{% endif %}<br>
            ☎ {% if company and company.phone_no %}{{ company.phone_no }}{% else %}N/A{% endif %}
            {% if company and company.gstin %}<br>GSTIN: {{ company.gstin }}{% endif %}
        </p>
    </header>

    <hr class="separator">

    <!-- === TRANSACTION INFO === -->
    <section class="info-section">
        <div class="info-item"><span>{{ _("Bill No") }}:</span> <span>{{ doc.name }}</span></div>
        <div class="info-item">
            <span>{{ _("Date") }}:</span>
            <span>{% if doc.posting_date %}{{ frappe.format_date(doc.posting_date, "dd/MM/yy") }}{% endif %}</span>
        </div>
        {% if doc.customer %}
        <div class="info-item"><span>{{ _("Customer") }}:</span> <span>{{ doc.customer_name }}</span></div>
        {% endif %}
    </section>

    <hr class="separator-heavy">

    <!-- === ITEMS BODY === -->
    <section class="items-body">
        <div class="items-header-row">
            <span class="item-name-header">{{ _("ITEM") }}</span>
            <span class="item-total-header">{{ _("TOTAL") }}</span>
        </div>

        {% for item in doc.items %}
        <div class="item-entry">
            <div class="item-name">
                {{ item.item_name }}
                {% set item_doc = frappe.get_doc("Item", item.item_code) if item.item_code else None %}
                {% if item_doc and item_doc.has_variant and item_doc.attributes %}
                    <div class="item-variants">
                    {% for attr in item_doc.attributes %}
                        <span>{{ attr.attribute }}: {{ item[frappe.scrub(attr.attribute)] or attr.attribute_value }}</span>
                    {% endfor %}
                    </div>
                {% endif %}
            </div>
            
            <!-- ### "Before Discount" price from custom_50_of_ssr field ### -->
            {% if item.custom_50_of_ssr and item.custom_50_of_ssr > 0 %}
            <div class="before-discount-price">
                <span>{{ _("Before Discount") }}:</span>
                <span>{{ frappe.format_value(item.custom_50_of_ssr, currency=doc.currency) }}</span>
            </div>
            {% endif %}

            <div class="item-calculation">
                <span>{{ frappe.format_value(item.qty) }} x {{ frappe.format_value(item.rate, currency=doc.currency) }}</span>
                <span class="line-total">{{ frappe.format_value(item.amount, currency=doc.currency) }}</span>
            </div>
            
            {% if item.discount_percentage %}
            <div class="discount-calculation">
                <span>{{ _("Discount") }} @ {{ item.discount_percentage }}%</span>
                <span class="line-total">- {{ frappe.format_value(item.discount_amount, currency=doc.currency) }}</span>
            </div>
            <div class="net-rate-calculation">
                <span>{{ _("Net Rate") }}: {{ frappe.format_value(item.net_rate, currency=doc.currency) }}</span>
            </div>
            {% endif %}
        </div>
        {% endfor %}
    </section>

    <hr class="separator-heavy">

    <!-- === TOTALS SECTION === -->
    <section class="totals-section">
        <div class="total-row">
            <span>{{ _("Subtotal") }}</span>
            <span>{{ frappe.format_value(doc.total, currency=doc.currency) }}</span>
        </div>
        {% if doc.discount_amount %}
        <div class="total-row">
            <span>{{ _("Total Discount") }}</span>
            <span>- {{ frappe.format_value(doc.discount_amount, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        {% if doc.taxes %}
        {% for tax in doc.taxes %}
        <div class="total-row tax-row">
            <span>{{ tax.description }}</span>
            <span>{{ frappe.format_value(tax.tax_amount, currency=doc.currency) }}</span>
        </div>
        {% endfor %}
        {% endif %}
        {% if doc.rounding_adjustment %}
        <div class="total-row">
            <span>{{ _("Rounding") }}</span>
            <span>{{ frappe.format_value(doc.rounding_adjustment, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        <div class="grand-total">
            <span>{{ _("GRAND TOTAL") }}</span>
            <span>{{ frappe.format_value(doc.grand_total, currency=doc.currency) }}</span>
        </div>
    </section>

    <hr class="separator">

    <!-- === PAYMENT SECTION === -->
    <section class="payment-section">
        <div class="section-header">{{ _("PAYMENT DETAILS") }}</div>
        {% if doc.payments %}
        {% for payment in doc.payments %}
            <div class="payment-row">
                <span>{{ payment.mode_of_payment }}:</span>
                <span>{{ frappe.format_value(payment.amount, currency=doc.currency) }}</span>
            </div>
        {% endfor %}
        {% set total_paid = doc.payments|sum(attribute='amount') %}
        {% if total_paid > doc.grand_total %}
        <div class="change-due">
            <span>{{ _("CHANGE DUE") }}:</span>
            <span>{{ frappe.format_value(total_paid - doc.grand_total, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        {% endif %}
    </section>

    <!-- === FOOTER === -->
    <footer class="receipt-footer">
        <p class="thank-you">{{ _("Thank you for your business!") }}</p>
        <p class="terms">
            {% if company and company.terms %}{{ company.terms }}{% else %}{{ _("Return/Exchange within 3 days with receipt.") }}{% endif %}
        </p>
        <div class="barcode-container">
            <svg id="barcode"></svg>
        </div>
        <div class="qrcode">
            <img src="/files/motitaka.svg" alt="Scan QR Code">
        </div>
        <div class="print-timestamp">
            {{ _("Printed") }}: {{ frappe.utils.now_datetime().strftime("%d/%m/%Y %I:%M %p") }}
        </div>
    </footer>
</div>

<!-- Barcode Generation Script -->
<script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.0/dist/JsBarcode.all.min.js"></script>
<script>
    document.addEventListener('DOMContentLoaded', function() {
        try {
            var billNumber = "{{ doc.name }}";
            if (billNumber && billNumber.trim() !== "") {
                JsBarcode("#barcode", billNumber, {
                    format: "CODE128",
                    displayValue: true,
                    width: 1.2,
                    height: 40,
                    margin: 0,
                    fontSize: 12,
                    textMargin: 2,
                    background: "#ffffff",
                    lineColor: "#000000"
                });
            }
        } catch (e) {
            console.error("Barcode generation failed:", e);
        }
    });
</script>
```


```css
/* --- Professional 72mm Thermal Receipt CSS --- */

/* Base Receipt Container */
.receipt {
    width: 72mm;
    max-width: 72mm;
    font-family: 'SF Mono', 'Courier New', monospace;
    font-size: 13px;
    color: #000;
    font-weight: bold;
    margin: 0 auto;
    box-sizing: border-box;
}

/* --- HEADER & FOOTER --- */
.receipt-header, 
.receipt-footer { 
    text-align: center; 
}

.logo { 
    max-width: 100%; 
    max-height: 60px; 
    margin-bottom: 5px; 
}

.company-name { 
    font-size: 20px; 
    margin: 0; 
}

.company-details { 
    font-size: 12px; 
    line-height: 1.4; 
    margin: 3px 0; 
}

.thank-you { 
    font-size: 15px; 
    margin-top: 10px; 
}

.terms { 
    font-size: 11px; 
}

/* --- SEPARATORS --- */
hr.separator { 
    border: none; 
    border-top: 1px dashed #000; 
    margin: 8px 0; 
}

hr.separator-heavy { 
    border: none; 
    border-top: 2px solid #000; 
    margin: 8px 0; 
}

/* --- TRANSACTION & PAYMENT INFO --- */
.info-section, 
.payment-section, 
.totals-section { 
    margin: 10px 0; 
}

.section-header { 
    text-align: center; 
    font-size: 15px; 
    margin-bottom: 8px; 
}

.info-item, 
.total-row, 
.payment-row { 
    display: flex; 
    justify-content: space-between; 
    margin-bottom: 3px; 
}

.change-due { 
    display: flex; 
    justify-content: space-between; 
    font-size: 15px; 
    margin-top: 8px; 
    padding-top: 5px; 
    border-top: 1px dashed #000; 
}

/* --- ITEMS SECTION --- */
.items-header-row { 
    display: flex; 
    justify-content: space-between; 
    font-size: 12px; 
    border-bottom: 1px solid #000; 
    padding-bottom: 4px; 
    margin-bottom: 4px; 
}

.item-name-header { 
    text-align: left; 
}

.item-total-header { 
    text-align: right; 
}

.item-entry { 
    margin-bottom: 10px; 
}

.item-name { 
    font-size: 14px; 
}

.item-variants { 
    font-size: 11px; 
    font-weight: normal; 
    margin-top: 2px; 
}

.item-calculation, 
.discount-calculation, 
.net-rate-calculation, 
.before-discount-price { 
    display: flex; 
    justify-content: space-between; 
    font-size: 13px; 
    padding-left: 10px; 
}

.net-rate-calculation { 
    justify-content: flex-end; 
}

.line-total { 
    text-align: right; 
}

.before-discount-price { 
    color: #333; 
    font-weight: normal; 
    font-style: italic; 
}

/* --- TOTALS --- */
.grand-total { 
    display: flex; 
    justify-content: space-between; 
    font-size: 18px; 
    border-top: 2px solid #000; 
    margin-top: 8px; 
    padding-top: 5px; 
}

/* --- BARCODE - FIXED FOR 72mm WIDTH --- */
.barcode-container { 
    text-align: center; 
    padding: 8px 0; 
    margin: 8px auto;
    max-width: 100%;
    overflow: hidden;
}

.barcode-container svg { 
    max-width: 100% !important; 
    width: 100% !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
}

/* --- QR CODE & TIMESTAMP --- */
.qrcode { 
    text-align: center; 
    margin-top: 10px; 
}

.qrcode img { 
    width: 160px; 
    height: 160px; 
}

.print-timestamp { 
    font-size: 11px; 
    text-align: center; 
    margin-top: 10px; 
    font-weight: normal; 
}

/* --- PRINT MEDIA OPTIMIZATION --- */
@media print {
    .receipt {
        width: 72mm;
        max-width: 72mm;
        margin: 0;
    }
    
    .barcode-container svg {
        max-width: 100% !important;
        width: 100% !important;
    }
}
```