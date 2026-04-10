# OrderUp API Testing Guide

This guide provides curl commands to test the OrderUp API running on `http://localhost:80`.

## Prerequisites

Ensure all services are running:
```bash
cd infra
docker-compose up -d
```

Run the Swagger UI at: http://localhost/orderup-docs/

## Quick Test: Health Check

```bash
curl -X GET http://localhost:80/menus
```

---

## Menu Service Tests

### 1. List All Menus
```bash
curl -X GET "http://localhost:80/menus"
```

### 2. List Active Menus Only
```bash
curl -X GET "http://localhost:80/menus?active=true"
```

### 3. Create a Menu
```bash
curl -X POST "http://localhost:80/menus" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Breakfast Menu",
    "active": true
  }'
```

### 4. Get Menu by ID
Replace `{menuId}` with an actual UUID from step 3.
```bash
curl -X GET "http://localhost:80/menus/{menuId}"
```

### 5. Update Menu
```bash
curl -X PUT "http://localhost:80/menus/{menuId}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Breakfast Menu",
    "active": false
  }'
```

### 6. Delete Menu
```bash
curl -X DELETE "http://localhost:80/menus/{menuId}"
```

---

## Menu Items Tests

### 1. List Menu Items
```bash
curl -X GET "http://localhost:80/menus/{menuId}/items"
```

### 2. List Available Menu Items Only
```bash
curl -X GET "http://localhost:80/menus/{menuId}/items?available=true"
```

### 3. Create Menu Item
```bash
curl -X POST "http://localhost:80/menus/{menuId}/items" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Pancakes",
    "description": "Fluffy pancakes with syrup",
    "price": 8.99,
    "imagePath": "images/pancakes.jpg",
    "available": true
  }'
```

### 4. Get Menu Item by ID
```bash
curl -X GET "http://localhost:80/menus/{menuId}/items/{menuItemId}"
```

### 5. Update Menu Item
```bash
curl -X PUT "http://localhost:80/menus/{menuId}/items/{menuItemId}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Pancakes with Whipped Cream",
    "description": "Fluffy pancakes with whipped cream and syrup",
    "price": 9.99,
    "imagePath": "images/pancakes.jpg",
    "available": true
  }'
```

### 6. Delete Menu Item
```bash
curl -X DELETE "http://localhost:80/menus/{menuId}/items/{menuItemId}"
```

---

## Order Service Tests

### 1. List All Orders
```bash
curl -X GET "http://localhost:80/orders"
```

### 2. List Orders by Source
Sources: `TABLET`, `WAITER`, `WEB`
```bash
curl -X GET "http://localhost:80/orders?source=TABLET"
```

### 3. List Orders by Status
Statuses: `CREATED`, `PREPARING`, `READY`, `SERVED`, `CANCELLED`, `PAID`
```bash
curl -X GET "http://localhost:80/orders?status=PREPARING"
```

### 4. Create an Order
```bash
curl -X POST "http://localhost:80/orders" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "TABLET",
    "items": []
  }'
```

### 5. Get Order by ID
```bash
curl -X GET "http://localhost:80/orders/{orderId}"
```

### 6. Add Item to Order
```bash
curl -X POST "http://localhost:80/orders/{orderId}/items" \
  -H "Content-Type: application/json" \
  -d '{
    "menuItemId": "{menuItemId}",
    "quantity": 2,
    "specialInstructions": "Extra syrup"
  }'
```

### 7. Update Order Item Quantity
```bash
curl -X PUT "http://localhost:80/orders/{orderId}/items/{orderItemId}" \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 3
  }'
```

### 8. Remove Item from Order
```bash
curl -X DELETE "http://localhost:80/orders/{orderId}/items/{orderItemId}"
```

---

## Order Status Tests

### 1. Get Current Order Status
```bash
curl -X GET "http://localhost:80/orders/{orderId}/status"
```

### 2. Get Order Status History
```bash
curl -X GET "http://localhost:80/orders/{orderId}/status-history"
```

### 3. Append Order Status Event
Statuses: `CREATED`, `PREPARING`, `READY`, `SERVED`, `CANCELLED`, `PAID`
```bash
curl -X POST "http://localhost:80/orders/{orderId}/status" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "PREPARING",
    "updatedBy": "kitchen-staff"
  }'
```

### 4. Get Order Item Status
```bash
curl -X GET "http://localhost:80/orders/{orderId}/items/{orderItemId}/status"
```

### 5. Get Order Item Status History
```bash
curl -X GET "http://localhost:80/orders/{orderId}/items/{orderItemId}/status-history"
```

### 6. Append Order Item Status Event
Item Statuses: `PENDING`, `PREPARING`, `READY`, `SERVED`, `CANCELLED`
```bash
curl -X POST "http://localhost:80/orders/{orderId}/items/{orderItemId}/status" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "PREPARING",
    "updatedBy": "kitchen-staff"
  }'
```

---

## Payment Service Tests

### 1. Get Payments by Order ID
```bash
curl -X GET "http://localhost:80/orders/{orderId}/payments"
```

### 2. Register Payment for Order
Payment Methods: `CREDIT_CARD`, `CASH`, `DIGITAL_WALLET`, etc.
```bash
curl -X POST "http://localhost:80/orders/{orderId}/payments" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 18.98,
    "method": "CREDIT_CARD",
    "status": "COMPLETED"
  }'
```

### 3. Get Payment by Payment ID
```bash
curl -X GET "http://localhost:80/payments/{paymentId}"
```

---

## Complete Test Flow Example

Here's a complete end-to-end test workflow:

```bash
#!/bin/bash

# 1. Create a menu
MENU=$(curl -s -X POST "http://localhost:80/menus" \
  -H "Content-Type: application/json" \
  -d '{"name": "Lunch Menu", "active": true}')
MENU_ID=$(echo $MENU | grep -oP '"id":"\K[^"]*')
echo "Created menu: $MENU_ID"

# 2. Add menu items
ITEM=$(curl -s -X POST "http://localhost:80/menus/$MENU_ID/items" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Burger",
    "description": "Delicious burger",
    "price": 12.99,
    "imagePath": "images/burger.jpg",
    "available": true
  }')
ITEM_ID=$(echo $ITEM | grep -oP '"id":"\K[^"]*')
echo "Created menu item: $ITEM_ID"

# 3. Create an order
ORDER=$(curl -s -X POST "http://localhost:80/orders" \
  -H "Content-Type: application/json" \
  -d '{"source": "WAITER", "items": []}')
ORDER_ID=$(echo $ORDER | grep -oP '"id":"\K[^"]*')
echo "Created order: $ORDER_ID"

# 4. Add items to order
curl -s -X POST "http://localhost:80/orders/$ORDER_ID/items" \
  -H "Content-Type: application/json" \
  -d "{\"menuItemId\": \"$ITEM_ID\", \"quantity\": 2}"
echo "Added item to order"

# 5. Update order status
curl -s -X POST "http://localhost:80/orders/$ORDER_ID/status" \
  -H "Content-Type: application/json" \
  -d '{"status": "PREPARING", "updatedBy": "kitchen"}'
echo "Updated order status"

# 6. Register payment
curl -s -X POST "http://localhost:80/orders/$ORDER_ID/payments" \
  -H "Content-Type: application/json" \
  -d '{"amount": 25.98, "method": "CREDIT_CARD", "status": "COMPLETED"}'
echo "Registered payment"
```

---

## Debugging Tips

### Check if API Gateway is Running
```bash
curl -v http://localhost:80/menus
```

### Check Service Discovery (Consul)
```bash
curl http://localhost:8500/v1/catalog/services
```

### Check Message Broker (ActiveMQ)
- Web UI: http://localhost:8161/admin/

### View Container Logs
```bash
docker logs menu-service
docker logs order-service
```

### Common Issues
- **Connection refused**: Services not running, check `docker-compose up`
- **Service not found**: Check Consul registration at http://localhost:8500
- **Timeout errors**: Services slow to start, wait 30+ seconds after `docker-compose up`
