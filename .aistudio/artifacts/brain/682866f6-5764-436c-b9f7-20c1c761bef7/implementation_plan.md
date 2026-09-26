# StockSense - Smart Inventory Management System

A production-grade mobile inventory management platform inspired by Odoo Inventory, built natively with Kotlin, Jetpack Compose, and Room local persistence. It equips warehouse managers and staff with end-to-end stock control, real-time KPI analytics, audit-grade stock ledgers, and streamlined inventory movements.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> **Confirmed Choices from Phase 1:**
> - **Platform:** Native Android application using modern Kotlin and Jetpack Compose with Material 3.
> - **Data Layer:** Local Room database providing complete offline persistence, instant responsiveness, and atomic stock transactions.
> - **Architecture:** Clean MVVM with Repository pattern, Kotlin Coroutines, StateFlow, and preloaded sample inventory for immediate testing.

- **Confirmed Decision 1**: Native Jetpack Compose implementation adapted from the MERN hackathon specification into an enterprise-grade mobile inventory client.
- **Confirmed Decision 2**: Room Database schema covering `Users`, `Categories`, `Products`, `StockTransactions`, and `Locations` with reactive `Flow` queries.
- **Open Decision (Defaulting to Recommended)**: Preload realistic enterprise demo data (Electronics, Hardware, Office Supplies, and historical stock movements) with a toggleable role switcher (`Admin`, `Inventory Manager`, `Warehouse Operator`) on the profile screen.

---

## 1. Overview & Core Concept

StockSense provides real-time visibility into inventory levels, incoming receipts, outgoing delivery dispatches, internal inter-warehouse transfers, and audit adjustments. 

### Target Audience & Personas
- **Warehouse Managers / Admins**: Need high-level KPI oversight (total asset valuation, turnover velocity, critical low-stock warnings) and category-level configurations.
- **Inventory Operators & Floor Staff**: Need quick, thumb-friendly interfaces for receiving shipments, picking and validating deliveries, logging internal bin/warehouse transfers, and reconciling physical inventory counts.

### Key Value Delivered
- **Zero Stock Drift**: Automated transaction journaling ensures every quantity modification (Receipt, Delivery, Transfer, Adjustment) updates product counts and writes an immutable audit record to the Stock Ledger.
- **Rapid Decision Making**: Color-coded stock level badges (Normal, Low Stock, Critical) and real-time Material 3 analytics charts visualize inventory velocity and replenishment priorities.

---

## 2. User Experience & Visual Design

### Key User Flows
1. **Authentication & Profile**:
   - Splash / Welcome screen with branded StockSense identity.
   - Login / Signup with email, role selection (`Admin`, `Manager`, `Staff`), and password recovery simulation.
   - Profile drawer / view with active role indicator, system stats, data reset, and demo account switching.
2. **Executive Dashboard**:
   - Header with active warehouse/facility selector and user badge.
   - 4 Hero KPI Cards: Total Products, Total Categories, Total Inventory Valuation ($), Low Stock Alerts count.
   - Interactive Stock Movement Chart (Canvas-rendered 7-day inflow vs. outflow bar/area trend).
   - Quick Action Floating Pills: "Receive", "Dispatch", "Transfer", "Adjust".
   - Recent Transactions list with instant drill-down.
3. **Product & Category Hub**:
   - Searchable, filterable catalog with category chips (`All`, `Electronics`, `Raw Materials`, `Apparel`, etc.).
   - Card layout showing SKU, Barcode icon, current stock vs reorder threshold, unit price, and status chip.
   - Create & Edit Product modal/screen with validation (SKU uniqueness, positive price, reorder alerts).
   - Category management modal.
4. **Operations (Receipts, Deliveries, Transfers, Adjustments)**:
   - **Receipts**: Add incoming stock with supplier invoice reference, auto-incrementing stock.
   - **Deliveries**: Dispatch outgoing stock with customer/order reference; blocks negative stock overdraws.
   - **Transfers**: Move quantities between locations (e.g., *Main Warehouse* ➔ *Production Line* ➔ *Retail Store*).
   - **Adjustments**: Physical count reconciliation with reason code (Damage, Lost, Audit Recount, Initial Count).
5. **Stock Ledger & Audit Trail**:
   - Comprehensive log of every movement with timestamp, type badge, SKU, quantity delta (+/-), reference number, and operator name.
   - Filter by movement type (`All`, `Receipt`, `Delivery`, `Transfer`, `Adjustment`) and search by reference/product.

### Visual Identity & Theme
- **Aesthetic Direction**: *Enterprise Utilitarian Polish* inspired by Odoo 17/18 and modern Material Design 3. Clean, high-density, card-based layouts with clear status hierarchy.
- **Color Palette**:
  - Primary: Deep Odoo Purple (`#714B67` / `#875A7B`) with vivid Amethyst accent (`#A24689`).
  - Secondary / Operations: Forest Emerald for Receipts (`#10B981`), Amber Orange for Adjustments/Transfers (`#F59E0B`), Crimson for Deliveries/Low Stock (`#EF4444`), Deep Slate Navy for Dark/Light surfaces.
  - Background: Crisp off-white `#F8FAFC` (Light) / Elevated Gunmetal `#121826` (Dark).
- **Typography & Layout**:
  - Clean M3 typography scale with tabular numerals for SKU codes, quantities, and currency amounts.
  - Strict 8dp grid spacing, 48dp touch targets, and subtle elevation cards.

---

## 3. Key Product Decisions & Trade-Offs

| Decision | Chosen Approach | Rationale & Trade-Off |
| :--- | :--- | :--- |
| **Persistence Engine** | Room SQLite Database with Flow queries | Instant offline responsiveness, zero latency, atomic transactions, no network dependencies during hackathon demos. |
| **Navigation Model** | Single-Activity with State-driven Tab/Screen Navigation & `BackHandler` | Clean backstack control, smooth transitions, instant tab switching between Dashboard, Products, Operations, and Ledger. |
| **Stock Ledger Integrity** | Strict Append-Only Transaction Log linked with Product Table | Every operational event updates the product balance and generates a timestamped `StockTransaction` entity to prevent discrepancy. |
| **Analytics Visualization** | Custom Jetpack Compose Canvas Bar & Area Chart | Highly responsive, zero external web/D3 overhead, smoothly animated bars with custom tooltips. |

---

## 4. Technical Architecture & Data Strategy

```
┌────────────────────────────────────────────────────────────────────────┐
│                        StockSense App UI Layer                         │
├─────────────┬─────────────────┬───────────────────┬────────────────────┤
│ Auth & Nav  │ Dashboard & KPI │ Products & Cat.   │ Operations & Ledger│
│ - Login     │ - 4 Stat Cards  │ - Product List    │ - Receipts / Inflow│
│ - Signup    │ - Trend Canvas  │ - Add/Edit Dialog │ - Deliveries / Out │
│ - Profile   │ - Quick Actions │ - Category Filter │ - Transfers / Move │
│ - Roles     │ - Recent Stream │ - Low Stock Filter│ - Audit Adjustment │
└──────┬──────┴────────┬────────┴─────────┬─────────┴─────────┬──────────┘
       │               │                  │                   │
       ▼               ▼                  ▼                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      StockSenseViewModel (MVVM)                        │
│  Exposes StateFlow<StockUiState> (DashboardStats, Products, Ledger)    │
│  Methods: login(), addProduct(), receiveStock(), dispatchStock(), etc. │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        InventoryRepository                             │
│  Coordinates atomic Room operations between Products and Ledger Logs   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                       Room Database (AppDatabase)                      │
├─────────────────┬───────────────────┬─────────────────┬────────────────┤
│   UserEntity    │   ProductEntity   │ CategoryEntity  │ TransactionLog │
│ (id, email,     │ (id, name, sku,   │ (id, name,      │ (id, productId,│
│  role, name)    │  catId, qty,      │  description)   │  type, qty,    │
│                 │  price, reorder)  │                 │  ref, time)    │
└─────────────────┴───────────────────┴─────────────────┴────────────────┘
```

### Data Entities & Schema
1. **UserEntity**: `id`, `name`, `email`, `passwordHash`, `role` (Admin, Manager, Staff), `avatarColor`.
2. **CategoryEntity**: `id`, `name`, `description`, `iconName`.
3. **ProductEntity**: `id`, `name`, `sku`, `categoryId`, `quantity`, `unitPrice`, `reorderLevel`, `barcode`, `location`.
4. **StockTransactionEntity**: `id`, `productId`, `productName`, `productSku`, `type` (`RECEIPT`, `DELIVERY`, `TRANSFER`, `ADJUSTMENT`), `quantityDelta`, `sourceLocation`, `targetLocation`, `reference`, `notes`, `performedBy`, `timestamp`.

### Verification Plan
- **Build Verification**: Compile with `compile_applet` to ensure zero compilation or KSP errors.
- **Persistence Verification**: Seed realistic starter records (6 categories, 12 products across various stock levels, 15 historical transactions) on first boot.
- **Workflow Verification**:
  - Test receiving +25 units of an item -> verify ledger entry and updated product count.
  - Test dispatching units -> verify stock decrement and warning if insufficient stock.
  - Test internal transfer between Main Warehouse and Production.
  - Test stock adjustment for physical inventory recount.
