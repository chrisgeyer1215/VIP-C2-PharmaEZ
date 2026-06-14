# PharmaEZ E-Commerce Features

This document provides a complete breakdown of all implemented features in the **PharmaEZ** Pharmacy & Healthcare E-Commerce application.

---

## Customer-Facing Features

### 1. Authentication & Security
*   **Sign Up & Sign In**: Register with name, email, phone, and password. Passwords are validated (minimum 6 characters) and hashed with `bcryptjs` before storage. Login issues a signed JWT token.
*   **JWT Security**: Stateless authentication using `jsonwebtoken`. The token is stored in `localStorage` and automatically attached to every API request via an Axios request interceptor.
*   **Profile Management**: Update personal details (name, phone, gender, date of birth), change password (with current password verification), and manage multiple saved delivery addresses with a default flag.
*   **Account Protection**: A global Axios response interceptor detects `401 Unauthorized` responses, clears the stored token, and redirects the user to the login page.

---

### 2. Search & Browsing
*   **Full-Text Search**: Search across product `name`, `genericName`, and `tags` simultaneously using MongoDB regex queries. The search bar is accessible from the persistent top navigation.
*   **Filtering**: Dynamically filter the product catalogue by Category (10 healthcare categories), Price Range (min/max), and Prescription Requirement (OTC / Rx / All).
*   **Sorting**: Sort results by Relevance (featured first), Price: Low to High, Price: High to Low, Top Rated, and Newest First.
*   **Pagination**: Products are paginated (12 per page by default) with numbered page controls and Prev/Next navigation.
*   **Category Navigation**: A persistent category strip in the navbar and the 8-tile category grid on the homepage allow direct browse by healthcare category.
*   **Active Filter Tags**: Applied filters are displayed as dismissible tags at the top of the product listing for clear visual feedback.

---

### 3. Product Discovery
*   **Product Cards**: Each card shows the product image, category label, name, pack size, star rating, current price with MRP strikethrough, discount badge (e.g., "25% OFF"), Rx badge for prescription-required items, and an Add to Cart button.
*   **Product Detail Page**: Full product information — image gallery with thumbnail strip, generic name, salt composition, dosage form, strength, manufacturer, pack size, star rating with review count, price breakdown, Rx prescription notice with upload link, stock status, quantity selector (bounded by available stock), and action buttons (Add to Cart / Buy Now / Wishlist).
*   **Tabbed Content**: The product detail page includes three tabs — Description & Clinical Details, Usage Instructions, and Customer Reviews — for organised information browsing.
*   **Wishlist**: Logged-in users can toggle any product in/out of their personal wishlist from both the product card and the detail page. Wishlist is stored in the database.

---

### 4. Prescription (Rx) Workflow
*   **Rx Product Flagging**: Products requiring a doctor's prescription are clearly marked with an "Rx Required" badge on the product card and a prominent notice on the product detail page.
*   **Prescription Upload**: Users can upload a prescription image (JPG/PNG) or PDF file (max 5MB) from their dashboard. The upload form includes an optional doctor's name field.
*   **Order Linking**: When an order contains Rx items, the `prescriptionStatus` field is automatically set to `pending`, flagging it for pharmacist review before fulfilment.
*   **Status Tracking**: Users can track the status of each uploaded prescription (`pending`, `approved`, `rejected`) from their Prescriptions dashboard page.

---

### 5. Shopping Cart
*   **Persistent Cart**: Cart data is stored in MongoDB and associated with the logged-in user — surviving browser refresh and session changes.
*   **Real-Time Stock Validation**: Adding or updating cart items validates against current stock levels. An error is returned if the requested quantity exceeds available stock.
*   **Quantity Controls**: Users can increase or decrease item quantities directly in the cart. Reducing quantity to zero removes the item.
*   **Rx Item Notice**: The cart page displays a prominent banner when it contains prescription-required medicines, prompting the user to upload a prescription.
*   **Free Shipping Progress Bar**: Dynamically shows how much more the user needs to spend to qualify for free delivery (threshold: ₹499). Shipping charge is ₹49 below the threshold.
*   **Price Breakdown**: The cart summary shows itemised subtotal, shipping charge (FREE if eligible), 5% GST, and the total payable amount. Savings on discounted items are highlighted.

---

### 6. Checkout Flow
*   **Multi-Step Checkout**: A guided 3-step process — (1) Delivery Address, (2) Payment Method, (3) Order Review — with a visual step indicator.
*   **Address Management**: Users can select from their saved addresses or fill in a new address inline. New addresses are saved to the user profile automatically.
*   **Payment Methods**: Supports Cash on Delivery (COD), UPI / QR Code, Credit/Debit Card, Net Banking, and Wallet. (Razorpay/Stripe gateway integration point is marked in the codebase for production.)
*   **Order Review**: The final step shows a complete summary — all items, selected address, payment method, and itemised pricing — before the user confirms placement.
*   **Stock Lock**: Stock levels are verified one final time at order placement. If any item's stock has changed since adding to cart, an error is returned before the order is persisted.

---

### 7. Order Management & Tracking
*   **Order Placement**: On successful checkout, stock is deducted, loyalty points are awarded, and the cart is cleared in a single atomic operation.
*   **Order Confirmation**: A confirmation page shows the auto-generated order number (e.g., `PEZ7829430001`), estimated delivery date (5 business days), payment details, and a prescription upload reminder for Rx orders.
*   **Order History**: Users can view all their past and active orders, each showing order number, date, item preview, total amount, and current status badge.
*   **Order Detail & Tracking**: A visual 5-step progress tracker (Placed → Confirmed → Processing → Shipped → Delivered) shows real-time order status. Full item list, delivery address, payment details, and price summary are displayed.
*   **Order Cancellation**: Users can cancel orders that have not yet been shipped. Stock is automatically restored upon cancellation.
*   **Loyalty Points**: 1 loyalty point is earned per ₹10 spent. Points are credited automatically and displayed in the user dashboard.

---

### 8. Reviews & Ratings
*   **Submit Reviews**: Logged-in users can submit a star rating (1–5) and a written comment on any product from the product detail page's Reviews tab.
*   **One Review Per User**: The backend enforces a single review per user per product — attempting to review twice returns an error.
*   **Aggregate Rating**: The product's overall `rating` and `numReviews` are recalculated and persisted after every new review submission.
*   **Review Display**: Reviews are listed with reviewer name, star rating visualisation, comment, and date.

---

## Admin Management Features

### 1. Business Insights Dashboard
*   **KPI Summary Cards**: At-a-glance cards for Total Orders, Total Revenue (excluding cancelled orders), Total Registered Users, and Total Products.
*   **7-Day Revenue Chart**: A bar chart visualising daily revenue for the last 7 days, built from MongoDB aggregation queries.
*   **Order Status Breakdown**: Horizontal progress bars showing the count and percentage of orders in each status bucket (placed, confirmed, processing, shipped, delivered, cancelled).
*   **Recent Orders Table**: The 5 most recently placed orders with customer name, order number, total, and status badge.
*   **Low-Stock Alert Panel**: Products with stock ≤ 10 units are surfaced on the dashboard with a direct Restock link.

---

### 2. Order Management
*   **All Orders View**: Paginated table of all orders with filters by status (All / Placed / Confirmed / Processing / Shipped / Delivered / Cancelled / Returned) and a search box for order number lookup.
*   **Inline Status Updates**: Admins can change an order's status directly from the table using a dropdown, with every change logged to the `statusHistory` audit trail.
*   **Order Detail Panel**: Clicking an order opens a side panel with complete customer info, delivery address, itemised invoice, payment details, and prescription status.

---

### 3. Product Catalogue Management (CRUD)
*   **Product Listing**: Searchable, paginated product table with image thumbnails, category badge, price, stock count (colour-coded green/amber/red), Rx badge, featured flag, and action buttons.
*   **Create / Edit Products**: A comprehensive form with 4 organised sections — Basic Information, Pricing & Inventory, Medical / Clinical Details (salt composition, dosage form, strength, side effects, usage & storage instructions), and Visibility & Tags. Supports image URL input and toggle controls for `isAvailable` and `isFeatured`.
*   **Delete Products**: Admins can permanently remove a product with a confirmation prompt.
*   **Featured Products**: The `isFeatured` toggle controls which products appear on the homepage's Featured Products section.

---

### 4. Inventory Management
*   **Stock Overview**: Three summary cards — In Stock (>10 units), Low Stock (1–10 units), Out of Stock (0 units) — with full product inventory table below.
*   **Category Stock Report**: A table breaking down product count and total stock units by category.
*   **Low-Stock Alerts**: Products approaching depletion are highlighted with colour-coded stock numbers and direct Edit / Restock links.
*   **Automatic Stock Deduction**: Stock levels are automatically decremented on order placement and restored on order cancellation — no manual intervention needed.

---

### 5. User Management
*   **User Directory**: Searchable, paginated table of all registered customers showing name, email, phone, join date, loyalty points, prescription count, and active/inactive status badge.
*   **Activate / Deactivate Accounts**: Admins can toggle a user's `isActive` status. Deactivated users cannot log in.

---

### 6. Prescription Review & Verification
*   **Prescription Queue**: All uploaded prescriptions are surfaced in the admin dashboard with patient name, email, prescribing doctor, upload date, and current status.
*   **File Review**: Admins can preview prescription images inline or open PDF files in a new browser tab.
*   **Approve / Reject**: Single-click Approve or Reject action with the status change persisted immediately. Rejected prescriptions can include a notes field for feedback.
*   **Status Filtering**: Filter the prescription queue by status (All / Pending / Approved / Rejected) for efficient triage.

---

[◄ Back to Project Architecture](../project_architecture.md) | [Back to Home](../README.md)
