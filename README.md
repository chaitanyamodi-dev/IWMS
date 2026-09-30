# IWMS
Inventory and Warehouse Management System built with Django REST Framework
# 📦 Inventory & Warehouse Management System (IWMS) API

A **REST API for managing products, multi-warehouse stock, purchasing, sales and inter-warehouse transfers.** Built with **Django** and **Django REST Framework**, secured with **JWT authentication**, and documented with **Swagger / OpenAPI**. 🚀

---

## 📑 Table of Contents

* ✨ [Features](#-features)
* 🛠️ [Tech Stack](#️-tech-stack)
* 📁 [Project Structure](#-project-structure)
* 🗄️ [Data Model](#️-data-model)
* 🚀 [Getting Started](#-getting-started)
* 🔐 [Authentication](#-authentication)
* 📚 [API Documentation](#-api-documentation)
* 🔌 [API Endpoints](#-api-endpoints)
* 🔄 [Business Workflows](#-business-workflows)
* 👥 [Roles & Permissions](#-roles--permissions)
* 🧪 [Running Tests](#-running-tests)
* 🗺️ [Roadmap](#️-roadmap)

---

## ✨ Features

* 📋 **Catalog management**: categories, products (unique SKU, price, reorder level) and warehouses
* 🏢 **Per-warehouse inventory** with on-hand and reserved quantities
* 📦 **Stock operations**: stock in, stock out and manual adjustments, each recorded as a `StockMovement` for a full audit trail
* 🛒 **Purchase orders**: draft → confirmed → partially received → received
* 🧾 **Sales orders**: draft → confirmed → reserved → partially fulfilled → completed
* 🔄 **Warehouse transfers**: draft → confirmed → in transit → received
* ⚠️ **Low-stock alerts** based on each product's reorder level
* 📊 **Reports**: inventory, purchases, sales and stock movements
* 🔒 **Concurrency safety**: services use `transaction.atomic` and `select_for_update` to prevent race conditions
* 🔎 **Search, filtering, ordering and pagination** on list endpoints
* 🔑 **JWT authentication** with an interactive Swagger UI

---

## 🛠️ Tech Stack

| Layer                | Technology                             |
| -------------------- | -------------------------------------- |
| 🐍 Language          | Python 3.13                            |
| 🌐 Framework         | Django 6.1, Django REST Framework 3.18 |
| 🔐 Authentication    | djangorestframework-simplejwt 5.5      |
| 📚 API Documentation | drf-spectacular 0.30 (OpenAPI 3)       |
| 🔎 Filtering         | django-filter 26.1                     |
| 🗄️ Database         | SQLite (development)                   |

---

## 📁 Project Structure

```text
inventory_management/
├── inventory/                    # 📦 Main application
│   ├── migrations/               # 🗄️ Database migrations (0001 – 0010)
│   ├── admin.py                  # ⚙️ Django admin registrations
│   ├── apps.py
│   ├── models.py                 # 🧩 Domain models
│   ├── permissions.py            # 🔐 Role-based permission classes
│   ├── serializers.py            # ✅ Validation and serialization
│   ├── services.py               # ⚙️ Business logic (stock, orders, transfers)
│   ├── tests.py                  # 🧪 Tests
│   ├── urls.py                   # 🔗 App routes
│   └── views.py                  # 🌐 API views
├── inventory_management/         # ⚙️ Project configuration
│   ├── settings.py
│   ├── urls.py                   # 🔗 Root routes, auth and docs
│   ├── asgi.py
│   └── wsgi.py
├── db.sqlite3                    # 🗄️ Local development database
├── manage.py                     # 🚀 Django management utility
├── requirements.txt              # 📋 Project dependencies
└── schema.yml                    # 📚 Exported OpenAPI schema
```

### 🏗️ Architecture

**Views** handle HTTP requests and validation, **serializers** validate input, and `services.py` holds the business rules and database transactions.

This keeps stock logic in one place and makes it reusable across multiple endpoints. 🔄

---

## 🗄️ Data Model

| Model                                 | Purpose                                                       |
| ------------------------------------- | ------------------------------------------------------------- |
| `Category`                            | 🏷️ Product grouping (unique name)                            |
| `Product`                             | 📦 Item with unique SKU, price and reorder level              |
| `Warehouse`                           | 🏢 Storage location with manager and active flag              |
| `Inventory`                           | 📊 Quantity and reserved quantity of a product in a warehouse |
| `StockMovement`                       | 📝 Immutable IN/OUT ledger entry tied to an inventory record  |
| `Supplier`                            | 🚚 Vendor contact details                                     |
| `PurchaseOrder` / `PurchaseOrderItem` | 🛒 Inbound orders and their line items                        |
| `SalesOrder` / `SalesOrderItem`       | 🧾 Outbound orders and their line items                       |
| `TransferOrder` / `TransferOrderItem` | 🔄 Warehouse-to-warehouse transfers                           |

---

## 🚀 Getting Started

### 📌 Prerequisites

* 🐍 Python 3.13
* 📦 pip

### ⚙️ Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>/inventory_management

# 2. Create and activate a virtual environment
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS / Linux

# 3. Install dependencies
pip install -r requirements.txt

# 4. Apply migrations
python manage.py migrate

# 5. Create an admin user
python manage.py createsuperuser

# 6. Start the server
python manage.py runserver
```

🎉 The API is now available at:

```text
http://127.0.0.1:8000/api/
```

---

## 🔐 Authentication

The API uses **JWT bearer tokens**.

* ⏱️ Access token: 2 hours
* 🔄 Refresh token: 1 day

### 🔑 Obtain a Token Pair

```bash
curl -X POST http://127.0.0.1:8000/api/auth/token/ \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your-password"}'
```

### 🔓 Use the Access Token

```bash
curl http://127.0.0.1:8000/api/products/ \
  -H "Authorization: Bearer <access_token>"
```

🔄 Refresh an expired access token via:

```text
POST /api/auth/token/refresh/
```

---

## 📚 API Documentation

| URL               | Description                                               |
| ----------------- | --------------------------------------------------------- |
| 📖 `/api/docs/`   | Swagger UI — use **Authorize** to paste your access token |
| 📄 `/api/schema/` | Raw OpenAPI schema                                        |
| ⚙️ `/admin/`      | Django admin                                              |

### 🔄 Regenerate OpenAPI Schema

```bash
python manage.py spectacular --file schema.yml
```

---

## 🔌 API Endpoints

All routes are prefixed with `/api/`.

### 📋 Catalog

| Method                  | Endpoint            | Description                 |
| ----------------------- | ------------------- | --------------------------- |
| GET, POST               | `/categories/`      | 📋 List / create categories |
| GET, PUT, PATCH, DELETE | `/categories/{id}/` | 🏷️ Category detail         |
| GET, POST               | `/warehouses/`      | 🏢 List / create warehouses |
| GET, PUT, PATCH, DELETE | `/warehouses/{id}/` | 🏢 Warehouse detail         |
| GET, POST               | `/products/`        | 📦 List / create products   |
| GET, PUT, PATCH, DELETE | `/products/{id}/`   | 📦 Product detail           |

### 📦 Inventory & Stock

| Method                  | Endpoint                   | Description                        |
| ----------------------- | -------------------------- | ---------------------------------- |
| GET, POST               | `/inventory/`              | 📊 List / create inventory records |
| GET, PUT, PATCH, DELETE | `/inventory/{id}/`         | 📊 Inventory detail                |
| GET                     | `/inventory/{id}/history/` | 📝 Movement history                |
| POST                    | `/stock/in/`               | ➕ Add stock                        |
| POST                    | `/stock/out/`              | ➖ Remove stock                     |
| POST                    | `/stock-adjustment/`       | 🔧 Manual adjustment               |
| GET                     | `/stock-movements/`        | 📜 Stock movement ledger           |
| GET                     | `/low-stock/`              | ⚠️ Low-stock items                 |

### 🛒 Purchase Orders

| Method    | Endpoint                         | Description                      |
| --------- | -------------------------------- | -------------------------------- |
| GET, POST | `/purchase-orders/`              | 📋 List / create purchase orders |
| GET       | `/purchase-orders/{id}/`         | 🔍 Purchase order detail         |
| POST      | `/purchase-orders/{id}/confirm/` | ✅ Confirm a draft order          |
| POST      | `/purchase-orders/{id}/receive/` | 📦 Receive items into warehouse  |

### 🧾 Sales Orders

| Method    | Endpoint                      | Description                   |
| --------- | ----------------------------- | ----------------------------- |
| GET, POST | `/sales-orders/`              | 📋 List / create sales orders |
| GET       | `/sales-orders/{id}/`         | 🔍 Sales order detail         |
| POST      | `/sales-orders/{id}/confirm/` | ✅ Confirm a draft order       |
| POST      | `/sales-orders/{id}/reserve/` | 🔒 Reserve stock              |
| POST      | `/sales-orders/{id}/fulfill/` | 📦 Fulfill items              |

### 🔄 Transfers

| Method | Endpoint                   | Description         |
| ------ | -------------------------- | ------------------- |
| POST   | `/transfers/`              | ➕ Create a transfer |
| GET    | `/transfers/list/`         | 📋 List transfers   |
| GET    | `/transfers/{id}/`         | 🔍 Transfer detail  |
| POST   | `/transfers/{id}/items/`   | 📦 Add an item      |
| POST   | `/transfers/{id}/confirm/` | ✅ Confirm transfer  |
| POST   | `/transfers/{id}/ship/`    | 🚚 Ship transfer    |
| POST   | `/transfers/{id}/receive/` | 📥 Receive transfer |

### 📊 Reports

| Method | Endpoint                    | Description                              |
| ------ | --------------------------- | ---------------------------------------- |
| GET    | `/reports/inventory/`       | 📦 Stock levels by warehouse and product |
| GET    | `/reports/purchase/`        | 🛒 Purchase orders with totals           |
| GET    | `/reports/sales/`           | 🧾 Sales orders with totals              |
| GET    | `/reports/stock-movements/` | 📜 Movement ledger                       |

📄 List endpoints are paginated:

```text
?page=N
```

Page size: **5**

---

## 🔄 Business Workflows

### 🛒 Purchase Order

```text
DRAFT → CONFIRMED → PARTIALLY_RECEIVED → RECEIVED
```

📦 Receiving increases warehouse stock, records a `StockMovement`, and prevents over-receiving.

### 🧾 Sales Order

```text
DRAFT → CONFIRMED → RESERVED → PARTIALLY_FULFILLED → COMPLETED
```

🔒 Reserving stock earmarks quantity in `reserved_quantity`.

📦 Fulfilling releases the reservation, deducts on-hand stock and logs the movement.

### 🔄 Transfer Order

```text
DRAFT → CONFIRMED → IN_TRANSIT → RECEIVED
```

🚚 Shipping deducts stock from the source warehouse.

📥 Receiving adds stock to the destination warehouse.

---

## 🧪 Example: Create a Purchase Order

```json
POST /api/purchase-orders/

{
  "po_number": "PO-1001",
  "supplier": 1,
  "warehouse": 1,
  "items": [
    {
      "product": 3,
      "quantity": 100,
      "unit_price": "45.50"
    }
  ]
}
```

---

## 👥 Roles & Permissions

The system defines three permission groups:

* 👑 **ADMIN**
* 🧑‍💼 **MANAGER**
* 👷 **STAFF**

Permission classes are defined in `permissions.py`.

Catalog and order endpoints use Django model permissions:

* 👀 Authenticated users can read
* ✏️ Write access requires the matching model permission
* ⚙️ Users can be assigned to groups through Django Admin

---

## 🧪 Running Tests

Run the Django test suite with:

```bash
python manage.py test
```

---

## 🗺️ Roadmap

* [ ] 🧪 Expand the automated test suite (services, order workflows, permissions)
* [ ] ❌ Order cancellation flows that release reserved stock
* [ ] 🚚 Supplier CRUD endpoints
* [ ] 🔐 Apply role-based permissions to stock, order and report endpoints
* [ ] ⚙️ Environment-based settings (`SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`) for deployment
* [ ] 🐘 Switch to PostgreSQL for production

---

## 📄 License

Add your license here (e.g. **MIT**).

