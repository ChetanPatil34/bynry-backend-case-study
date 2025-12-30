# Part 3: Low Stock Alert API Implementation

## Objective

The purpose of this API is to generate low-stock alerts for a company.
These alerts help businesses proactively reorder products before stock runs out,
especially when products are actively being sold.

The API considers multiple warehouses, product-specific thresholds,
and supplier information for reordering.

---

## Endpoint Definition

GET /api/companies/{company_id}/alerts/low-stock

---

## Business Rules

- Low-stock threshold varies by product type.
- Alerts are generated only for products with recent sales activity.
- Products may exist in multiple warehouses.
- Supplier information is included to support reordering decisions.

---

## Assumptions

- Recent sales activity refers to sales in the last 30 days.
- Low-stock threshold is stored at the product level.
- Average daily sales are used to estimate days until stockout.
- Each product has at most one primary supplier for reordering.

---

## High-Level Approach

1. Fetch all products belonging to the given company.
2. For each product, identify inventory records across warehouses.
3. Check if current stock is below the defined threshold.
4. Verify that the product has recent sales activity.
5. Fetch supplier details for the product.
6. Construct the alert response per warehouse.

---

## Sample Implementation (Java – Spring Boot Style)

```java
@GetMapping("/api/companies/{companyId}/alerts/low-stock")
public LowStockAlertResponse getLowStockAlerts(@PathVariable Long companyId) {

    List<LowStockAlert> alerts = new ArrayList<>();

    List<Product> products = productRepository.findByCompanyId(companyId);

    for (Product product : products) {

        if (!salesRepository.hasRecentSales(product.getId(), 30)) {
            continue;
        }

        List<Inventory> inventories =
                inventoryRepository.findByProductId(product.getId());

        for (Inventory inventory : inventories) {

            if (inventory.getQuantity() < product.getLowStockThreshold()) {

                Supplier supplier =
                        supplierRepository.findPrimarySupplierByProductId(product.getId());

                LowStockAlert alert = new LowStockAlert();
                alert.setProductId(product.getId());
                alert.setProductName(product.getName());
                alert.setSku(product.getSku());
                alert.setWarehouseId(inventory.getWarehouse().getId());
                alert.setWarehouseName(inventory.getWarehouse().getName());
                alert.setCurrentStock(inventory.getQuantity());
                alert.setThreshold(product.getLowStockThreshold());
                alert.setDaysUntilStockout(estimateStockoutDays(product.getId()));
                alert.setSupplier(supplier);

                alerts.add(alert);
            }
        }
    }

    return new LowStockAlertResponse(alerts, alerts.size());
}
