# Contract Expiry & Renewal Report Specification

## 📊 Report Overview
**Purpose**: Proactive management of supplier agreements to prevent service disruptions and create strategic leverage during renewals  
**Audience**: C-Suite executives, Procurement leadership, Legal teams  
**Frequency**: Weekly monitoring with monthly strategic reviews  
**Database Coverage**: 65% → 95% (Major Enhancement Required)

---

## 🔍 Current State Analysis

### Major Data Gaps (35% missing)
The Contract Expiry & Renewal Report represents the **largest coverage gap** in our current database, requiring significant schema enhancements and external system integration.

| Gap Category | Impact | Current Coverage | Enhanced Coverage |
|-------------|--------|------------------|-------------------|
| **Contract Master Data** | Critical | 20% | 95% |
| **Renewal Tracking** | High | 15% | 90% |
| **Legal Review Status** | Medium | 10% | 85% |
| **Auto-Renewal Alerts** | High | 5% | 95% |

---

## 📋 Report Sections & Data Requirements

### 1. Executive Summary
**Purpose**: Snapshot of total expiring contracts and strategic risks

| Data Element | Current Source | Coverage | Enhanced Source | Enhanced Coverage |
|-------------|---------------|----------|----------------|-------------------|
| # of contracts expiring | Manual tracking | ❌ 10% | `dim_contracts` | ✅ 95% |
| % of spend affected | Estimated calculation | 🟡 60% | Calculated from contract values | ✅ 95% |
| Critical contract alerts | Ad-hoc identification | ❌ 20% | Automated risk scoring | ✅ 90% |
| Strategic risks | Manual assessment | 🟡 50% | Risk-weighted analysis | ✅ 85% |

**Enhanced SQL Implementation (Phase 1)**:
```sql
-- Executive contract summary
SELECT 
    COUNT(*) as total_expiring_contracts,
    SUM(c.contract_value) as total_value_at_risk,
    COUNT(CASE WHEN c.end_date <= DATE('now', '+90 days') THEN 1 END) as urgent_renewals,
    COUNT(CASE WHEN c.auto_renewal_flag = 1 AND c.renewal_notice_days > 0 THEN 1 END) as auto_renewal_risk,
    SUM(CASE WHEN v.vendor_tier = 'Strategic' THEN c.contract_value ELSE 0 END) as strategic_value_at_risk
FROM dim_contracts c
JOIN dim_vendors v ON c.vendor_key = v.vendor_key
WHERE c.contract_status = 'Active'
  AND c.end_date <= DATE('now', '+365 days');  -- Next 12 months
```

---

### 2. Expiring Contracts List
**Purpose**: Detailed table of contracts expiring within defined timeframes

| Data Element | Current Source | Coverage | Enhanced Source | Enhanced Coverage |
|-------------|---------------|----------|----------------|-------------------|
| Supplier name | `dim_vendors.vendor_name` | ✅ 100% | Same | ✅ 100% |
| Contract name/ID | External spreadsheets | ❌ 25% | `dim_contracts.contract_id` | ✅ 95% |
| Contract value | External estimates | 🟡 40% | `dim_contracts.contract_value` | ✅ 95% |
| Start/end dates | Partial data | 🟡 35% | `dim_contracts.start_date/end_date` | ✅ 95% |
| Auto-renewal status | Unknown for most | ❌ 15% | `dim_contracts.auto_renewal_flag` | ✅ 90% |

**Enhanced SQL Implementation**:
```sql
-- Comprehensive expiring contracts analysis
SELECT 
    c.contract_id,
    c.contract_name,
    v.vendor_name,
    c.contract_value,
    c.start_date,
    c.end_date,
    DATEDIFF(c.end_date, DATE('now')) as days_until_expiry,
    c.auto_renewal_flag,
    c.renewal_notice_days,
    v.vendor_tier,
    v.risk_rating,
    CASE 
        WHEN DATEDIFF(c.end_date, DATE('now')) <= 30 THEN 'Critical'
        WHEN DATEDIFF(c.end_date, DATE('now')) <= 90 THEN 'Urgent'
        WHEN DATEDIFF(c.end_date, DATE('now')) <= 180 THEN 'Plan'
        ELSE 'Monitor'
    END as renewal_priority,
    -- Calculate annual spend under this contract
    COALESCE(spend_summary.annual_spend, 0) as annual_spend_impact
FROM dim_contracts c
JOIN dim_vendors v ON c.vendor_key = v.vendor_key
LEFT JOIN (
    SELECT 
        f.vendor_key,
        f.contract_key,
        SUM(f.spend_amount) as annual_spend
    FROM fact_spend_analytics f
    JOIN dim_time t ON f.time_key = t.time_key
    WHERE t.year = 2024  -- Current year
    GROUP BY f.vendor_key, f.contract_key
) spend_summary ON c.contract_key = spend_summary.contract_key
WHERE c.contract_status = 'Active'
  AND c.end_date <= DATE('now', '+365 days')
ORDER BY 
    CASE renewal_priority
        WHEN 'Critical' THEN 1
        WHEN 'Urgent' THEN 2
        WHEN 'Plan' THEN 3
        ELSE 4
    END,
    c.contract_value DESC;
```

---

### 3. Auto-Renewal Risk Section
**Purpose**: Identify contracts that auto-renew without action

| Data Element | Current Source | Coverage | Enhanced Source | Enhanced Coverage |
|-------------|---------------|----------|----------------|-------------------|
| Notice period requirements | Unknown | ❌ 5% | `dim_contracts.renewal_notice_days` | ✅ 90% |
| Renewal clause summary | Manual research | ❌ 10% | `dim_contracts` + external CLM | 🟡 70% |
| Renewal deadline date | Calculated when known | 🟡 20% | Automated calculation | ✅ 95% |
| Risk of unintended renewal | Not tracked | ❌ 0% | Automated risk scoring | ✅ 85% |

**Enhanced SQL Implementation**:
```sql
-- Auto-renewal risk analysis
SELECT 
    c.contract_id,
    c.contract_name,
    v.vendor_name,
    c.end_date,
    c.auto_renewal_flag,
    c.renewal_notice_days,
    DATE(c.end_date, '-' || c.renewal_notice_days || ' days') as notice_deadline,
    DATEDIFF(DATE(c.end_date, '-' || c.renewal_notice_days || ' days'), DATE('now')) as days_to_notice_deadline,
    c.contract_value,
    CASE 
        WHEN c.auto_renewal_flag = 1 
         AND DATEDIFF(DATE(c.end_date, '-' || c.renewal_notice_days || ' days'), DATE('now')) <= 30 
        THEN 'HIGH RISK - Notice deadline within 30 days'
        WHEN c.auto_renewal_flag = 1 
         AND DATEDIFF(DATE(c.end_date, '-' || c.renewal_notice_days || ' days'), DATE('now')) <= 60 
        THEN 'MEDIUM RISK - Notice deadline within 60 days'
        WHEN c.auto_renewal_flag = 1 
        THEN 'LOW RISK - Notice deadline > 60 days away'
        ELSE 'NO AUTO-RENEWAL'
    END as auto_renewal_risk_level
FROM dim_contracts c
JOIN dim_vendors v ON c.vendor_key = v.vendor_key
WHERE c.contract_status = 'Active'
  AND c.end_date <= DATE('now', '+365 days')
  AND c.auto_renewal_flag = 1
ORDER BY days_to_notice_deadline ASC;
```

---

### 4. Business Impact Rating
**Purpose**: Risk assessment based on contract criticality and spend volume

| Data Element | Current Source | Coverage | Enhanced Source | Enhanced Coverage |
|-------------|---------------|----------|----------------|-------------------|
| Criticality rating | Manual assessment | 🟡 40% | Enhanced vendor tier + spend analysis | ✅ 90% |
| % of category spend | Spend analysis | ✅ 85% | Improved with contract attribution | ✅ 95% |
| Impact on operations | Subjective assessment | 🟡 30% | Data-driven impact scoring | ✅ 80% |
| Service continuity risk | Not assessed | ❌ 0% | Automated risk calculation | ✅ 75% |

**Enhanced SQL Implementation**:
```sql
-- Business impact assessment
WITH contract_impact AS (
    SELECT 
        c.contract_key,
        c.contract_id,
        c.contract_name,
        v.vendor_name,
        v.vendor_tier,
        c.contract_value,
        -- Calculate category concentration
        cat_spend.category_total_spend,
        contract_spend.contract_annual_spend,
        ROUND((contract_spend.contract_annual_spend / cat_spend.category_total_spend) * 100, 1) as category_concentration_pct,
        -- Calculate business unit impact
        bu_impact.affected_business_units,
        bu_impact.primary_business_unit
    FROM dim_contracts c
    JOIN dim_vendors v ON c.vendor_key = v.vendor_key
    -- Annual spend under this contract
    LEFT JOIN (
        SELECT 
            f.contract_key,
            SUM(f.spend_amount) as contract_annual_spend
        FROM fact_spend_analytics f
        JOIN dim_time t ON f.time_key = t.time_key
        WHERE t.year = 2024
        GROUP BY f.contract_key
    ) contract_spend ON c.contract_key = contract_spend.contract_key
    -- Total category spend for concentration analysis
    LEFT JOIN (
        SELECT 
            co.parent_category,
            SUM(f.spend_amount) as category_total_spend
        FROM fact_spend_analytics f
        JOIN dim_commodities co ON f.commodity_key = co.commodity_key
        JOIN dim_time t ON f.time_key = t.time_key
        WHERE t.year = 2024
        GROUP BY co.parent_category
    ) cat_spend ON TRUE  -- Will need proper category join in Phase 1
    -- Business unit impact analysis (Phase 1 enhancement)
    LEFT JOIN (
        SELECT 
            f.contract_key,
            COUNT(DISTINCT f.business_unit_key) as affected_business_units,
            -- Primary BU = highest spend
            (SELECT bu.business_unit_name 
             FROM fact_spend_analytics f2 
             JOIN dim_business_units bu ON f2.business_unit_key = bu.business_unit_key
             WHERE f2.contract_key = f.contract_key 
             GROUP BY bu.business_unit_name 
             ORDER BY SUM(f2.spend_amount) DESC 
             LIMIT 1) as primary_business_unit
        FROM fact_spend_analytics f
        WHERE f.contract_key IS NOT NULL
        GROUP BY f.contract_key
    ) bu_impact ON c.contract_key = bu_impact.contract_key
    WHERE c.contract_status = 'Active'
      AND c.end_date <= DATE('now', '+365 days')
)
SELECT 
    contract_id,
    contract_name,
    vendor_name,
    vendor_tier,
    contract_annual_spend,
    category_concentration_pct,
    affected_business_units,
    primary_business_unit,
    CASE 
        WHEN vendor_tier = 'Strategic' AND category_concentration_pct > 50 THEN 'CRITICAL'
        WHEN vendor_tier = 'Strategic' OR category_concentration_pct > 25 THEN 'HIGH'
        WHEN vendor_tier = 'Preferred' OR category_concentration_pct > 10 THEN 'MEDIUM'
        ELSE 'LOW'
    END as business_impact_level,
    CASE 
        WHEN vendor_tier = 'Strategic' AND category_concentration_pct > 50 
        THEN 'Major operational risk - single source for critical category'
        WHEN vendor_tier = 'Strategic' 
        THEN 'Strategic supplier relationship at risk'
        WHEN category_concentration_pct > 25 
        THEN 'Significant category spend concentration'
        ELSE 'Standard renewal process recommended'
    END as impact_assessment
FROM contract_impact
ORDER BY 
    CASE vendor_tier
        WHEN 'Strategic' THEN 1
        WHEN 'Preferred' THEN 2
        ELSE 3
    END,
    category_concentration_pct DESC;
```

---

### 5. Renewal Status Tracker
**Purpose**: Track progress of renewal planning for each expiring contract

| Data Element | Current Source | Coverage | Enhanced Source | Enhanced Coverage |
|-------------|---------------|----------|----------------|-------------------|
| Renewal owner | Manual tracking | 🟡 30% | `dim_contracts.contract_owner` | ✅ 85% |
| Renewal stage | Ad-hoc updates | ❌ 15% | `fact_contract_events` | ✅ 90% |
| Issues raised | Email/meetings | ❌ 10% | Contract event tracking | ✅ 80% |
| Approval status | Manual tracking | ❌ 20% | Workflow integration | 🟡 70% |

**Enhanced with Phase 1 Implementation**:
```sql
-- Renewal status tracking
SELECT 
    c.contract_id,
    c.contract_name,
    v.vendor_name,
    c.contract_owner,
    c.end_date,
    -- Latest renewal status from contract events
    latest_event.event_type as latest_action,
    latest_event.event_date as last_update_date,
    latest_event.responsible_party,
    latest_event.approval_status,
    CASE 
        WHEN latest_event.event_type = 'Renewal Initiated' THEN 'Planning'
        WHEN latest_event.event_type = 'Negotiation Started' THEN 'Negotiating'
        WHEN latest_event.event_type = 'Legal Review' THEN 'Legal Review'
        WHEN latest_event.event_type = 'Approved' THEN 'Approved'
        WHEN latest_event.event_type = 'Executed' THEN 'Completed'
        ELSE 'Not Started'
    END as renewal_stage,
    DATEDIFF(c.end_date, DATE('now')) as days_remaining
FROM dim_contracts c
JOIN dim_vendors v ON c.vendor_key = v.vendor_key
LEFT JOIN (
    -- Get latest contract event for each contract
    SELECT 
        ce.contract_key,
        ce.event_type,
        ce.event_date,
        ce.responsible_party,
        ce.approval_status,
        ROW_NUMBER() OVER (PARTITION BY ce.contract_key ORDER BY ce.event_date DESC) as rn
    FROM fact_contract_events ce
    WHERE ce.event_type IN ('Renewal Initiated', 'Negotiation Started', 'Legal Review', 'Approved', 'Executed')
) latest_event ON c.contract_key = latest_event.contract_key AND latest_event.rn = 1
WHERE c.contract_status = 'Active'
  AND c.end_date <= DATE('now', '+365 days')
ORDER BY c.end_date ASC;
```

---

## 🎯 Implementation Strategy

### Phase 1: Contract Foundation (Months 1-2)
**Priority**: Critical
**Investment**: $75K

#### 1.1 Contract Master Data Creation
```sql
-- Create contracts dimension table
CREATE TABLE dim_contracts (
    contract_key INTEGER PRIMARY KEY,
    contract_id TEXT UNIQUE NOT NULL,
    contract_name TEXT,
    vendor_key INTEGER REFERENCES dim_vendors(vendor_key),
    contract_type TEXT, -- 'Master', 'SOW', 'Purchase Agreement'
    contract_status TEXT, -- 'Active', 'Expired', 'Under Renewal', 'Terminated'
    start_date DATE,
    end_date DATE,
    auto_renewal_flag BOOLEAN DEFAULT FALSE,
    renewal_notice_days INTEGER,
    contract_value DECIMAL(15,2),
    currency_code TEXT DEFAULT 'USD',
    payment_terms_days INTEGER,
    contract_owner TEXT,
    legal_review_date DATE,
    next_review_date DATE,
    created_date DATE DEFAULT CURRENT_DATE,
    is_current_record BOOLEAN DEFAULT TRUE
);
```

#### 1.2 Contract Events Tracking
```sql
-- Create contract events fact table
CREATE TABLE fact_contract_events (
    event_key INTEGER PRIMARY KEY,
    contract_key INTEGER REFERENCES dim_contracts(contract_key),
    time_key INTEGER REFERENCES dim_time(time_key),
    event_type TEXT, -- 'Created', 'Renewed', 'Amended', 'Terminated', 'Renewal Initiated'
    event_description TEXT,
    responsible_party TEXT,
    approval_required BOOLEAN,
    approval_status TEXT,
    approval_date DATE,
    financial_impact DECIMAL(15,2),
    risk_level TEXT,
    created_date DATE DEFAULT CURRENT_DATE
);
```

#### 1.3 Data Migration Plan
1. **Contract Discovery**: Audit existing contract sources
2. **Data Extraction**: Pull from legal systems, ERP, spreadsheets
3. **Data Cleansing**: Standardize formats and classifications
4. **Contract Loading**: Populate dim_contracts table
5. **Relationship Mapping**: Link contracts to spend data
6. **Validation**: Ensure data quality and completeness

### Phase 2: External Integration (Months 3-4)
**Priority**: High
**Investment**: $50K

#### 2.1 Contract Management System Integration
- Connect to legal/CLM systems for real-time updates
- Establish API connections for contract status
- Implement webhook notifications for contract changes

#### 2.2 Workflow Integration
- Connect to approval workflow systems
- Track renewal progress through stages
- Enable automated notifications and alerts

---

## 📊 Expected Outcomes

### Coverage Improvement
| Report Section | Current | Phase 1 | Phase 2 | Target |
|---------------|---------|---------|---------|--------|
| Contract Master Data | 20% | 90% | 95% | 95% |
| Renewal Tracking | 15% | 85% | 90% | 90% |
| Auto-Renewal Alerts | 5% | 90% | 95% | 95% |
| Business Impact | 40% | 80% | 85% | 85% |
| **Overall Report** | **65%** | **88%** | **92%** | **95%** |

### Business Value
- **Prevented Auto-Renewals**: 15-20 contracts annually ($2-5M value)
- **Improved Negotiation Position**: 30-day average advance notice
- **Risk Mitigation**: 95% reduction in contract expiration surprises
- **Cost Avoidance**: $500K-2M annually in unfavorable renewals

### Operational Benefits
- **Automated Monitoring**: Replace manual contract tracking
- **Proactive Management**: 90-day advance warning system
- **Strategic Planning**: Contract renewal pipeline visibility
- **Risk Reduction**: Eliminate service interruption risks

---

This report transformation from 65% to 95% coverage represents the **highest-impact enhancement** in the entire database improvement project, directly addressing critical business continuity and strategic leverage opportunities.