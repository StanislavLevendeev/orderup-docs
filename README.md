# OrderUp API Documentation

This documentation is written in the context of the group project for the course "Service-Oriented Architecture" at the University of Twente. The project involves designing and implementing a system called "OrderUp," which consists of multiple services that interact with each other to manage menus, orders, order statuses, notifications, and payments.

## Solution Design Documentation

### Overview
This document describes the architecture and design of the OrderUp system, which is composed of multiple services. Each service is responsible for a specific domain within the system, and they interact with each other through well-defined relationships and events.

### Services and Classes

#### Menu Service
The `Menu Service` is responsible for managing menus and their items.

- **Menu**: Represents a menu with the following attributes:
  - `id` (UUID): Unique identifier for the menu.
  - `name` (String): Name of the menu.
  - `active` (boolean): Indicates whether the menu is active.

- **MenuItem**: Represents an item in a menu with the following attributes:
  - `id` (UUID): Unique identifier for the menu item.
  - `name` (String): Name of the menu item.
  - `description` (String): Description of the menu item.
  - `imagePath` (String): Path to the image of the menu item.
  - `price` (BigDecimal): Price of the menu item.
  - `productId` (UUID): Identifier for the associated product.
  - `available` (boolean): Indicates whether the menu item is available.

**Relationships**:
- A `Menu` can have one or more `MenuItem` objects.

#### Order Service
The `Order Service` handles customer orders and their items.

- **Order**: Represents a customer order with the following attributes:
  - `id` (UUID): Unique identifier for the order.
  - `createdAt` (LocalDateTime): Timestamp when the order was created.
  - `totalPrice` (BigDecimal): Total price of the order.
  - `source` (String): Source of the order (e.g., web, mobile app).

- **OrderItem**: Represents an item in an order with the following attributes:
  - `id` (UUID): Unique identifier for the order item.
  - `orderId` (UUID): Identifier for the associated order.
  - `menuItemId` (UUID): Identifier for the associated menu item.
  - `quantity` (int): Quantity of the item ordered.
  - `price` (BigDecimal): Price of the item.

**Relationships**:
- An `Order` can have one or more `OrderItem` objects.
- An `OrderItem` references a `MenuItem` by its `menuItemId`.

#### Order Status Tracking Service
The `Order Status Tracking Service` tracks the status of orders and their items.

- **OrderStatusHistory**: Represents the status history of an order with the following attributes:
  - `id` (UUID): Unique identifier for the status history entry.
  - `orderId` (UUID): Identifier for the associated order.
  - `status` (OrderStatusType): Current status of the order.
  - `timestamp` (LocalDateTime): Timestamp of the status update.
  - `updatedBy` (String): Identifier of the user or system that updated the status.

- **OrderItemStatusHistory**: Represents the status history of an order item with the following attributes:
  - `id` (UUID): Unique identifier for the status history entry.
  - `orderItemId` (UUID): Identifier for the associated order item.
  - `status` (OrderItemStatusType): Current status of the order item.
  - `timestamp` (LocalDateTime): Timestamp of the status update.
  - `updatedBy` (String): Identifier of the user or system that updated the status.

**Relationships**:
- `OrderStatusHistory` can derive the state of an order from one or more `OrderItemStatusHistory` objects.

#### Order Notification Service
The `Order Notification Service` handles events related to order and order item status changes.

- **OrderItemStatusChangedEvent**: Represents an event triggered when the status of an order item changes, with the following attributes:
  - `orderId` (UUID): Identifier for the associated order.
  - `orderItemId` (UUID): Identifier for the associated order item.
  - `newStatus` (OrderItemStatusType): New status of the order item.
  - `timestamp` (LocalDateTime): Timestamp of the status change.

- **OrderStatusChangedEvent**: Represents an event triggered when the status of an order changes, with the following attributes:
  - `orderId` (UUID): Identifier for the associated order.
  - `newStatus` (OrderStatusType): New status of the order.
  - `timestamp` (LocalDateTime): Timestamp of the status change.

#### Payment Service
The `Payment Service` handles payment processing for orders.

- **Payment**: Represents a payment with the following attributes:
  - `id` (UUID): Unique identifier for the payment.
  - `orderId` (UUID): Identifier for the associated order.
  - `amount` (BigDecimal): Amount of the payment.
  - `status` (PaymentStatus): Status of the payment (e.g., pending, completed).
  - `method` (String): Payment method used (e.g., credit card, PayPal).
  - `createdAt` (LocalDateTime): Timestamp when the payment was created.

**Relationships**:
- A `Payment` references an `Order` by its `orderId`.

### Cross-Service Relationships
- `OrderItem` references `MenuItem` by `menuItemId`.
- `Order` references `Payment` by `orderId`.
- `Order` references `OrderStatusHistory` by `orderId`.
- `OrderItem` references `OrderItemStatusHistory` by `orderItemId`.
- `OrderItemStatusChangedEvent` references `OrderItem` by `orderItemId`.
- `OrderStatusChangedEvent` references `Order` by `orderId`.