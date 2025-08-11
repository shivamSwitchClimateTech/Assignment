# Assignment 1: EPA Fuel Management System 🇺🇸

**Assigned to:** Devanshi Nikam  
**Standard Focus:** EPA (Environmental Protection Agency - USA)  
**Duration:** 4 weeks  

---

## 📋 WHAT YOU WILL GET

### Master Fuel Table
```sql
-- I WILL PROVIDE THIS TABLE TO YOU
CREATE TABLE master_fuels (
    id SERIAL PRIMARY KEY,
    fuel_type VARCHAR(50) NOT NULL,        -- Gaseous fuels, Liquid fuels, Solid fuels
    fuel_name VARCHAR(255) NOT NULL UNIQUE, -- LPG, Natural Gas, Gasoline, etc.
    unit VARCHAR(100) NOT NULL,            -- Litres (L), Cubic metres (m3), tonne (t)
    epa_factor DECIMAL(10,6),              -- EPA emission factor (YOUR FOCUS)
    ipcc_factor DECIMAL(10,6),             -- IPCC emission factor  
    defra_factor DECIMAL(10,6),            -- DEFRA emission factor
    ghg_factor DECIMAL(10,6),              -- GHG Protocol emission factor
    is_default BOOLEAN DEFAULT FALSE,      -- TRUE if factor was set to 1 (default)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- SAMPLE DATA I WILL INSERT FOR YOU (with NULL factors initially)
INSERT INTO master_fuels (fuel_type, fuel_name, unit, epa_factor, ipcc_factor, defra_factor, ghg_factor) VALUES
('Gaseous fuels', 'LPG', 'Litres (L)', NULL, NULL, NULL, NULL),
('Gaseous fuels', 'Natural Gas', 'Cubic metres (m3)', NULL, NULL, NULL, NULL),
('Liquid fuels', 'Gasoline', 'Litres (L)', NULL, NULL, NULL, NULL),
('Liquid fuels', 'Diesel', 'Litres (L)', NULL, NULL, NULL, NULL),
('Liquid fuels', 'Jet Kerosene', 'Litres (L)', NULL, NULL, NULL, NULL),
('Solid fuels', 'Anthracite', 'tonne (metric ton) (t)', NULL, NULL, NULL, NULL),
('Solid fuels', 'Bituminous Coal', 'tonne (metric ton) (t)', NULL, NULL, NULL, NULL),
('Solid fuels', 'Sub-bituminous Coal', 'tonne (metric ton) (t)', NULL, NULL, NULL, NULL),
('Solid fuels', 'Lignite', 'tonne (metric ton) (t)', NULL, NULL, NULL, NULL),
('Solid fuels', 'Peat', 'tonne (metric ton) (t)', NULL, NULL, NULL, NULL);
```

### 🔍 Your First Task: Research EPA Factors

**IMPORTANT:** You must research and find the EPA emission factor for each fuel:
1. Research EPA emission factors for each of the 10 fuels
2. Update the `epa_factor` column with the correct values
3. If no EPA factor is found, set it to `1.000000` and mark `is_default = TRUE`

**Example Update Query:**
```sql
-- After research, update with real EPA factors
UPDATE master_fuels SET epa_factor = 2.6260, is_default = FALSE WHERE fuel_name = 'Diesel';
UPDATE master_fuels SET epa_factor = 1.0000, is_default = TRUE WHERE fuel_name = 'Some Fuel';
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
    quantity DECIMAL(12,2) NOT NULL CHECK (quantity >= 0),
    unit VARCHAR(100) NOT NULL,
    purchase_date DATE DEFAULT CURRENT_DATE,
    cost_per_unit DECIMAL(8,2) CHECK (cost_per_unit >= 0), -- Cost in USD
    supplier VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign key constraint
    CONSTRAINT fk_fuel_name FOREIGN KEY (fuel_name) REFERENCES master_fuels(fuel_name) ON UPDATE CASCADE,
    
    -- Check constraint for valid unit matching
    CONSTRAINT valid_unit CHECK (
        (fuel_name, unit) IN (
            SELECT fuel_name, unit FROM master_fuels
        )
    )
);

-- Indexes for performance
CREATE INDEX idx_epa_inventory_facility ON epa_inventory(facility_name);
CREATE INDEX idx_epa_inventory_fuel ON epa_inventory(fuel_name);
CREATE INDEX idx_epa_inventory_date ON epa_inventory(purchase_date);
```

### Table 2: Fuel Usage (Track when fuel is used)
```sql
CREATE TABLE epa_usage (
    id SERIAL PRIMARY KEY,
    inventory_id INTEGER NOT NULL,
    usage_date DATE NOT NULL CHECK (usage_date <= CURRENT_DATE),
    quantity_used DECIMAL(12,2) NOT NULL CHECK (quantity_used > 0),
    purpose VARCHAR(255),                    -- 'Heating', 'Power', 'Transport'
    operator_name VARCHAR(100),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign key constraint
    CONSTRAINT fk_inventory_id FOREIGN KEY (inventory_id) REFERENCES epa_inventory(id) ON DELETE CASCADE,
    
    -- Check constraint: cannot use more than available
    CONSTRAINT check_quantity_available CHECK (
        quantity_used <= (
            SELECT quantity FROM epa_inventory WHERE id = inventory_id
        )
    )
);

-- Indexes for performance
CREATE INDEX idx_epa_usage_inventory ON epa_usage(inventory_id);
CREATE INDEX idx_epa_usage_date ON epa_usage(usage_date);
CREATE INDEX idx_epa_usage_purpose ON epa_usage(purpose);
```

### Table 3: EPA Emissions (Store calculated emissions)
```sql
CREATE TABLE epa_emissions (
    id SERIAL PRIMARY KEY,
    usage_id INTEGER NOT NULL UNIQUE,       -- One calculation per usage
    fuel_name VARCHAR(255) NOT NULL,
    quantity_used DECIMAL(12,2) NOT NULL CHECK (quantity_used > 0),
    epa_factor DECIMAL(10,6) NOT NULL CHECK (epa_factor > 0),
    co2_emissions_kg DECIMAL(15,3) NOT NULL CHECK (co2_emissions_kg >= 0), -- Calculated result
    calculation_method VARCHAR(100) DEFAULT 'EPA Standard',
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign key constraint
    CONSTRAINT fk_usage_id FOREIGN KEY (usage_id) REFERENCES epa_usage(id) ON DELETE CASCADE,
    
    -- Check constraint: ensure calculation is correct
    CONSTRAINT check_calculation CHECK (
        ABS(co2_emissions_kg - (quantity_used * epa_factor)) < 0.001
    )
);

-- Indexes for performance
CREATE INDEX idx_epa_emissions_usage ON epa_emissions(usage_id);
CREATE INDEX idx_epa_emissions_fuel ON epa_emissions(fuel_name);
CREATE INDEX idx_epa_emissions_date ON epa_emissions(calculated_at);
```

---

## 🔨 YOUR 5 REQUIRED APIs

### API 1: Get Available EPA Fuels
```
GET /api/epa/fuels
```

**What it does:** Shows all fuels with their EPA emission factors

**Request Parameters:** None

**Response Format:**
```json
{
    "success": true,
    "message": "EPA fuels retrieved successfully",
    "data": [
        {
            "id": 1,
            "fuel_type": "Gaseous fuels",
            "fuel_name": "LPG",
            "unit": "Litres (L)",
            "epa_factor": 1.5549,
            "is_default": false
        },
        {
            "id": 2,
            "fuel_type": "Liquid fuels", 
            "fuel_name": "Gasoline",
            "unit": "Litres (L)",
            "epa_factor": 2.3920,
            "is_default": false
        }
    ],
    "total_count": 10
}
```

**Error Response:**
```json
{
    "success": false,
    "message": "Failed to retrieve EPA fuels",
    "error": "Database connection error"
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
1. `facility_name` is required (min 3 characters)
2. `fuel_name` must exist in `master_fuels` table
3. `quantity` must be positive number
4. `unit` must match the unit in `master_fuels` for that fuel
5. `cost_per_unit` must be positive if provided

**Success Response:**
```json
{
    "success": true,
    "message": "Fuel added to inventory successfully",
    "data": {
        "id": 1,
        "facility_name": "Houston Plant",
        "fuel_name": "Diesel",
        "quantity": 1000.0,
        "unit": "Litres (L)",
        "cost_per_unit": 1.25,
        "supplier": "Shell Energy",
        "purchase_date": "2024-08-11",
        "created_at": "2024-08-11T10:30:00Z"
    }
}
```

**Error Response:**
```json
{
    "success": false,
    "message": "Validation failed",
    "errors": [
        {
            "field": "fuel_name",
            "message": "Fuel 'Unknown Fuel' does not exist in master fuels"
        },
        {
            "field": "quantity",
            "message": "Quantity must be a positive number"
        }
    ]
}
```

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
    "operator_name": "John Smith",
    "notes": "Monthly heating for Building A"
}
```

**Validation Rules:**
1. `inventory_id` must exist in `epa_inventory` table
2. `quantity_used` must be positive number
3. `quantity_used` cannot be more than available quantity in inventory
4. `usage_date` cannot be in the future
5. `purpose` is required

**Success Response:**
```json
{
    "success": true,
    "message": "Fuel usage recorded successfully",
    "data": {
        "id": 1,
        "inventory_id": 1,
        "usage_date": "2024-08-10",
        "quantity_used": 50.0,
        "purpose": "Heating",
        "operator_name": "John Smith",
        "notes": "Monthly heating for Building A",
        "created_at": "2024-08-11T10:35:00Z",
        "remaining_inventory": 950.0
    }
}
```

**Error Response:**
```json
{
    "success": false,
    "message": "Insufficient inventory",
    "error": "Requested quantity (50.0) exceeds available inventory (30.0)",
    "available_quantity": 30.0
}
```

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

**Success Response:**
```json
{
    "success": true,
    "message": "EPA emissions calculated successfully",
    "data": {
        "id": 1,
        "usage_id": 1,
        "fuel_name": "Diesel",
        "quantity_used": 50.0,
        "epa_factor": 2.6260,
        "co2_emissions_kg": 131.30,
        "calculation_method": "EPA Standard",
        "calculated_at": "2024-08-11T10:40:00Z",
        "unit": "kg CO2"
    }
}
```

**Error Response:**
```json
{
    "success": false,
    "message": "Calculation failed",
    "error": "Usage ID 1 not found or already calculated"
}
```

### API 5: Get Emissions Report
```
GET /api/epa/report?facility=Houston Plant&start_date=2024-08-01&end_date=2024-08-31
```

**What it does:** Generate emissions report for a facility and time period

**Query Parameters:**
- `facility` (required): Facility name
- `start_date` (required): Start date (YYYY-MM-DD)
- `end_date` (required): End date (YYYY-MM-DD)

**Success Response:**
```json
{
    "success": true,
    "message": "EPA emissions report generated successfully",
    "data": {
        "facility_name": "Houston Plant",
        "period": {
            "start_date": "2024-08-01",
            "end_date": "2024-08-31"
        },
        "summary": {
            "total_emissions_kg": 1250.75,
            "total_fuel_consumed": 725.5,
            "total_activities": 15,
            "average_daily_emissions": 40.35
        },
        "fuel_breakdown": [
            {
                "fuel_name": "Diesel",
                "fuel_type": "Liquid fuels",
                "quantity_used": 500.0,
                "unit": "Litres (L)",
                "epa_factor": 2.6260,
                "emissions_kg": 1313.0,
                "percentage_of_total": 52.1
            },
            {
                "fuel_name": "Natural Gas", 
                "fuel_type": "Gaseous fuels",
                "quantity_used": 200.0,
                "unit": "Cubic metres (m3)",
                "epa_factor": 2.0344,
                "emissions_kg": 406.88,
                "percentage_of_total": 16.1
            }
        ],
        "purpose_breakdown": [
            {
                "purpose": "Heating",
                "emissions_kg": 650.25,
                "percentage_of_total": 52.0
            },
            {
                "purpose": "Power Generation",
                "emissions_kg": 400.50,
                "percentage_of_total": 32.0
            }
        ],
        "monthly_trend": [
            {
                "date": "2024-08-01",
                "daily_emissions_kg": 45.2
            },
            {
                "date": "2024-08-02", 
                "daily_emissions_kg": 38.7
            }
        ]
    }
}
```

**Error Response:**
```json
{
    "success": false,
    "message": "Invalid date range",
    "error": "Start date must be before end date"
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
6. **Required Fields:** All required fields must be provided

### Calculation Validation:
1. **EPA Factor Lookup:** Always get current EPA factor from master_fuels
2. **Math Check:** Emissions = Quantity × EPA Factor (exactly)
3. **Decimal Precision:** Store emissions with 3 decimal places
4. **No Negative Results:** Emissions cannot be negative
5. **Duplicate Prevention:** One emission calculation per usage record

---

## 🎯 SUCCESS CRITERIA

You succeed when:
- ✅ All 5 APIs work correctly with proper error handling
- ✅ EPA factors are researched and populated correctly
- ✅ Emissions calculations are mathematically correct
- ✅ Data validation prevents bad data entry
- ✅ Reports show accurate totals and breakdowns
- ✅ Database constraints prevent data integrity issues
- ✅ System handles 100+ records without performance issues
- ✅ Code is organized and readable with proper comments

---

## 📝 STEP-BY-STEP GUIDE

### Week 1: Setup & Research
1. **Create database** and connect to PostgreSQL
2. **Insert master_fuels table** with the data I provide
3. **Research EPA emission factors** for all 10 fuels
4. **Update master_fuels** with your researched EPA factors
5. **Create your 3 tables** (inventory, usage, emissions)
6. **Test basic database connections**

### Week 2: Basic APIs
1. **Build API 1:** Get available fuels (simplest)
2. **Build API 2:** Add inventory (with validation)
3. **Test with sample data**
4. **Add proper error handling**

### Week 3: Advanced APIs  
1. **Build API 3:** Record usage (with inventory checks)
2. **Build API 4:** Calculate emissions (core logic)
3. **Test emission calculations manually**
4. **Implement all validations**

### Week 4: Reporting & Testing
1. **Build API 5:** Generate comprehensive reports
2. **Add comprehensive testing**
3. **Write documentation**
4. **Prepare demo with real data**

---

## 🔍 TESTING CHECKLIST

Test these scenarios:
- ✅ Add inventory for all 10 fuel types
- ✅ Record usage that reduces inventory correctly
- ✅ Calculate emissions for each fuel type
- ✅ Generate report with multiple fuels and purposes
- ✅ Handle error: fuel name doesn't exist
- ✅ Handle error: not enough inventory
- ✅ Handle error: negative quantities
- ✅ Handle error: future usage dates
- ✅ Handle error: duplicate emission calculations
- ✅ Handle error: invalid date ranges in reports

---

## 💡 HELPFUL TIPS

### EPA Research Sources:
- EPA official website: https://www.epa.gov/
- EPA emission factors database
- Climate registry protocols
- Energy Information Administration (EIA)

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
5. **No error handling:** Not providing helpful error messages

### Database Best Practices:
1. **Use transactions** for related operations
2. **Add indexes** on frequently queried columns
3. **Use foreign keys** to maintain data integrity
4. **Handle database errors** gracefully
5. **Use prepared statements** to prevent SQL injection

---

## 📚 RESOURCES

- **PostgreSQL Docs:** https://postgresql.org/docs/
- **EPA Official Site:** https://www.epa.gov/
- **REST API Guidelines:** Use consistent JSON responses
- **Calculator:** Use for manual verification of calculations
- **Postman:** For testing your APIs

## 🆘 WHEN TO ASK FOR HELP

Ask for help if:
- Stuck on same problem for 2+ hours
- Cannot find EPA factors for specific fuels
- Calculations don't match manual math
- Database constraints are causing issues
- APIs return unexpected results

**Remember:** You're building a real system for EPA compliance. Focus on accuracy, data integrity, and proper validation! 🚀

**Good Luck, Devanshi! You've got this! 💪**
