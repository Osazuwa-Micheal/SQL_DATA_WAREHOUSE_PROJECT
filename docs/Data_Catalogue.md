# Data Catalogue — Gold Layer

**Author:** Osazuwa Micheal Kelvin  
**Project:** SQL Data Warehouse — Gold Layer  
**Purpose:**  
The **Gold Layer** provides **business-ready, analytics-optimized data models** derived from the **cleaned and validated Silver Layer**.  
It consists of **Dimension Tables** for descriptive context and **Fact Tables** for transactional and measurable data — forming a **Star Schema** for BI and reporting.

---

## 1. `gold.dim_customers`

### **Purpose**
Provides a unified and enriched view of all customers, combining CRM and ERP data sources.  
This dimension contains personal details, demographics, and geographic information for analytical segmentation.

### **Source Tables**
- `silver.crm_cust_info`
- `silver.erp_cust_az12`
- `silver.erp_loc_a101`

### **Primary Key**
`customer_key` — Surrogate key generated via `ROW_NUMBER()`.

### **Schema Definition**

| Column Name | Data Type | Description |
|--------------|------------|-------------|
| `customer_key` | INT | Surrogate key for each customer (Primary Key). |
| `customer_id` | INT | Original numeric customer ID from CRM. |
| `customer_number` | VARCHAR | Customer reference number from CRM. |
| `firstname` | VARCHAR | Customer’s first name. |
| `lastname` | VARCHAR | Customer’s last name. |
| `country` | VARCHAR | Customer’s country of residence from ERP location data. |
| `marital_status` | VARCHAR | Marital status (e.g., Single, Married). |
| `gender` | VARCHAR | Gender, derived from CRM or ERP. |
| `birthdate` | DATE | Customer’s date of birth (from ERP system). |
| `date_created` | DATE | Date the customer record was created in CRM. |

---

## 2. `gold.dim_products`

### **Purpose**
Maintains a consistent and descriptive catalog of products across CRM and ERP sources, including category, cost, and maintenance data.

### **Source Tables**
- `silver.crm_prd_info`
- `silver.erp_px_cat_g1v2`

### **Primary Key**
`product_key` — Surrogate key generated via `ROW_NUMBER()`.

### **Schema Definition**

| Column Name | Data Type | Description |
|--------------|------------|-------------|
| `product_key` | INT | Surrogate key for each product (Primary Key). |
| `product_id` | INT | Unique product ID from CRM. |
| `product_number` | VARCHAR | Product reference key. |
| `category_id` | TEXT | Category identifier linking to ERP product category. |
| `category` | VARCHAR | Main product category. |
| `sub_category` | VARCHAR | Subcategory under the main category. |
| `maintenance` | VARCHAR | Type or requirement of maintenance for the product. |
| `product_name` | VARCHAR | Name of the product. |
| `product_line` | VARCHAR | Product line or business segment. |
| `product_cost` | INT | Cost of the product. |
| `product_start_date` | DATE | Date the product became available. |

**Filter Applied:**  
`prd_end_dt IS NULL` — filters out all historical data, leaving only active products.

---

## 3. `gold.fact_sales`

### **Purpose**
Stores measurable sales transactions linking customers and products.  
Serves as the **central fact table** for analytical reporting and dashboard metrics.

### **Source Tables**
- `silver.crm_sales_details`
- `gold.dim_customers`
- `gold.dim_products`

### **Primary Key**
Composite key: (`sls_ord_num`, `product_key`, `customer_key`)

### **Schema Definition**

| Column Name | Data Type | Description |
|--------------|------------|-------------|
| `sls_ord_num` | VARCHAR | Unique sales order identifier. |
| `product_key` | INT | Foreign key referencing `gold.dim_products`. |
| `customer_key` | INT | Foreign key referencing `gold.dim_customers`. |
| `sls_order_dt` | DATE | Date when the order was placed. |
| `sls_due_dt` | DATE | Payment or delivery due date. |
| `sls_sales` | NUMERIC | Total revenue for the order. |
| `sls_quantity` | INT | Number of product units sold. |
| `sls_price` | NUMERIC(10,2) | Price per unit of product. |

### **Relationships**
- `fact_sales.product_key` → `dim_products.product_key`
- `fact_sales.customer_key` → `dim_customers.customer_key`

---

## Schema Overview

| Layer | Object Type | Object Name | Description |
|--------|--------------|--------------|--------------|
| `gold` | Dimension | `dim_customers` | Customer master table combining CRM and ERP data. |
| `gold` | Dimension | `dim_products` | Product catalog with category and maintenance details. |
| `gold` | Fact | `fact_sales` | Sales transaction data for reporting and analytics. |

---



