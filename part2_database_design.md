# Part 2: Database Design

## Objective

The goal of this database design is to support a scalable B2B inventory management system where:
- Companies can manage multiple warehouses
- Products can exist in multiple warehouses with different quantities
- Inventory changes are tracked over time
- Suppliers provide products
- Some products can be bundles of other products

The design focuses on normalization, scalability, and data integrity.

---

## Proposed Database Schema

### 1. companies
Stores information about each company using the platform.

- id (PK)
- name
- created_at

---

### 2. warehouses
Represents physical or logical storage locations for a company.

- id (PK)
- company_id (FK → companies.id)
- name
- location

**Relationship:**  
One company can have multiple warehouses.

---

### 3. products
Stores product-level information independent of warehouses.

- id (PK)
- company_id (FK → companies.id)
- name
- sku (UNIQUE)
- price
- product_type
- low_stock_threshold

**Design Note:**  
SKU is kept unique to avoid ambiguity in reporting and integrations.

---

### 4. inventory
Tracks quantity of a product in a specific warehouse.

- id (PK)
- product_id (FK → products.id)
- warehouse_id (FK → warehouses.id)
- quantity

**Relationship:**  
This table enables a many-to-many relationship between products and warehouses.

---

### 5. inventory_logs
Maintains a history of inventory changes for auditing and analytics.

- id (PK)
- product_id (FK → products.id)
- warehouse_id (FK → warehouses.id)
- change_quantity
- reason
- created_at

**Examples of reasons:** sale, restock, adjustment, transfer.

---

### 6. suppliers
Stores supplier information.

- id (PK)
- name
- contact_email

---

### 7. product_suppliers
Maps products to suppliers.

- product_id (FK → products.id)
- supplier_id (FK → suppliers.id)

**Relationship:**  
Supports multiple suppliers for a single product.

---

### 8. bundles
Defines bundled products.

- bundle_id (FK → products.id)
- product_id (FK → products.id)
- quantity

**Design Note:**  
Bundles are treated as products themselves to allow pricing, inventory, and tracking.

---

## Indexing & Constraints

- UNIQUE index on `products.sku`
- Indexes on foreign keys:
  - inventory.product_id
  - inventory.warehouse_id
  - inventory_logs.product_id
- Foreign key constraints to maintain referential integrity

---

## Missing Requirements / Clarification Questions

The following questions would be discussed with the product team:

1. Is SKU uniqueness global or per company?
2. How long should inventory history be retained?
3. Can a product have multiple active suppliers?
4. Is bundle pricing fixed or derived from child products?
5. Should low-stock thresholds vary per warehouse?

---

## Design Rationale

- Separating inventory from products allows flexible multi-warehouse support.
- Inventory logs enable traceability and analytics.
- Normalized schema reduces data duplication.
- Indexing ensures performance as data grows.

---

## Summary

This database design supports scalability, flexibility, and real-world inventory management needs.
It allows the system to grow in terms of companies, warehouses, products, and suppliers without major schema changes.
