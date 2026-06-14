# PharmaEZ — Pharmacy & Healthcare E-Commerce UI Walkthrough

Welcome to the **PharmaEZ** UI Walkthrough! This document lists and demonstrates the user interfaces built for the MERN Stack Pharmacy & Healthcare E-Commerce application.

All screenshots are stored directly inside the repository under the `screenshots/` directory.

---

## 📸 UI Screenshots & Flow

### 1. Landing Page (Homepage)

The homepage is the first page a user sees. It features an animated hero section with the tagline **"Your Health, Our Priority"**, three trust badge cards (100% Genuine Products, Fast Delivery, 24/7 Pharmacist Support, and 4.8★ Rated), an 8-tile healthcare category grid, three promotional offer banners (40% Off Vitamins & Supplements, Free Delivery above ₹499, and Prescription Medicines upload), a Featured Products section, and a Health & Wellness Tips section at the bottom.

![PharmaEZ Homepage](./screenshots/homepage.png)

---

### 2. Products Listing Page

The product listing page displays all healthcare products in a 4-column responsive grid. A sticky left sidebar provides filter controls — category radio buttons (10 healthcare categories), prescription requirement filter (OTC / Rx / All), and a price range input with an Apply button. Applied filters appear as dismissible tags above the grid. A sort dropdown in the top-right allows ordering by Relevance, Price (Low to High / High to Low), Top Rated, and Newest.

![Products Listing Page](./screenshots/products_listing.png)

---

### 3. Product Detail Page

Clicking on any product card opens the full product detail view. It displays a main product image with a thumbnail strip (for multiple images), the product's Discount badge and Rx Required badge (where applicable), the generic name and manufacturer, a star rating display with review count, the current price alongside the MRP with strikethrough, savings amount, quantity selector, Add to Cart / Buy Now / Add to Wishlist action buttons, and a prominent Rx prescription notice with a link to the Upload Prescription page for Rx medicines. Below the fold, a tabbed panel provides Description & Clinical Details, Usage Instructions, and Customer Reviews.

![Product Detail Page](./screenshots/product_detail.png)

---

### 4. Shopping Cart

The shopping cart page lists all added medicines and healthcare products with their images, names, pack sizes, and Rx badges. Each item has quantity increase/decrease controls and a remove button. At the top, a banner alerts the user if any cart item requires a prescription. The right-side summary card shows the subtotal, a dynamic free-shipping progress bar (showing how much more is needed to reach the ₹499 free delivery threshold), shipping charge, 5% GST, total payable amount, and savings on discounted items. The Proceed to Checkout button is prominently placed.

![Shopping Cart](./screenshots/cart.png)

---

### 5. Create Account (Register Page)

A clean, centred registration form with fields for Full Name, Email Address, Phone Number, and Password (minimum 6 characters). The PharmaEZ logo and tagline are displayed at the top. Demo credentials are shown below the form for quick testing. A link to the Login page is provided for existing users.

![Register Page](./screenshots/register.png)

---

### 6. Welcome Back (Login Page)

A clean, secure login form for user authentication using JSON Web Tokens (JWT). Fields are Email Address and Password. Demo credentials for both Admin and User roles are displayed inside a highlighted box for easy testing. A link to the Register page is provided for new users.

![Login Page](./screenshots/login.png)

---

### 7. Checkout — Multi-Step Flow

The checkout page presents a 3-step guided process with a visual step indicator at the top:

*   **Step 1 — Delivery Address**: Users can select from their saved delivery addresses (shown as selectable cards) or fill in a new address inline. The form captures Full Name, Phone, Street Address, City, State, and Pincode.
*   **Step 2 — Payment Method**: Four payment options are displayed as selectable cards — Cash on Delivery (COD), UPI / QR Code, Credit / Debit Card, and Net Banking. An optional Order Notes textarea is available.
*   **Step 3 — Review Order**: A complete order summary — item list with quantities and prices, selected delivery address, chosen payment method, and full price breakdown (subtotal, shipping, GST, total) — is shown before the final Place Order button.

![Checkout Flow](./screenshots/checkout.png)

---

### 8. Order Confirmation Page

After successfully placing an order, users are taken to the confirmation page. It prominently displays the unique auto-generated order number (format: `PEZ######`, e.g., `PEZ7829430001`), the estimated delivery date (5 business days), payment method and current payment status. If the order contains Rx medicines, a yellow warning box prompts the user to upload a valid prescription. Two action buttons — **Track Order** and **Continue Shopping** — are provided.

![Order Confirmation](./screenshots/order_confirmation.png)

---

### 9. User Dashboard

The user dashboard is the account home page for registered customers. A welcome card at the top displays the user's avatar initial, full name, email address, and a Loyalty Points badge (e.g., "🎁 145 Loyalty Points"). Below, four quick-access cards navigate to: My Orders, Wishlist, Prescriptions, and Edit Profile.

![User Dashboard](./screenshots/user_dashboard.png)

---

### 10. Order Tracking Page

The order detail page features a 5-step visual progress tracker at the top showing the order's journey: **Placed → Confirmed → Processing → Shipped → Delivered** (with filled and unfilled connectors indicating current progress). Below the tracker, the full items list is shown with images, names, quantities, and line totals. The right column shows the Delivery Address card, Payment Details card, and a Price Summary card with itemised breakdown. A red Cancel Order button is available for orders not yet shipped.

![Order Tracking](./screenshots/order_tracking.png)

---

### 11. Prescription Management Page

The Prescriptions page includes two sections: an upload form at the top (with a dashed drag-and-drop style border, file input accepting JPG/PNG/PDF up to 5MB, and an optional Doctor's Name field), and a list of all previously uploaded prescriptions below. Each prescription entry in the list shows the file name, doctor name, upload date, and a colour-coded status badge — **Pending** (amber), **Approved** (green), or **Rejected** (red).

![Prescription Management](./screenshots/prescriptions.png)

---

### 12. Admin Dashboard

A clean, sidebar-layout admin panel accessible only to administrators (`admin@pharmaez.com` / `admin123`). At the top, four KPI cards display Total Orders, Total Revenue (₹), Total Registered Users, and Total Products. Below, two side-by-side panels show the Order Status Breakdown (horizontal bar chart by status) and a 7-Day Revenue bar chart. At the bottom, a Recent Orders table and a Low-Stock Alert table are displayed side by side.

![Admin Dashboard](./screenshots/admin_dashboard.png)

---

### 13. All Orders — Admin Panel

The admin orders page features filter pills at the top for every order status (All, Placed, Confirmed, Processing, Shipped, Delivered, Cancelled, Returned) and a search box for order number lookup. The main area is a paginated table listing every order with Order Number, Customer name/email, Date, Item count, Total, Payment method badge, and an inline Status dropdown for quick updates. Clicking a row opens a slide-in Order Detail panel on the right showing the customer's full details, itemised invoice, delivery address, price summary, and a status update selector.

![Admin Orders](./screenshots/admin_orders.png)

---

### 14. All Products — Admin Panel

The admin products page shows the complete pharmacy catalogue in a searchable, paginated table. Each row displays the product image thumbnail, product name with generic name below, a category badge, current price with MRP, stock count (colour-coded green for healthy, amber for low, red for out-of-stock), Rx badge, Featured badge, and three action buttons: View (opens store product page), Edit (opens the product form), and Delete (with confirmation prompt).

![Admin Products](./screenshots/admin_products.png)

---

### 15. Add / Edit Product — Admin Panel

The product form page is used for both creating new healthcare products and editing existing ones. The form is divided into four organised sections:
1.  **Basic Information** — Product Name, Generic Name, Description, Manufacturer, Category, Subcategory.
2.  **Pricing & Inventory** — MRP Price, Discounted Price, Stock Quantity, Unit, Pack Size, Dosage Form.
3.  **Medical / Clinical Details** — Salt Composition, Strength, Prescription Required toggle, Usage Instructions, Storage Instructions, Side Effects.
4.  **Visibility & Tags** — Tags (comma-separated), Image URLs, Is Available toggle, Featured Product toggle.

![Admin Product Form](./screenshots/admin_product_form.png)

---

### 16. Prescription Review — Admin Panel

The prescription review page gives pharmacists a dedicated workspace for verifying customer-uploaded prescriptions. Four stat cards at the top show counts by status (Total, Pending, Approved, Rejected). Below, filter pills let admins triage by status. The prescription table shows Patient Name/Email, Doctor Name, a View File button, Upload Date, current Status badge, and Approve/Reject action buttons. Clicking a prescription row opens a detail panel showing patient information, an inline image preview (or PDF open link), and large Approve / Reject buttons with immediate feedback.

![Prescription Review](./screenshots/admin_prescriptions.png)

---

## 🚀 Local Execution & Verification

To run this project locally, refer to the setup guide in [README.md](./README.md) or follow these commands:

1.  **Seed the Database** (first time only):
    ```bash
    cd server
    npm run seed
    ```

2.  **Start Backend Server**:
    ```bash
    cd server
    npm run dev
    ```
    Backend runs on: `http://localhost:8000`

3.  **Start Frontend Client** (in a new terminal):
    ```bash
    cd client
    npm run dev
    ```
    Frontend runs on: `http://localhost:5173`

---

### Default Login Credentials (after seeding)

| Role | Email | Password |
| :--- | :--- | :--- |
| Admin | `admin@pharmaez.com` | `admin123` |
| User | `raj@example.com` | `user123` |
