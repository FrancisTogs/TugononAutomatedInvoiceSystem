# 🏗️ Automated Invoicing and Transaction Tracking System
> **Client / Organization:** Tugonon Construction Services  
> **Tech Stack:** Svelte 5 (TypeScript) + Go (Golang) + MySQL  

---

## 📌 System Overview

The **Automated Invoicing and Transaction Tracking System** is an enterprise-grade full-stack web application designed for **Tugonon Construction Services**[cite: 9]. It automates and streamlines client order tracking, project quotation preparation, tax and discount calculations, sequential invoice generation, payment monitoring, and official receipt issuance[cite: 9].

The system features strict **Role-Based Access Control (RBAC)** to separate operational duties between **Admin/Sales** personnel and **Finance/Accounting** personnel.

---

## ✨ Features by Module & Role

### 👥 Admin / Sales Module
* **Client Management (`ClientManager.svelte`):** Register new client accounts, search client records, and manage billing contact information[cite: 9, 10].
* **Orders & Requests (`OrderManager.svelte`):** Record client project requests, track site locations, estimate budgets, and manage project statuses (`New`, `Quotation Created`, `In Progress`, `Completed`)[cite: 9, 10].
* **Project Quotations (`QuotationForm.svelte`):** Build detailed project quotations with dynamic line-item calculators, unit costs, automatic VAT estimations, and target completion dates[cite: 9, 10, 12].

### 💳 Finance / Accounting Module
* **Invoice Management (`InvoiceManager.svelte`):** Convert approved quotations into formal invoices, auto-generate sequential invoice numbers (e.g., `INV-2026-015`), and calculate discounts and withholding tax rates (BIR 2307)[cite: 9, 10, 11].
* **Payment Status Monitoring (`PaymentStatus.svelte`):** Track invoice payment statuses (`Paid`, `Partial`, `Unpaid`, `Overdue`) and record partial or full client payments[cite: 9, 10].
* **Official Receipts (`OfficialReceipts.svelte`):** Generate and issue formal Official Receipts (`OR-2026-001`) with amount-in-words formatting and printable certificate layouts[cite: 9, 10].

---

## 🛠️ Tech Stack

* **Frontend:** Svelte 5 (TypeScript), HTML5, CSS3 (Dark Theme Workspace)
* **Backend:** Go (Golang) with REST API architecture and `net/http`
* **Database:** MySQL (Third Normal Form / 3NF relational design)[cite: 9, 10]

---

## 📁 Repository Structure

```text
tugonon-invoicing-system/
│
├── backend/                       # Go REST API Backend
│   ├── main.go                    # Go entry point & HTTP routes
│   ├── schema.sql                 # MySQL relational schema (3NF)
│   ├── go.mod
│   └── go.sum
│
└── frontend/                      # Svelte 5 Single-Page Application
    ├── src/
    │   ├── lib/
    │   ├── App.svelte             # Main workspace container & Auth router
    │   ├── ClientManager.svelte   # Client CRUD module
    │   ├── OrderManager.svelte    # Order request module
    │   ├── QuotationForm.svelte   # Line-item quotation calculator
    │   ├── InvoiceManager.svelte   # Invoice generator & tax calculator
    │   ├── PaymentStatus.svelte   # Payment tracking module
    │   ├── OfficialReceipts.svelte# Official receipt issuer & previewer
    │   └── main.ts
    ├── package.json
    └── vite.config.ts