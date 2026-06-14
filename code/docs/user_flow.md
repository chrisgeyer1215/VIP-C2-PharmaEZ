# User Flows

This document maps out the key user journeys in the **PharmaEZ** Pharmacy & Healthcare E-Commerce platform.

---

## 1. Customer Checkout Flow

This diagram outlines how a user adds healthcare products to the cart, proceeds through the 3-step checkout, and successfully places an order — including stock validation and loyalty point award.

```mermaid
sequenceDiagram
    actor User
    participant Frontend as React App
    participant Backend  as Node/Express API
    participant DB       as MongoDB

    User->>Frontend: Add Product to Cart (Add to Cart button)
    Frontend->>Backend: POST /api/cart {productId, quantity}
    Backend->>DB: Validate stock availability
    DB-->>Backend: Stock OK
    Backend->>DB: Save/update cart document
    Backend-->>Frontend: Return updated cart
    Frontend->>User: Update cart badge count & show toast notification

    User->>Frontend: Click "Proceed to Checkout"
    Frontend->>Backend: Validate JWT token (Axios interceptor)
    Backend-->>Frontend: Token valid — proceed

    Frontend->>User: Step 1 — Display saved addresses / New address form
    User->>Frontend: Select delivery address
    Frontend->>User: Step 2 — Display payment method options
    User->>Frontend: Select payment method (COD / UPI / Card)
    Frontend->>User: Step 3 — Display full order review & total

    User->>Frontend: Click "Place Order ₹XXX"
    Frontend->>Backend: POST /api/orders {shippingAddress, paymentMethod, notes}
    Backend->>DB: Final stock validation for all cart items
    DB-->>Backend: Stock confirmed

    Backend->>DB: Create Order document (status: placed)
    Backend->>DB: Deduct stock for all ordered items
    Backend->>DB: Award loyalty points to user (1 per ₹10)
    Backend->>DB: Clear user's cart

    Backend-->>Frontend: Return Order object {orderNumber, estimatedDelivery, totalPrice}
    Frontend->>User: Display Order Confirmation page with order number (PEZ######)
```

---

## 2. Prescription Upload & Verification Flow

This diagram shows the end-to-end Rx (prescription medicine) workflow — from a customer uploading a prescription to a pharmacist/admin approving it and the order proceeding to fulfilment.

```mermaid
sequenceDiagram
    actor User
    actor Admin as Admin / Pharmacist
    participant Frontend  as React App
    participant Backend   as Node/Express API
    participant DB        as MongoDB
    participant Storage   as File Storage (uploads/)

    User->>Frontend: Navigate to Prescriptions page or Product Detail (Rx item)
    Frontend->>User: Display prescription upload form (file + doctor name)

    User->>Frontend: Select prescription file (JPG / PNG / PDF, max 5MB)
    User->>Frontend: Enter doctor's name (optional)
    User->>Frontend: Click "Upload Prescription"

    Frontend->>Backend: POST /api/auth/prescription (multipart/form-data)
    Backend->>Storage: Save file via Multer → uploads/prescription-{timestamp}.pdf
    Backend->>DB: Push {fileName, fileUrl, doctorName, status: "pending"} to user.prescriptions[]
    Backend-->>Frontend: Return updated prescriptions array
    Frontend->>User: Show success toast — "Prescription uploaded successfully"

    Note over User,DB: User places an order containing Rx items

    Backend->>DB: Set order.prescriptionStatus = "pending"
    Backend-->>Frontend: Order confirmation with Rx pending notice

    Note over Admin,DB: Admin reviews prescription queue

    Admin->>Frontend: Navigate to Admin → Prescriptions page
    Frontend->>Backend: GET /api/admin/prescriptions
    Backend->>DB: Fetch all users with uploaded prescriptions
    Backend-->>Frontend: Return prescriptions list
    Frontend->>Admin: Display prescription table with patient info & file preview

    Admin->>Frontend: Click "View" — preview image inline or open PDF
    Admin->>Frontend: Click "Approve" or "Reject"

    Frontend->>Backend: PUT /api/admin/prescriptions/:userId/:prescriptionId {status: "approved"}
    Backend->>DB: Update prescription.status → "approved"
    Backend-->>Frontend: Return success response

    Frontend->>Admin: Show status updated toast
    Note over User,DB: Order prescription status updated — fulfilment can proceed
```

---

## 3. Admin Order Management Flow

This diagram outlines how an administrator manages the order lifecycle — from viewing placed orders to updating fulfilment status — with a full audit trail.

```mermaid
sequenceDiagram
    actor Admin
    participant Frontend as Admin React Panel
    participant Backend  as Node/Express API
    participant DB       as MongoDB

    Admin->>Frontend: Navigate to Admin → Orders page
    Frontend->>Backend: GET /api/orders/admin/all?status=placed&page=1
    Backend->>DB: Query orders with filters & pagination
    DB-->>Backend: Return orders array + total count
    Backend-->>Frontend: Paginated orders response
    Frontend->>Admin: Display orders table with filter pills & status badges

    Admin->>Frontend: Click on an order row
    Frontend->>Admin: Slide-in Order Detail Panel (customer, items, address, invoice)

    Admin->>Frontend: Change status dropdown → "confirmed"
    Frontend->>Backend: PUT /api/orders/admin/:id/status {status: "confirmed", note: "Payment verified"}
    Backend->>DB: Update order.orderStatus → "confirmed"
    Backend->>DB: Push {status: "confirmed", timestamp: now, note} to order.statusHistory[]
    Backend-->>Frontend: Return updated order object
    Frontend->>Admin: Update status badge in table — show toast "Order status updated"

    Note over Admin,DB: Order progresses through fulfilment stages

    Admin->>Frontend: Update status → "shipped"
    Frontend->>Backend: PUT /api/orders/admin/:id/status {status: "shipped"}
    Backend->>DB: Update order.orderStatus → "shipped"
    Backend->>DB: Append to statusHistory[]
    Backend-->>Frontend: Updated order

    Admin->>Frontend: Update status → "delivered"
    Frontend->>Backend: PUT /api/orders/admin/:id/status {status: "delivered"}
    Backend->>DB: Update order.orderStatus → "delivered"
    Backend->>DB: Set order.deliveredAt = current timestamp
    Backend->>DB: Set order.paymentStatus → "paid"
    Backend->>DB: Append to statusHistory[]
    Backend-->>Frontend: Updated order
    Frontend->>Admin: Order marked as delivered — customer loyalty points already credited
```

---

## 4. User Registration & Login Flow

```mermaid
sequenceDiagram
    actor User
    participant Frontend as React App
    participant Backend  as Node/Express API
    participant DB       as MongoDB

    User->>Frontend: Fill registration form (name, email, phone, password)
    Frontend->>Backend: POST /api/auth/register
    Backend->>DB: Check if email already exists
    DB-->>Backend: Email is unique — OK

    Backend->>DB: Create User document (password hashed via bcryptjs pre-save hook)
    Backend->>Backend: Generate JWT token (7-day expiry)
    Backend-->>Frontend: {token, user: {_id, name, email, role, loyaltyPoints}}

    Frontend->>Frontend: Store token in localStorage
    Frontend->>Frontend: Set AuthContext user state
    Frontend->>User: Redirect to homepage — show "Welcome to PharmaEZ 🎉"

    Note over User,DB: On subsequent visits

    User->>Frontend: Enter email & password on Login page
    Frontend->>Backend: POST /api/auth/login
    Backend->>DB: Find user by email
    Backend->>Backend: Compare password with bcrypt.compare()
    Backend->>Backend: Check user.isActive === true
    Backend->>Backend: Generate new JWT token
    Backend-->>Frontend: {token, user object}

    Frontend->>Frontend: Store token & user in localStorage
    Frontend->>Frontend: Restore AuthContext state
    Frontend->>Frontend: Fetch cart from /api/cart (CartContext)
    Frontend->>User: Redirect to previous page or homepage — show "Welcome back!"
```

---

[◄ Back to Project Architecture](../project_architecture.md) | [Back to Home](../README.md)
