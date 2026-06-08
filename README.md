# Odoo QWeb PDF Report Templates

This folder contains custom Odoo QWeb XML templates used for PDF reports and warehouse worksheets. These files are intended to be copied into Odoo as report views under the Technical menu, then connected to report actions so users can print them from the correct model.

## Files in This Folder

| File | QWeb Template Name | Purpose |
| --- | --- | --- |
| `route_load_detail_pdf_template.xml` | `route_load_detail_pdf_template` | Route Load Detail Report grouped by delivery date, route, and site. Shows total weight, cases, and units. |
| `route_load_summary_pdf_template.xml` | `sol_report_pdf_template` | Route Load Summary Report grouped by delivery date and route. Shows route totals and grand totals. |
| `warehouse_bulk_pick_template.xml` | `warehouse_bulk_pick_template` | Warehouse Bulk Pick Worksheet. Groups selected sales order lines by delivery date and product, then suggests inventory locations and lots. |
| `warehouse_receiving_template.xml` | `warehouse_receiving_worksheet_template` | Warehouse Receiving Worksheet for incoming stock receipts. Shows receipt details, vendor, PO number, products, barcodes, and receiving fields. |

## What These Templates Are

Each XML file is a QWeb report template. QWeb is Odoo's templating language for rendering HTML and PDF reports.

The template usually starts with:

```xml
<t t-name="your_template_name">
  <t t-call="web.html_container">
    ...
  </t>
</t>
```

The `t-name` value is important. It is the internal template name that the Odoo report action must reference when generating the PDF.

Most reports use the variable `docs`, which contains the records selected by the user or passed by the report action. For example:

```xml
<t t-if="not docs">
  <p>No records selected.</p>
</t>
```

## Current Report Models

These templates are designed around Odoo records such as:

| Report | Expected Model / Record Type |
| --- | --- |
| Route Load Detail Report | Sales Order Lines, commonly `sale.order.line` |
| Route Load Summary Report | Sales Order Lines, commonly `sale.order.line` |
| Warehouse Bulk Pick Worksheet | Sales Order Lines, commonly `sale.order.line` |
| Warehouse Receiving Worksheet | Stock Pickings / Receipts, commonly `stock.picking` |

Confirm the exact model in Odoo when creating the report action. The model must match the records that will be selected when printing the report.

## Creating a New Report in Odoo

Use these steps when creating a new report from one of these templates or when building a new report from scratch.

### 1. Enable Developer Mode

1. Open Odoo.
2. Go to `Settings`.
3. Enable Developer Mode.
4. Open the `Technical` menu.

Developer Mode is required because the report setup is done through technical records such as views, external identifiers, actions, and reports.

### 2. Create the QWeb View

1. Go to `Settings > Technical > User Interface > Views`.
2. Click `New`.
3. Fill in the view information:
   - `View Name`: A readable name, for example `Route Load Detail PDF Template`.
   - `View Type`: `QWeb`.
   - `Key`: The template name, for example `route_load_detail_pdf_template`.
   - `Architecture`: Paste the full XML template.
4. Save the view.

The `Key` should match the value used in the XML `t-name`.

Example:

```xml
<t t-name="route_load_detail_pdf_template">
```

The view key should be:

```text
route_load_detail_pdf_template
```

### 3. Create or Confirm the External Identifier

1. Open the QWeb view you just created.
2. From the debug menu, open `View Metadata`.
3. Confirm that the view has an External ID.
4. If needed, create an External Identifier under `Settings > Technical > Sequences & Identifiers > External Identifiers`.

Use a clear module and name. For example:

```text
Module: custom_reports
Name: route_load_detail_pdf_template
Complete External ID: custom_reports.route_load_detail_pdf_template
```

The complete External ID is what the report action references.

### 4. Create the Report Action

1. Go to `Settings > Technical > Actions > Reports`.
2. Click `New`.
3. Fill in the report action:
   - `Name`: The label users will see, for example `Route Load Detail Report`.
   - `Model`: The Odoo model the report prints from, for example `sale.order.line`.
   - `Report Type`: `QWeb PDF`.
   - `Report Name`: The External ID of the QWeb template, for example `custom_reports.route_load_detail_pdf_template`.
   - `Report File`: Usually the same value as `Report Name`.
   - `Print Report Name`: Optional Python expression for the downloaded PDF name.
4. Save the report action.

Important: `Report Name` must point to the QWeb template's complete External ID, not just the display name.

### 5. Add the Report to the Print Menu

Depending on the Odoo version and report configuration, the report action can appear automatically on the model's print menu after it is saved.

If it does not appear:

1. Confirm the report action uses the correct model.
2. Confirm `Report Type` is `QWeb PDF`.
3. Confirm the QWeb view has a valid External ID.
4. Confirm the report action's `Report Name` matches that External ID.
5. Refresh the browser or reload Odoo.

### 6. Test the Report

1. Go to the screen for the model used by the report.
2. Select one or more records.
3. Open the `Print` menu.
4. Choose the new report.
5. Confirm the PDF renders correctly.

Test with:

- No records selected, if the report can be called that way.
- One record.
- Multiple records.
- Records with missing optional values.
- Records with the custom fields used by the template.

## Checklist for Creating a New Template

Before creating another report, confirm these items:

- The XML has one top-level `<t t-name="...">`.
- The `t-name` is unique.
- The Odoo view key matches the `t-name`.
- The QWeb view has an External ID.
- The report action uses the correct model.
- The report action's `Report Name` points to the complete External ID.
- The template handles empty or missing values.
- The template has been tested with realistic records.

## Common Fields Used by These Reports

Some templates depend on custom fields. Confirm these exist in the target Odoo database before reusing the templates.

| Field | Used For |
| --- | --- |
| `x_studio_delivery_date` | Groups route and warehouse reports by delivery date. |
| `x_studio_route` | Groups sales order lines by route. |
| `x_studio_stop` | Sorts route detail lines by stop order. |
| `x_studio_line_total_weight` | Calculates total route and site weight. |
| `x_studio_mfg_code` | Shows manufacturer code on the receiving worksheet. |

If a field is missing or named differently in another database, update the XML before creating the QWeb view.

## Troubleshooting

### Report does not show in the Print menu

- Check that the report action model matches the model of the records you are viewing.
- Check that the report action is active.
- Check that the report type is `QWeb PDF`.
- Refresh the browser after saving the action.

### Odoo says the template cannot be found

- Confirm the QWeb view has an External ID.
- Confirm the report action's `Report Name` uses the complete External ID.
- Confirm the template `t-name`, view key, and External ID are not mixed up.

### PDF renders but data is missing

- Confirm the report action uses the correct model.
- Confirm the selected records contain the fields referenced in the XML.
- Check custom fields such as `x_studio_delivery_date`, `x_studio_route`, and `x_studio_line_total_weight`.

### PDF fails with a QWeb error

- Read the field name mentioned in the error.
- Confirm that field exists on the report model or related model.
- Check for missing values and add fallback logic such as `or ''` where needed.

## Naming Recommendation

Use consistent names when creating new reports:

```text
Template file: warehouse_example_template.xml
QWeb t-name: warehouse_example_template
View key: warehouse_example_template
External ID: custom_reports.warehouse_example_template
Report name: Warehouse Example Report
Report action Report Name: custom_reports.warehouse_example_template
```

Keeping these names aligned makes the report easier to find, troubleshoot, and recreate in another Odoo database.
