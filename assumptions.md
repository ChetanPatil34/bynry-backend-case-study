# Assumptions

While solving this case study, some requirements were intentionally left open-ended.
The following assumptions were made to proceed with a practical and realistic backend design.

---

## Business Assumptions

- A company can have multiple warehouses.
- A product can exist in multiple warehouses with different inventory quantities.
- Each product belongs to a single company.
- A product may or may not have an associated supplier.
- If multiple suppliers exist, one primary supplier is assumed for reordering.

---

## Inventory & Sales Assumptions

- Recent sales activity refers to sales within the last 30 days.
- Products without recent sales are excluded from low-stock alerts.
- Inventory quantity represents the current available stock.
- Zero-stock products are valid and should trigger alerts if applicable.

---

## Threshold & Alert Assumptions

- Low-stock threshold is stored at the product level.
- Thresholds may vary by product type.
- Alerts are generated per warehouse, not per product.
- Days until stockout are estimated using average daily sales.

---

## Technical Assumptions

- All operations are assumed to be handled within transactional boundaries.
- Code samples are illustrative and not production-ready.
- Proper indexing and foreign key constraints exist at the database level.
- Authentication and authorization are handled outside the scope of this case study.

---

## Notes

These assumptions were documented to ensure clarity and transparency in the absence of complete product requirements.
They reflect common practices in real-world backend systems.
