# Roles and Responsibilities

This document defines the Role-Based Access Control (RBAC) rules mapping for the **PharmaEZ** Pharmacy & Healthcare E-Commerce application.

---

## RBAC Matrix

| Role | Responsibilities | Permissions & Capabilities |
| :--- | :--- | :--- |
| **Guest / Visitor** | Browse the pharmacy catalogue and research products | • View homepage, hero banner, category grid, and offer banners.<br>• Search and filter products by category, price, and prescription requirement.<br>• View full product detail pages (description, usage, reviews).<br>• Add items to a temporary cart (redirected to login at checkout). |
| **Registered User** | Manage their health profile and place medicine orders | • Register and log in to a personal account.<br>• Manage profile details, delivery addresses, and password.<br>• Add products to a persistent cart and wishlist.<br>• Upload and track doctor prescriptions (image/PDF).<br>• Perform multi-step checkout and complete payment.<br>• View complete order history and real-time order status tracking.<br>• Cancel orders prior to shipment.<br>• Write product reviews and star ratings.<br>• Earn and view loyalty points. |
| **Administrator / Pharmacist** | Manage the full store operations and prescription verification | • Access the secure admin panel at `/admin`.<br>• Create, Read, Update, and Delete (CRUD) healthcare products with pharmacy-specific fields.<br>• Review, approve, or reject customer-uploaded prescriptions.<br>• Manage order fulfilment — update order status with full audit trail.<br>• Activate or deactivate customer accounts.<br>• Monitor inventory levels and receive low-stock alerts.<br>• View sales analytics dashboard (KPIs, revenue chart, order breakdown).<br>• Manage store configuration (shipping thresholds, tax rate, banners). |

---

## Access Control Middleware Design

### Token Verification — `protect` Middleware

JWT tokens are generated upon successful login using `jsonwebtoken`. The backend verifies this token for any protected endpoint using the `protect` middleware in `server/middleware/auth.js`:

```javascript
const protect = async (req, res, next) => {
    let token;

    // Read token from Authorization header
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
        token = req.headers.authorization.split(' ')[1];
    }

    if (!token) {
        return res.status(401).json({ success: false, message: 'Not authorized, no token' });
    }

    try {
        // Verify token signature and expiry
        const decoded = jwt.verify(token, process.env.JWT_SECRET);

        // Attach the authenticated user to the request object
        req.user = await User.findById(decoded.id).select('-password');

        if (!req.user) {
            return res.status(401).json({ success: false, message: 'User not found' });
        }

        next();
    } catch (error) {
        return res.status(401).json({ success: false, message: 'Not authorized, token failed' });
    }
};
```

---

### Role Checks — `adminOnly` Middleware

To protect Admin-only resources (product CRUD, order management, prescription review, inventory), the `adminOnly` middleware is chained immediately after `protect` on all admin routes:

```javascript
const adminOnly = (req, res, next) => {
    if (req.user && req.user.role === 'admin') {
        next();
    } else {
        return res.status(403).json({ success: false, message: 'Admin access required' });
    }
};
```

---

### Token Generation — `generateToken` Utility

```javascript
const generateToken = (id) => {
    return jwt.sign(
        { id },
        process.env.JWT_SECRET,
        { expiresIn: process.env.JWT_EXPIRES_IN || '7d' }
    );
};
```

---

### Route-Level Usage Examples

```javascript
// Public route — no middleware
router.get('/products', getProducts);

// Protected route — logged-in users only
router.post('/orders', protect, placeOrder);
router.get('/orders/my', protect, getMyOrders);
router.post('/auth/prescription', protect, upload.single('prescription'), uploadPrescription);

// Admin-only routes — protect + adminOnly chained
router.post('/products',           protect, adminOnly, upload.array('images', 5), createProduct);
router.put('/products/:id',        protect, adminOnly, updateProduct);
router.delete('/products/:id',     protect, adminOnly, deleteProduct);
router.get('/orders/admin/all',    protect, adminOnly, getAllOrders);
router.put('/orders/admin/:id/status', protect, adminOnly, updateOrderStatus);
router.get('/admin/inventory',     protect, adminOnly, getInventoryReport);
router.put('/admin/prescriptions/:userId/:prescriptionId', protect, adminOnly, updatePrescriptionStatus);
```

---

### Frontend Route Guards

The React frontend enforces access control using wrapper components in `src/App.jsx`:

```jsx
// Redirect unauthenticated users to /login
const ProtectedRoute = ({ children }) => {
    const { isLoggedIn } = useAuth();
    return isLoggedIn ? children : <Navigate to="/login" replace />;
};

// Redirect non-admins to homepage
const AdminRoute = ({ children }) => {
    const { isLoggedIn, isAdmin } = useAuth();
    if (!isLoggedIn) return <Navigate to="/login" replace />;
    if (!isAdmin) return <Navigate to="/" replace />;
    return children;
};

// Redirect already-logged-in users away from auth pages
const PublicRoute = ({ children }) => {
    const { isLoggedIn, isAdmin } = useAuth();
    if (isLoggedIn) return <Navigate to={isAdmin ? '/admin' : '/'} replace />;
    return children;
};
```

---

[◄ Back to Project Architecture](../project_architecture.md) | [Back to Home](../README.md)
