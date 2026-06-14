# Entity Relationship (ER) Diagram

This document details the database schema and relationship designs for the **PharmaEZ** Pharmacy & Healthcare E-Commerce system.

```mermaid
erDiagram
    USER ||--o{ ORDER        : "places"
    USER ||--o{ REVIEW       : "writes"
    USER ||--o{ CART         : "has"
    USER ||--o{ PRESCRIPTION : "uploads"

    PRODUCT ||--o{ REVIEW       : "receives"
    PRODUCT ||--o{ ORDER_ITEM   : "referenced_in"
    PRODUCT ||--o{ CART_ITEM    : "referenced_in"

    ORDER ||--|{ ORDER_ITEM  : "contains"
    ORDER ||--o| PRESCRIPTION : "requires"

    CART  ||--|{ CART_ITEM   : "contains"

    ADMIN_CONFIG ||--o{ BANNER   : "manages"

    USER {
        ObjectId _id           PK
        string   name
        string   email         UK
        string   password
        string   phone
        string   role          "user | admin"
        number   loyaltyPoints
        boolean  isActive
        array    addresses
        array    wishlist
        date     createdAt
        date     updatedAt
    }

    PRODUCT {
        ObjectId _id                  PK
        string   name
        string   genericName
        string   description
        string   manufacturer
        string   category             "enum: 10 categories"
        string   subcategory
        array    images
        number   price
        number   discountPrice
        number   discountPercent
        number   stock
        string   unit
        string   packSize
        boolean  prescriptionRequired
        string   saltComposition
        string   dosageForm
        string   strength
        array    sideEffects
        string   usageInstructions
        string   storageInstructions
        date     expiryDate
        string   batchNumber
        boolean  isAvailable
        boolean  isFeatured
        array    reviews
        number   rating
        number   numReviews
        array    tags
        date     createdAt
        date     updatedAt
    }

    CART {
        ObjectId _id       PK
        ObjectId user      FK
        array    items
        string   couponCode
        number   discount
        date     createdAt
        date     updatedAt
    }

    CART_ITEM {
        ObjectId product             FK
        string   name
        string   image
        number   price
        number   discountPrice
        boolean  prescriptionRequired
        number   quantity
        string   packSize
    }

    ORDER {
        ObjectId _id                PK
        ObjectId user               FK
        string   orderNumber        UK
        array    items
        object   shippingAddress
        string   paymentMethod      "cod | upi | card | netbanking | wallet"
        string   paymentStatus      "pending | paid | failed | refunded"
        string   paymentId
        number   itemsPrice
        number   shippingPrice
        number   taxPrice
        number   discountAmount
        number   totalPrice
        string   orderStatus        "placed | confirmed | processing | shipped | delivered | cancelled | returned"
        boolean  prescriptionUploaded
        string   prescriptionUrl
        string   prescriptionStatus "not_required | pending | approved | rejected"
        array    statusHistory
        date     estimatedDelivery
        date     deliveredAt
        string   notes
        string   couponCode
        date     createdAt
        date     updatedAt
    }

    ORDER_ITEM {
        ObjectId product             FK
        string   name
        string   image
        number   price
        number   quantity
        string   packSize
        boolean  prescriptionRequired
    }

    PRESCRIPTION {
        ObjectId _id        PK
        ObjectId user       FK
        ObjectId order      FK
        string   fileName
        string   fileUrl
        string   doctorName
        string   notes
        string   status     "pending | approved | rejected"
        date     uploadedAt
    }

    REVIEW {
        ObjectId _id       PK
        ObjectId user      FK
        ObjectId product   FK
        string   name
        number   rating
        string   comment
        date     createdAt
    }

    ADMIN_CONFIG {
        ObjectId _id      PK
        array    banners
        object   settings
        date     createdAt
        date     updatedAt
    }

    BANNER {
        string   title
        string   subtitle
        string   image
        string   link
        boolean  isActive
        number   order
    }
```

---

## Schema Definitions

### 1. User Schema
Stores all registered customer and administrator data. The `email` field is indexed and unique. Passwords are hashed using `bcryptjs` (10 salt rounds) via a Mongoose `pre('save')` hook before persisting. The `addresses` array allows multiple saved delivery addresses with an `isDefault` flag. The `prescriptions` sub-array tracks uploaded prescription files with status lifecycle (`pending → approved | rejected`). The `wishlist` array holds `ObjectId` references to `Product` documents. The `loyaltyPoints` field is auto-incremented on each successful order delivery (1 point per ₹10 spent).

### 2. Product Schema
Represents every healthcare product in the catalogue. In addition to standard e-commerce fields, it stores pharmacy-specific metadata: `genericName`, `saltComposition`, `dosageForm`, `strength`, `sideEffects`, `usageInstructions`, and `storageInstructions`. The `prescriptionRequired` boolean drives the Rx workflow throughout the application. The `category` field is constrained to an enum of 10 valid pharmacy categories. An embedded `reviews` array stores user feedback; `rating` and `numReviews` are aggregate fields recalculated on each review submission. A `pre('save')` hook auto-computes `discountPercent` from `price` and `discountPrice`.

### 3. Cart Schema
Ensures persistent shopping cart records linked one-to-one with a `User` (`unique: true`). Each item in the `items` array snapshots the product's price at time of adding to avoid stale pricing issues. Virtual fields `totalItems` and `subtotal` are computed dynamically using `.toJSON({ virtuals: true })` without persisting to the database.

### 4. Order Schema
Links users to their complete purchase history. The `orderNumber` field is auto-generated via a `pre('save')` hook using the format `PEZ` + last 6 digits of timestamp + 4-digit zero-padded sequence (e.g., `PEZ7829430001`). The `statusHistory` array provides a full audit trail — every status change is logged with a `timestamp` and optional `note`. The `prescriptionStatus` field links the order to the prescription review workflow for Rx items.

### 5. Prescription Schema (Embedded in User)
Tracks prescription files uploaded by users. Stored as an embedded sub-document array inside the `User` model. Each entry records the uploaded file URL, the prescribing doctor's name, admin review notes, and the current approval status. The admin dashboard surfaces these for pharmacist review.

### 6. Review Schema (Embedded in Product)
Collects customer feedback on individual products. Stored as an embedded array within `Product` documents. After each new review, the controller recalculates and persists the aggregate `rating` and `numReviews` on the parent Product document.

### 7. Admin Config Schema
A single-document collection (one record in the database) that stores global store settings — including the store name, support email, free shipping threshold (`₹499`), flat shipping charge (`₹49`), GST tax rate (`5%`), and homepage banner configuration.

---

[◄ Back to Project Architecture](../project_architecture.md) | [Back to Home](../README.md)
