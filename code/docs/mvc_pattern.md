# MVC Pattern in MERN

This document explains the Model-View-Controller architecture configuration for the **PharmaEZ** Pharmacy & Healthcare E-Commerce application.

```
       [ BROWSER ] ◄────────────────────────────────────────────┐
            │                                                   │
            ▼                                                   │
┌───────────────────────────┐                                   │
│           VIEW            │  (React.js — SPA, Context API)   │  HTTP
│  Pages / Components / UI  │                                   │  Response
└───────────┬───────────────┘                                   │  (JSON)
            │  Trigger Events                                   │
            │  (Axios API Requests)                             │
            ▼                                                   │
┌────────────────────────────────────────────────────────────┐  │
│                   SERVER (Node.js / Express)               │  │
│                                                            │  │
│  ┌─────────────────┐         ┌───────────────────────┐     │  │
│  │     ROUTER      │ ──────► │     CONTROLLER        │     │──┘
│  │  (Express.js)   │         │  (Business Logic)     │     │
│  └─────────────────┘         └──────────┬────────────┘     │
│                                         │ interacts        │
│                                         ▼                  │
│                               ┌─────────────────┐          │
│                               │      MODEL      │          │
│                               │   (Mongoose)    │          │
│                               └────────┬────────┘          │
└────────────────────────────────────────┼───────────────────┘
                                         ▼
                                  ┌─────────────┐
                                  │  DATABASE   │  (MongoDB)
                                  └─────────────┘
```

---

## MVC Core Division

### 1. View (Client Layer — React.js)

*   Dynamic single-page application that renders state data from `AuthContext` and `CartContext`.
*   All UI components reside in `client/src/` — pages, layout components, and admin components.
*   State is managed via the Context API (`AuthContext` for session management, `CartContext` for real-time cart updates).
*   Sends HTTP requests to backend endpoints via the pre-configured Axios instance (`src/utils/api.js`) which automatically injects the JWT Bearer token.
*   Route access is controlled by `ProtectedRoute`, `AdminRoute`, and `PublicRoute` wrapper components.

**Key View Files:**

| File | Responsibility |
| :--- | :--- |
| `src/pages/HomePage.jsx` | Hero section, categories, offer banners, featured products |
| `src/pages/ProductsPage.jsx` | Filtered/sorted product listing with sidebar and pagination |
| `src/pages/ProductDetailPage.jsx` | Full product view with Rx notice, reviews, and cart actions |
| `src/pages/CartPage.jsx` | Cart management with Rx notice and free shipping bar |
| `src/pages/CheckoutPage.jsx` | 3-step checkout (Address → Payment → Review) |
| `src/pages/admin/AdminDashboard.jsx` | KPI cards, revenue chart, recent orders, low-stock alerts |
| `src/pages/admin/AdminOrders.jsx` | Order management with inline status update |
| `src/pages/admin/AdminPrescriptions.jsx` | Prescription review and approval workflow |
| `src/components/common/ProductCard.jsx` | Reusable card component used across product listings |
| `src/components/admin/AdminLayout.jsx` | Collapsible sidebar admin shell |

---

### 2. Router (Entry Layer — Express Routes)

*   Catches incoming HTTP requests and dispatches to the correct controller.
*   Applies `protect` and `adminOnly` middleware before reaching protected controllers.
*   Configures Multer for file upload routes (prescriptions, product images).
*   All routes are namespaced under `/api/` for clean separation from the static file server.

**Route Files:**

| File | Prefix | Key Endpoints |
| :--- | :--- | :--- |
| `authRoutes.js` | `/api/auth` | register, login, profile, addresses, wishlist, prescription upload |
| `productRoutes.js` | `/api/products` | list (filters+pagination), detail, featured, category, reviews, CRUD (admin) |
| `cartRoutes.js` | `/api/cart` | get, add, update quantity, remove item, clear |
| `orderRoutes.js` | `/api/orders` | place, my orders, detail, cancel, admin all orders & stats, status update |
| `adminRoutes.js` | `/api/admin` | users, prescription review, config, inventory report |

---

### 3. Controller (Logic Layer — Express Controllers)

Acts as the mediator. Receives the validated `req` object from the router, coordinates CRUD operations with Models, handles business logic, and formats the output JSON response.

**Controller Files:**

| File | Key Functions |
| :--- | :--- |
| `authController.js` | `register`, `login`, `getProfile`, `updateProfile`, `changePassword`, `addAddress`, `deleteAddress`, `toggleWishlist`, `uploadPrescription` |
| `productController.js` | `getProducts` (full filter engine), `getProductById`, `getFeaturedProducts`, `createProduct`, `updateProduct`, `deleteProduct`, `addReview`, `getProductsByCategory` |
| `cartController.js` | `getCart`, `addToCart` (stock validation), `updateCartItem`, `removeFromCart`, `clearCart` |
| `orderController.js` | `placeOrder` (stock deduction + loyalty points), `getMyOrders`, `getOrderById`, `cancelOrder` (stock restore), `getAllOrders`, `updateOrderStatus`, `getDashboardStats` |
| `adminController.js` | `getAllUsers`, `toggleUserStatus`, `getPrescriptions`, `updatePrescriptionStatus`, `getAdminConfig`, `updateAdminConfig`, `getInventoryReport` |

---

### 4. Model (Data Layer — Mongoose Schemas)

Specifies database schema rules, field validation, indexes, pre-save hooks (password hashing, order number generation, discount calculation), instance methods, and virtual field definitions.

**Model Files:**

| File | Collection | Key Features |
| :--- | :--- | :--- |
| `User.js` | `users` | Unique email index, bcryptjs pre-save hook, `matchPassword()` instance method, embedded addresses[] and prescriptions[] sub-documents, wishlist ref array |
| `Product.js` | `products` | 10-value category enum, pharmacy-specific fields (saltComposition, dosageForm, strength), embedded reviews[], auto-calculate `discountPercent` pre-save hook |
| `Cart.js` | `carts` | One-to-one user reference (unique), `totalItems` and `subtotal` virtual fields with `toJSON({virtuals:true})` |
| `Order.js` | `orders` | Auto-generated `orderNumber` pre-save hook, `statusHistory[]` audit trail, `prescriptionStatus` enum for Rx workflow |
| `AdminConfig.js` | `adminconfigs` | Single-document store config, embedded banners[] and settings object |

---

## Data Flow Example — Place Order

```
User clicks "Place Order"
        │
        ▼
  [VIEW — CheckoutPage.jsx]
  calls: api.post('/orders', payload)
        │
        ▼
  [ROUTER — orderRoutes.js]
  POST /api/orders
  middleware: protect (verify JWT, attach req.user)
  → orderController.placeOrder
        │
        ▼
  [CONTROLLER — orderController.js: placeOrder()]
  1. Fetch cart from Cart model (with populated Product refs)
  2. Validate stock for all items
  3. Calculate itemsPrice, shippingPrice, taxPrice, totalPrice
  4. Create Order document
  5. Deduct stock on each Product model
  6. Award loyalty points on User model
  7. Clear cart on Cart model
  8. Return order object as JSON
        │
        ▼
  [MODEL — Order.js]
  - pre('save') hook generates orderNumber
  - Persists to MongoDB orders collection

  [MODEL — Product.js]
  - stock decremented via $inc operator

  [MODEL — User.js]
  - loyaltyPoints incremented via $inc operator

  [MODEL — Cart.js]
  - items[] cleared, saved
        │
        ▼
  [VIEW — OrderConfirmationPage.jsx]
  Displays order number, estimated delivery, Rx notice
```

---

## Advantages of MVC in PharmaEZ

*   **Separation of Concerns**: UI logic (React), business logic (controllers), and data logic (Mongoose models) are cleanly separated — a change in one layer does not cascade into others.
*   **Scalability**: New features (e.g., a doctor consultation module, loyalty redemption) are added by creating new routes, controllers, and model fields without touching existing code.
*   **Reusability**: The `protect` and `adminOnly` middleware, global error handler, and Mongoose model methods are reused across the entire application without duplication.
*   **Testability**: Each layer can be independently unit tested — controllers can be tested with mocked models, and models can be tested against an in-memory MongoDB instance.
*   **Parallel Development**: Frontend and backend teams can develop simultaneously against agreed API contracts without blocking each other.

---

[◄ Back to Project Architecture](../project_architecture.md) | [Back to Home](../README.md)
