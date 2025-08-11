# Assignment 1: EPA Fuel Management System

## 📋 WHAT YOU WILL GET

I will provide you with this **master fuel table** with complete data:

### Master Fuel Table
```sql
-- I WILL PROVIDE THIS TABLE TO YOU
CREATE TABLE master_fuels (
    id SERIAL PRIMARY KEY,
    fuel_type VARCHAR(50) NOT NULL,        -- Gaseous fuels, Liquid fuels, Solid fuels
    fuel_name VARCHAR(255) NOT NULL,       -- LPG, Natural Gas, Gasoline, etc.
    unit VARCHAR(100) NOT NULL,            -- Litres (L), Cubic metres (m3), tonne (t)
    epa_factor DECIMAL(10,6) NOT NULL,     -- EPA emission factor (YOUR FOCUS)
    ipcc_factor DECIMAL(10,6) NOT NULL,    -- IPCC emission factor  
    defra_factor DECIMAL(10,6) NOT NULL,   -- DEFRA emission factor
    ghg_factor DECIMAL(10,6) NOT NULL      -- GHG Protocol emission factor
);

-- SAMPLE DATA I WILL INSERT FOR YOU
INSERT INTO master_fuels (fuel_type, fuel_name, unit, epa_factor, ipcc_factor, defra_factor, ghg_factor) VALUES
('Gaseous fuels', 'LPG', 'Litres (L)', 1.5549, 1.5549, 1.5549, 1.5549),
('Gaseous fuels', 'Natural Gas', 'Cubic metres (m3)', 2.0344, 2.0344, 2.0344, 2.0344),
('Liquid fuels', 'Gasoline', 'Litres (L)', 2.3920, 2.3920, 2.3920, 2.3920),
('Liquid fuels', 'Diesel', 'Litres (L)', 2.6260, 2.6260, 2.6260, 2.6260),
('Liquid fuels', 'Jet Kerosene', 'Litres (L)', 2.5197, 2.5197, 2.5197, 2.5197),
('Solid fuels', 'Anthracite', 'tonne (metric ton) (t)', 2360.4928, 2360.4928, 2360.4928, 2360.4928),
('Solid fuels', 'Bituminous Coal', 'tonne (metric ton) (t)', 2109.2028, 2109.2028, 2109.2028, 2109.2028),
('Solid fuels', 'Sub-bituminous Coal', 'tonne (metric ton) (t)', 1800.0000, 1800.0000, 1800.0000, 1800.0000),
('Solid fuels', 'Lignite', 'tonne (metric ton) (t)', 1200.0000, 1200.0000, 1200.0000, 1200.0000),
('Solid fuels', 'Peat', 'tonne (metric ton) (t)', 1000.0000, 1000.0000, 1000.0000, 1000.0000);
```

---

## 🎯 YOUR ASSIGNMENT: EPA STANDARD

### What You Need to Do:
You will build **5 simple APIs** for EPA fuel management using **ONLY** the `epa_factor` column from the master table.

### Your Focus Fuels:
All 10 fuels in the table, but you calculate emissions using **EPA factors only**.

---

## 🗃️ YOUR DATABASE TABLES

### Table 1: Fuel Inventory (Store fuel quantities)
```sql
CREATE TABLE epa_inventory (
    id SERIAL PRIMARY KEY,
    facility_name VARCHAR(255) NOT NULL,
    fuel_name VARCHAR(255) NOT NULL,         -- Must match master_fuels.fuel_name
    quantity DECIMAL(12,2) NOT NULL,
    unit VARCHAR(100) NOT NULL,
    purchase_date DATE DEFAULT CURRENT_DATE,
    cost_per_unit DECIMAL(8,2),              -- Cost in USD
    supplier VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Table 2: Fuel Usage (Track when fuel is used)
```sql
CREATE TABLE epa_usage (
    id SERIAL PRIMARY KEY,
    inventory_id INTEGER REFERENCES epa_inventory(id),
    usage_date DATE NOT NULL,
    quantity_used DECIMAL(12,2) NOT NULL,
    purpose VARCHAR(255),                    -- 'Heating', 'Power', 'Transport'
    operator_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Table 3: EPA Emissions (Store calculated emissions)
```sql
CREATE TABLE epa_emissions (
    id SERIAL PRIMARY KEY,
    usage_id INTEGER REFERENCES epa_usage(id),
    fuel_name VARCHAR(255) NOT NULL,
    quantity_used DECIMAL(12,2) NOT NULL,
    epa_factor DECIMAL(10,6) NOT NULL,
    co2_emissions_kg DECIMAL(15,3) NOT NULL,  -- Calculated result
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🔨 YOUR 5 REQUIRED APIs

### API 1: Get Available EPA Fuels
```
GET /api/epa/fuels
```

**What it does:** Shows all fuels with their EPA emission factors

**SQL Query Example:**
```sql
SELECT fuel_type, fuel_name, unit, epa_factor 
FROM master_fuels 
ORDER BY fuel_type, fuel_name;
```

**Response Example:**
```json
{
    "success": true,
    "data": [
        {
            "fuel_type": "Gaseous fuels",
            "fuel_name": "LPG",
            "unit": "Litres (L)",
            "epa_factor": 1.5549
        },
        {
            "fuel_type": "Liquid fuels", 
            "fuel_name": "Gasoline",
            "unit": "Litres (L)",
            "epa_factor": 2.3920
        }
    ]
}
```

### API 2: Add Fuel to Inventory
```
POST /api/epa/inventory
```

**What it does:** Add new fuel to facility inventory

**Request Body:**
```json
{
    "facility_name": "Houston Plant",
    "fuel_name": "Diesel",
    "quantity": 1000.0,
    "unit": "Litres (L)",
    "cost_per_unit": 1.25,
    "supplier": "Shell Energy"
}
```

**Validation Rules:**
1. `fuel_name` must exist in `master_fuels` table
2. `quantity` must be positive number
3. `unit` must match the unit in `master_fuels` for that fuel

### API 3: Record Fuel Usage
```
POST /api/epa/usage
```

**What it does:** Record when fuel is consumed

**Request Body:**
```json
{
    "inventory_id": 1,
    "usage_date": "2024-08-10",
    "quantity_used": 50.0,
    "purpose": "Heating",
    "operator_name": "John Smith"
}
```

**Validation Rules:**
1. `inventory_id` must exist in `epa_inventory` table
2. `quantity_used` cannot be more than available quantity in inventory
3. `usage_date` cannot be in the future

### API 4: Calculate EPA Emissions
```
POST /api/epa/calculate
```

**What it does:** Calculate CO2 emissions for fuel usage using EPA factors

**Request Body:**
```json
{
    "usage_id": 1
}
```

**Calculation Logic:**
```
1. Get usage details from epa_usage table
2. Get fuel_name from related inventory record  
3. Get EPA factor from master_fuels table
4. Calculate: CO2 Emissions = quantity_used × epa_factor
5. Save result in epa_emissions table
```

**Response Example:**
```json
{
    "success": true,
    "data": {
        "usage_id": 1,
        "fuel_name": "Diesel",
        "quantity_used": 50.0,
        "epa_factor": 2.6260,
        "co2_emissions_kg": 131.30,
        "unit": "kg CO2"
    }
}
```

### API 5: Get Emissions Report
```
GET /api/epa/report?facility=:name&start_date=:date&end_date=:date
```

**What it does:** Generate emissions report for a facility and time period

**Response Example:**
```json
{
    "success": true,
    "data": {
        "facility_name": "Houston Plant",
        "period": "2024-08-01 to 2024-08-31",
        "total_emissions_kg": 1250.75,
        "fuel_breakdown": [
            {
                "fuel_name": "Diesel",
                "quantity_used": 500.0,
                "unit": "Litres (L)",
                "emissions_kg": 1313.0
            },
            {
                "fuel_name": "Natural Gas", 
                "quantity_used": 200.0,
                "unit": "Cubic metres (m3)",
                "emissions_kg": 406.88
            }
        ]
    }
}
```

---

## 💾 SAMPLE DATA TO INSERT

### Sample Inventory Data
```sql
INSERT INTO epa_inventory (facility_name, fuel_name, quantity, unit, cost_per_unit, supplier) VALUES
('Houston Plant', 'Diesel', 5000.0, 'Litres (L)', 1.25, 'Shell Energy'),
('Houston Plant', 'Natural Gas', 10000.0, 'Cubic metres (m3)', 0.45, 'Texas Gas'),
('Dallas Office', 'Gasoline', 800.0, 'Litres (L)', 1.35, 'Exxon'),
('Chicago Plant', 'LPG', 1200.0, 'Litres (L)', 0.85, 'AmeriGas'),
('Detroit Plant', 'Bituminous Coal', 50.0, 'tonne (metric ton) (t)', 120.00, 'Coal Supply Co');
```

### Sample Usage Data
```sql
INSERT INTO epa_usage (inventory_id, usage_date, quantity_used, purpose, operator_name) VALUES
(1, '2024-08-10', 150.0, 'Generator Power', 'John Smith'),
(2, '2024-08-10', 300.0, 'Heating', 'Maria Garcia'),
(3, '2024-08-11', 45.0, 'Vehicle Fleet', 'Mike Johnson'),
(4, '2024-08-11', 75.0, 'Forklift Operations', 'Sarah Wilson'),
(5, '2024-08-12', 2.0, 'Boiler Fuel', 'David Brown');
```

---

## ✅ VALIDATION REQUIREMENTS

### Input Validation:
1. **Fuel Name Check:** Must exist in master_fuels table
2. **Quantity Check:** Must be positive numbers
3. **Date Check:** Usage dates cannot be future dates
4. **Inventory Check:** Cannot use more fuel than available
5. **Unit Matching:** Unit must match master_fuels unit for that fuel

### Calculation Validation:
1. **EPA Factor Lookup:** Always get current EPA factor from master_fuels
2. **Math Check:** Emissions = Quantity × EPA Factor (exactly)
3. **Decimal Precision:** Store emissions with 3 decimal places
4. **No Negative Results:** Emissions cannot be negative

---

## 🎯 SUCCESS CRITERIA

You succeed when:
- ✅ All 5 APIs work correctly
- ✅ Emissions calculations are mathematically correct
- ✅ Data validation prevents bad data entry
- ✅ Reports show accurate totals
- ✅ System handles 100+ records without issues
- ✅ Code is organized and readable

---

## 📝 STEP-BY-STEP GUIDE

### Week 1: Setup
1. **Create database** and connect to PostgreSQL
2. **Insert master_fuels table** with the data I provide
3. **Create your 3 tables** (inventory, usage, emissions)
4. **Test basic database connections**

### Week 2: Basic APIs
1. **Build API 1:** Get available fuels (simplest)
2. **Build API 2:** Add inventory (with validation)
3. **Test with sample data**

### Week 3: Advanced APIs  
1. **Build API 3:** Record usage (with inventory checks)
2. **Build API 4:** Calculate emissions (core logic)
3. **Test emission calculations manually**

### Week 4: Reporting & Testing
1. **Build API 5:** Generate reports
2. **Add comprehensive testing**
3. **Write documentation**
4. **Prepare demo**

---

## 🔍 TESTING CHECKLIST

Test these scenarios:
- ✅ Add inventory for all 10 fuel types
- ✅ Record usage that reduces inventory correctly
- ✅ Calculate emissions for each fuel type
- ✅ Generate report with multiple fuels
- ✅ Handle error: fuel name doesn't exist
- ✅ Handle error: not enough inventory
- ✅ Handle error: negative quantities
- ✅ Handle error: future usage dates

---

## 💡 HELPFUL TIPS

### Calculation Examples:
```
Diesel: 100 Litres × 2.6260 = 262.60 kg CO2
Natural Gas: 50 m³ × 2.0344 = 101.72 kg CO2  
LPG: 25 Litres × 1.5549 = 38.87 kg CO2
```

### Common Mistakes to Avoid:
1. **Wrong factor:** Using IPCC factor instead of EPA factor
2. **Unit mismatch:** Diesel in m³ instead of Litres
3. **Missing validation:** Allowing negative quantities
4. **Calculation errors:** Not storing enough decimal places

### Database Best Practices:
1. **Use transactions** for related operations
2. **Add indexes** on frequently queried columns
3. **Use foreign keys** to maintain data integrity
4. **Handle database errors** gracefully

---

## 📚 RESOURCES

- **PostgreSQL Docs:** https://postgresql.org/docs/
- **REST API Guidelines:** Use consistent JSON responses
- **Calculator:** Use for manual verification of calculations
- **Postman:** For testing your APIs

## 🆘 WHEN TO ASK FOR HELP

Ask for help if:
- Stuck on same problem for 2+ hours
- Calculations don't match manual math
- Database won't connect or queries fail
- APIs return unexpected results

**Remember:** You're building a real system for EPA compliance. Focus on accuracy and reliability! 🚀
