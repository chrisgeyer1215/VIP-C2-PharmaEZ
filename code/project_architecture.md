# PharmaEZ — Pharmacy & Healthcare E-Commerce Project Architecture

This document serves as the central index for the **PharmaEZ MERN Stack Pharmacy & Healthcare E-Commerce Application Architecture**. Click on the links below to view the detailed document for each architectural component.

---

## Architecture Sub-topics

### 1. [Technical Architecture](docs/technical_architecture.md)
*   **Description**: Conceptual decoupling of client-server application flows using MongoDB, Express, React, and Node.js — tailored for a pharmacy and healthcare domain with prescription verification and inventory management.
*   **File Path**: [docs/technical_architecture.md](docs/technical_architecture.md)

---

### 2. [Entity Relationship (ER) Diagram](docs/er_diagram.md)
*   **Description**: High-level database schema relationships, collections structures, and reference points — covering Users, Products, Carts, Orders, Prescriptions, and Admin Configuration modelled via Mongoose.
*   **File Path**: [docs/er_diagram.md](docs/er_diagram.md)

---

### 3. [E-Commerce Features](docs/features.md)
*   **Description**: Complete specification of customer-facing functionalities (authentication, browsing, prescription upload, checkout) and administration features (order management, inventory, prescription review, analytics).
*   **File Path**: [docs/features.md](docs/features.md)

---

### 4. [Roles and Responsibilities](docs/roles_responsibilities.md)
*   **Description**: Role-Based Access Control (RBAC) matrix defining Guest, Registered User, and Admin permissions — including JWT `protect` and `adminOnly` middleware implementations.
*   **File Path**: [docs/roles_responsibilities.md](docs/roles_responsibilities.md)

---

### 5. [User Flows](docs/user_flow.md)
*   **Description**: Interactive sequence diagrams mapping the checkout process, prescription upload workflow, and admin order management flow — between the React client, Express API, MongoDB, and external services.
*   **File Path**: [docs/user_flow.md](docs/user_flow.md)

---

### 6. [MVC Pattern in MERN](docs/mvc_pattern.md)
*   **Description**: Deep-dive on how the classic Model-View-Controller design maps across the decoupled PharmaEZ MERN codebase — covering all 5 models, 5 controllers, and the routing layer.
*   **File Path**: [docs/mvc_pattern.md](docs/mvc_pattern.md)

---

[◄ Back to Home](README.md)
