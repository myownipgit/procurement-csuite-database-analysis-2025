# Current Database Schema (2009-2018)

## 📊 Database Overview
- **Total Records**: 71,932 transactions
- **Date Range**: 2009-2018 (10 years)
- **Total Spend**: $509.9M
- **Vendors**: 2,716 unique suppliers
- **Commodities**: 6,569 unique codes
- **Schema Type**: Star schema with 1 fact table and 3 dimension tables

---

## 🏗️ Schema Structure

```sql
-- Current 4-table star schema
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   dim_vendors   │────│ fact_spend_      │────│ dim_commodities │
│                 │    │   analytics      │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              │
                       ┌─────────────┐
                       │  dim_time   │
                       └─────────────┘
```

---

## 📋 Table Specifications

### 1. `fact_spend_analytics` (Central Fact Table)
**Purpose**: Core transactional data with performance metrics
**Records**: 71,932 transactions

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `fact_key` | INTEGER PK | Unique transaction identifier | 1, 2, 3... |
| `vendor_key` | INTEGER FK | Link to vendor dimension | 1001, 1002... |
| `commodity_key` | INTEGER FK | Link to commodity dimension | 2001, 2002... |
| `contract_key` | INTEGER FK | Contract reference (nullable) | 3001, NULL... |
| `time_key` | INTEGER FK | Link to time dimension | 20091201, 20091202... |
| `business_unit_key` | INTEGER FK | Business unit reference | 101, 102... |
| `spend_amount` | REAL | Transaction amount in USD | 1500.00, 25000.50... |
| `transaction_count` | INTEGER | Number of line items | 1, 5, 12... |
| `quantity` | REAL | Quantity purchased | 10.0, 2.5... |
| `unit_price` | REAL | Price per unit | 150.00, 10000.20... |
| `delivery_performance_score` | REAL | OTIF performance (0-100) | 95.5, 87.2... |
| `quality_score` | REAL | Quality rating (0-100) | 98.1, 92.7... |
| `compliance_score` | REAL | Contract compliance (0-100) | 100.0, 85.3... |
| `risk_weighted_spend` | REAL | Risk-adjusted spend value | 1350.00, 22500.45... |
| `esg_weighted_spend` | REAL | ESG-adjusted spend value | 1425.00, 23750.48... |
| `savings_amount` | REAL | Documented savings | 150.00, 2500.00... |
| `discount_amount` | REAL | Discounts applied | 75.00, 1250.00... |
| `source_transaction_id` | TEXT | Original system transaction ID | "TXN-2009-001", "PO-12345"... |
| `load_date` | DATE | Data warehouse load date | 2023-01-15, 2023-02-01... |

---

### 2. `dim_vendors` (Vendor Master Data)
**Purpose**: Supplier information with ESG and risk attributes
**Records**: 2,716 unique vendors

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `vendor_key` | INTEGER PK | Unique vendor identifier | 1001, 1002... |
| `vendor_id` | TEXT | Business vendor ID | "SUPP-001", "V12345"... |
| `vendor_name` | TEXT | Supplier name | "ABC Corp", "Global Solutions Ltd"... |
| `vendor_tier` | TEXT | Strategic classification | "Strategic", "Preferred", "Approved"... |
| `diversity_classification` | TEXT | Diversity status | "Women-Owned", "Minority-Owned", "SME"... |
| `risk_rating` | TEXT | Risk assessment | "Low", "Medium", "High"... |
| `esg_score` | REAL | ESG performance (0-100) | 85.5, 92.1... |
| `country` | TEXT | Supplier country | "United States", "Germany", "China"... |
| `region` | TEXT | Geographic region | "North America", "Europe", "Asia-Pacific"... |
| `effective_start_date` | DATE | Record validity start | 2009-01-01, 2010-06-15... |
| `effective_end_date` | DATE | Record validity end | 2018-12-31, NULL... |
| `is_current_record` | BOOLEAN | Current record flag | TRUE, FALSE |

---

### 3. `dim_commodities` (Category & Product Data)
**Purpose**: Procurement category hierarchy and classifications
**Records**: 6,569 unique commodity codes

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `commodity_key` | INTEGER PK | Unique commodity identifier | 2001, 2002... |
| `commodity_id` | TEXT | Business commodity code | "IT-001", "FACILITY-025"... |
| `commodity_description` | TEXT | Product/service description | "Laptops", "Office Cleaning Services"... |
| `parent_category` | TEXT | Top-level category | "IT Equipment", "Facilities", "Marketing"... |
| `sub_category` | TEXT | Detailed subcategory | "Hardware", "Maintenance", "Digital Advertising"... |
| `business_criticality` | TEXT | Strategic importance | "Critical", "Important", "Standard"... |
| `sourcing_complexity` | TEXT | Sourcing difficulty | "High", "Medium", "Low"... |
| `category_manager` | TEXT | Responsible category manager | "John Smith", "Sarah Johnson"... |
| `effective_start_date` | DATE | Record validity start | 2009-01-01, 2011-03-20... |
| `is_current_record` | BOOLEAN | Current record flag | TRUE, FALSE |

---

### 4. `dim_time` (Time Dimension)
**Purpose**: Date hierarchy for temporal analysis
**Records**: Time dimension covering 2009-2018

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `time_key` | INTEGER PK | Date key (YYYYMMDD format) | 20091201, 20101215... |
| `date_actual` | DATE | Actual date | 2009-12-01, 2010-12-15... |
| `year` | INTEGER | Calendar year | 2009, 2010, 2011... |
| `quarter` | INTEGER | Calendar quarter (1-4) | 1, 2, 3, 4 |
| `month` | INTEGER | Calendar month (1-12) | 1, 2, 3...12 |
| `fiscal_year` | INTEGER | Fiscal year | 2010, 2011, 2012... |
| `fiscal_quarter` | INTEGER | Fiscal quarter (1-4) | 1, 2, 3, 4 |
| `month_name` | TEXT | Month name | "January", "February"... |
| `quarter_name` | TEXT | Quarter label | "Q1", "Q2", "Q3", "Q4" |
| `day_of_week` | TEXT | Day name | "Monday", "Tuesday"... |
| `week_of_year` | INTEGER | Week number (1-53) | 1, 2, 3...53 |

---

## 🔍 Data Quality & Coverage Analysis

### Completeness Assessment
| Table | Field | Coverage | Data Quality |
|-------|--------|----------|--------------|
| `fact_spend_analytics` | Core spend fields | 100% | High |
| `fact_spend_analytics` | Performance scores | 95% | High |
| `fact_spend_analytics` | Savings tracking | 85% | Medium |
| `dim_vendors` | Basic vendor data | 100% | High |
| `dim_vendors` | ESG scores | 92% | Medium |
| `dim_vendors` | Risk ratings | 88% | Medium |
| `dim_commodities` | Category hierarchy | 100% | High |
| `dim_commodities` | Category managers | 75% | Medium |

### Strengths
✅ **Complete transaction history** with full spend coverage
✅ **Robust vendor master** with ESG and risk metrics
✅ **Comprehensive category hierarchy** with business context
✅ **Strong performance tracking** for delivery, quality, compliance
✅ **Geographic and diversity** classifications for suppliers
✅ **Time dimension** supporting fiscal and calendar analysis

### Limitations
❌ **No contract lifecycle tracking** (contract_key present but not populated)
❌ **Limited real-time capabilities** (historical data only)
❌ **No incident or event tracking** beyond performance scores
❌ **Missing procurement process metrics** (cycle times, approval flows)
❌ **No external market intelligence** integration points
❌ **Limited supplier relationship depth** (basic classification only)

---

## 📊 Sample Queries & Use Cases

### Basic Spend Analysis
```sql
-- Top 10 suppliers by spend
SELECT 
    v.vendor_name,
    SUM(f.spend_amount) as total_spend,
    COUNT(*) as transaction_count
FROM fact_spend_analytics f
JOIN dim_vendors v ON f.vendor_key = v.vendor_key
WHERE v.is_current_record = TRUE
GROUP BY v.vendor_name
ORDER BY total_spend DESC
LIMIT 10;
```

### Performance Analysis
```sql
-- Supplier performance trends
SELECT 
    v.vendor_name,
    t.year,
    AVG(f.delivery_performance_score) as avg_delivery,
    AVG(f.quality_score) as avg_quality,
    AVG(f.compliance_score) as avg_compliance
FROM fact_spend_analytics f
JOIN dim_vendors v ON f.vendor_key = v.vendor_key
JOIN dim_time t ON f.time_key = t.time_key
WHERE v.is_current_record = TRUE
GROUP BY v.vendor_name, t.year
ORDER BY v.vendor_name, t.year;
```

### ESG Analysis
```sql
-- ESG spend by region
SELECT 
    v.region,
    SUM(f.esg_weighted_spend) as esg_spend,
    SUM(f.spend_amount) as total_spend,
    (SUM(f.esg_weighted_spend) / SUM(f.spend_amount)) * 100 as esg_percentage
FROM fact_spend_analytics f
JOIN dim_vendors v ON f.vendor_key = v.vendor_key
WHERE v.is_current_record = TRUE
GROUP BY v.region
ORDER BY esg_percentage DESC;
```

---

## 🎯 Current Schema Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| **Data Completeness** | 🟢 95% | Strong historical coverage |
| **Performance Metrics** | 🟢 90% | Good supplier performance tracking |
| **ESG Capabilities** | 🟢 85% | Solid ESG and diversity foundation |
| **Risk Management** | 🟡 80% | Basic risk classification needs enhancement |
| **Contract Management** | 🔴 30% | Major gap - contract lifecycle missing |
| **Real-time Capabilities** | 🔴 20% | Historical only - no live updates |
| **External Integration** | 🔴 15% | Limited integration points |

**Overall Assessment**: Strong foundation for historical analysis and basic reporting, requiring targeted enhancements for comprehensive C-Suite reporting capabilities.