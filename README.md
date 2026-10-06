# 🥛 Dairy & Milk Parlour POS Management System

[![Tech Stack](https://img.shields.io/badge/Frontend-HTML5%20%7C%20CSS3%20%7C%20Vanilla%20JS-blue)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Backend](https://img.shields.io/badge/Backend-Supabase%20%28PostgreSQL%29-3ECF8E?logo=supabase&logoColor=white)](https://supabase.com)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

An enterprise-grade, lightweight Point of Sale (POS), Inventory Control, Customer CRM, and Sales Analytics web application designed specifically for dairy outlets, milk parlours, and retail dairy counters (**S. Muhammad Rafi Milk Parlar**).

Built as a high-performance Single Page Application (SPA) with zero external build dependencies, backed by **Supabase (PostgreSQL)** for real-time synchronization, secure authentication, and transactional integrity.

---

## 🌟 Key Features

### 1. ⚡ Fast Point of Sale (POS) & Billing
- **Instant Product Search**: Filter items by name or category with responsive tile grids.
- **Dynamic Cart Management**: Real-time total calculations, quantity adjustments, and stock limit validations.
- **Custom Price Overrides**: Modify line-item prices on the fly for specific customer negotiations without altering master product prices.
- **Flexible Payment Methods**: Full support for **Cash**, **UPI / Online**, and **Credit (Udhaar / Khata)**.
- **Discounts & Bill Notes**: Apply flat discounts and manage walk-in or registered customer billing.
- **Automatic Stock Deduction**: Atomically decrements inventory upon successful bill creation via database RPC functions.

### 2. 🧾 Thermal Receipt & Invoice Printing
- **Print-Ready Thermal Receipts**: Tailored layout supporting standard 58mm / 80mm thermal receipt printers as well as A4 print formats.
- **Store Branding**: Header displays store name, phone number, address, timestamp, sequential bill number, and customer info.
- **Credit Notice**: Automatically tags pending credit invoices with payment reminders.

### 3. 📦 Inventory & Stock Management
- **Catalog Management**: Add, update, and manage dairy products with units (`Ltr`, `Kg`, `Pkt`, etc.), selling prices, and categories (`Milk`, `Curd`, `Ghee`, `Paneer`, `Sweets`, etc.).
- **Low Stock & Out-of-Stock Alerts**: Visual status indicators (`In Stock`, `Low Stock`, `Out of Stock`) driven by configurable minimum threshold alerts (`min_stock`).
- **Quick Stock Adjustments**: Fast increment/decrement modal with automatic stock validation.

### 4. 👥 Customer Management & Khata (Udhaar Tracking)
- **Customer Directory**: Store contact details, delivery addresses, and customer notes.
- **Customer 360° Profile**: View total order count, total lifetime spend, outstanding credit balance, and complete order history.
- **Credit Settlement**: Settle pending bills with a dedicated "Mark as Paid" modal supporting Cash, UPI, or other methods.
- **Safe Bill Rollback**: Deleting a bill safely restores all sold item quantities back into the product stock.

### 5. 📊 Real-Time Analytics & Reports
- **Executive Dashboard**: Key performance indicators including Today's Sales, Today's Bill Count, Pending Credit, and Out-of-Stock alerts.
- **Sales Performance Tracking**: Filter sales by date range (Today, Last 7 Days, Month-to-Date).
- **Payment Method Distribution**: Breakdown of revenue collected across Cash vs. UPI vs. Credit.
- **Product Revenue Ranking**: Identify top-selling items by quantity and revenue contribution.
- **Inventory Valuation**: Calculate total stock value in real time.

---

## 🛠️ Tech Stack

- **Frontend**: Pure HTML5, CSS3 (Modern custom design system with CSS variables and responsive grid), Vanilla JavaScript (ES6+ async/await).
- **Database & Auth**: [Supabase](https://supabase.com) (PostgreSQL 15+, Supabase JS v2 client, Row Level Security, RPC stored procedures).
- **Typography & Icons**: Inter Font (Google Fonts), SVG icons.

---

## 📂 Project Structure

```text
├── README.md               # Project documentation
├── milk-business.html      # Main application (Auth, POS, Inventory, Customers, Reports)
└── milk-business1.html     # Backup / distribution copy
```

---

## 🚀 Getting Started

### Prerequisites
- Modern web browser (Chrome, Edge, Firefox, Safari).
- A [Supabase](https://supabase.com) account (for hosting the PostgreSQL database).

### 1. Database Setup (Supabase SQL Editor)
Run the following SQL script in your Supabase SQL editor to create the required tables and stored procedures:

```sql
-- 1. Products Table
CREATE TABLE products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  price NUMERIC(10, 2) NOT NULL DEFAULT 0,
  stock NUMERIC(10, 2) NOT NULL DEFAULT 0,
  min_stock NUMERIC(10, 2) NOT NULL DEFAULT 5,
  category TEXT DEFAULT 'Other',
  unit TEXT DEFAULT '',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 2. Customers Table
CREATE TABLE customers (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  phone TEXT,
  address TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 3. Settings Table
CREATE TABLE settings (
  id INT PRIMARY KEY DEFAULT 1,
  shop_name TEXT NOT NULL DEFAULT 'S. MUHAMMAD RAFI MILK PARLAR',
  address TEXT,
  phone TEXT,
  upi_id TEXT,
  next_bill_no INT DEFAULT 1001
);

INSERT INTO settings (id, shop_name, next_bill_no)
VALUES (1, 'S. MUHAMMAD RAFI MILK PARLAR', 1001)
ON CONFLICT (id) DO NOTHING;

-- 4. Bills Table
CREATE TABLE bills (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  bill_no INT NOT NULL,
  customer TEXT NOT NULL DEFAULT 'Walk-in',
  customer_id UUID REFERENCES customers(id) ON DELETE SET NULL,
  subtotal NUMERIC(10, 2) NOT NULL DEFAULT 0,
  discount NUMERIC(10, 2) NOT NULL DEFAULT 0,
  total NUMERIC(10, 2) NOT NULL DEFAULT 0,
  payment TEXT NOT NULL DEFAULT 'Cash',
  status TEXT NOT NULL DEFAULT 'PAID',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 5. Bill Items Table
CREATE TABLE bill_items (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  bill_id UUID REFERENCES bills(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id) ON DELETE SET NULL,
  name TEXT NOT NULL,
  price NUMERIC(10, 2) NOT NULL,
  qty NUMERIC(10, 2) NOT NULL,
  unit TEXT DEFAULT '',
  subtotal NUMERIC(10, 2) NOT NULL
);

-- 6. RPC: Increment Bill Number Atomically
CREATE OR REPLACE FUNCTION next_bill_no()
RETURNS INT AS $$
DECLARE
  v_bill_no INT;
BEGIN
  SELECT next_bill_no INTO v_bill_no FROM settings WHERE id = 1 FOR UPDATE;
  UPDATE settings SET next_bill_no = next_bill_no + 1 WHERE id = 1;
  RETURN v_bill_no;
END;
$$ LANGUAGE plpgsql;

-- 7. RPC: Decrement Product Stock Atomically
CREATE OR REPLACE FUNCTION decrement_stock(p_product_id UUID, p_qty NUMERIC)
RETURNS VOID AS $$
BEGIN
  UPDATE products
  SET stock = GREATEST(0, stock - p_qty)
  WHERE id = p_product_id;
END;
$$ LANGUAGE plpgsql;
```

### 2. Configure Credentials
In `milk-business.html`, update the Supabase project configuration:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_PROJECT_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_PUBLIC_KEY';
```

### 3. Running Locally
Simply open `milk-business.html` in your web browser:
- Double-click `milk-business.html`, or
- Use a local development server (e.g., Live Server in VS Code or `npx serve .`).

---

## 🔒 Authentication

The application uses Supabase Auth for admin access. Create an admin user under **Authentication > Users** in your Supabase dashboard and log in with your email and password.

---

## 📄 License

This project is licensed under the MIT License.
