# Part 1: Code Review & Debugging

## Given Context

In this section, I reviewed the provided API endpoint responsible for creating a new product and initializing its inventory.
Although the code works at a basic level, it contains several technical and business-logic issues that can cause problems in a real production environment.

The review focuses on data consistency, scalability, and proper backend practices.

---

## Issues Identified

1. **SKU uniqueness is not enforced**  
   The API does not validate whether a product with the same SKU already exists in the system.

2. **Product is tightly coupled with a single warehouse**  
   The product entity directly stores the warehouse ID, which prevents the same product from being managed across multiple warehouses.

3. **Lack of transaction management**  
   Product creation and inventory creation are committed separately. If one operation fails, inconsistent data may be saved.

4. **Missing input validation**  
   Required fields such as product name, SKU, price, and quantity are accessed directly without validation.

5. **Improper price handling**  
   The price field is not validated for decimal precision, which may lead to rounding or calculation issues.

6. **No rollback mechanism**  
   If inventory creation fails, the product record still remains in the database.

7. **Warehouse existence is not validated**  
   The API assumes the provided warehouse ID is valid without checking its existence.

---

## Impact in Production

- Duplicate SKUs can cause reporting and billing issues.
- Partial data commits may leave products without inventory.
- The system cannot scale efficiently for multi-warehouse setups.
- Invalid data may be stored, leading to runtime errors.
- Debugging and maintenance become more difficult.

---

## Improved Approach

To improve reliability and scalability, the following changes are recommended:

- Enforce SKU uniqueness before saving a product.
- Separate product creation from warehouse inventory management.
- Use database transactions to ensure atomic operations.
- Validate all required inputs before processing.
- Roll back database changes if any step fails.

---

## Sample Improved Code (Java – Illustrative)

```java
@Transactional
public Product createProduct(CreateProductRequest request) {

    if (productRepository.existsBySku(request.getSku())) {
        throw new IllegalArgumentException("SKU already exists");
    }

    Product product = new Product();
    product.setName(request.getName());
    product.setSku(request.getSku());
    product.setPrice(request.getPrice());

    productRepository.save(product);

    Inventory inventory = new Inventory();
    inventory.setProduct(product);
    inventory.setWarehouseId(request.getWarehouseId());
    inventory.setQuantity(request.getInitialQuantity());

    inventoryRepository.save(inventory);

    return product;
}
