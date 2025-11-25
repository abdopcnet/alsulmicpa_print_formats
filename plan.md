## Print Format Error Fix - ZATCA Sales Invoice

- Issue: Error in print format on line 6: Unknown column `qr_code_image` in SELECT.
- Root cause:
  - `docs/title.html` was selecting fields from `Sales Invoice Additional Fields` including:
    - `qr_code_image` (is_virtual=1) → not a DB column.
    - `signed_qr_code` (does not exist).
    - `uuid1` (typo; actual field is `uuid`).
  - Attempting to select virtual/missing fields triggered MySQL 1054.
- Verified schemas:
  - `Sales Invoice Additional Fields` has real columns like `invoice_type_transaction`, `invoice_counter`, `uuid`. `qr_code_image` is virtual; `qr_image_src` is virtual.
  - `Sales Invoice` customizations include `custom_qr_code_image` (Image) and `custom_qr_image_src` (Long Text).

### Change Applied
- In `print_format/zatca_sales_invoice/docs/title.html`, updated `frappe.db.get_value` to only fetch existing DB-backed fields:
  - From: `["invoice_type_transaction", "qr_code_image", "signed_qr_code", "invoice_counter", "uuid1"]`
  - To: `["invoice_type_transaction", "invoice_counter", "uuid"]`

### Notes
- `docs/qr.html` checks `doc.custom_qr_code_image` and uses `inv_additional_fields.qr_image_src`. Ensure the calling context sets `inv_additional_fields` appropriately; otherwise, adapt to use `details.siaf` or fetch `qr_image_src` safely if needed.

### Next
- Test printing the ZATCA format again to confirm the error is resolved.


