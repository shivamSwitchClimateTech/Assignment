# Fuel Management API Assignment 🚀

## Assignment Overview

You will build a **Fuel Management System** using our real production fuel database. we will provide you with **ONE main table containing all fuel data** and you need to filter and work with **ONLY your assigned emission standard**.

### What You'll Receive:
- **One master fuel table** with all fuel types and emission factors
- **Your specific assignment** focusing on one emission standard
- **Clear task description** of what APIs to build
- **Sample data** to get you started immediately

### What You Need to Do:
1. **Filter the fuel data** for your assigned standard
2. **Build CRUD APIs** to manage fuel inventory and usage
3. **Calculate emissions** using your standard's factors
4. **Create practical features** that our company can actually use

---

## 📊 THE MASTER FUEL TABLE

Providing you with this **complete fuel database table**. You will use this as your foundation:

### Table Structure (DDL):
```sql
-- Master fuels table - THIS IS PROVIDED TO YOU
CREATE TABLE switch.fuels (
    "_type" VARCHAR(50) NULL,
    fuel VARCHAR(255) NOT NULL PRIMARY KEY,
    unit VARCHAR(100) NULL,
    ghg_factor VARCHAR(20) NULL,
    n2ce_factor VARCHAR(20) NULL,
    emission_scope VARCHAR(20) NULL,
    ipcc_factor VARCHAR(20) NULL,
    epa_factor VARCHAR(20) NULL,
    defra_factor VARCHAR(20) NULL,
    n2ce_reference VARCHAR(255) NULL
);
```

### Sample Data (DML) - THIS IS WHAT YOU GET:
```sql
-- Complete fuel database - I WILL PROVIDE THIS TO YOU
INSERT INTO switch.fuels VALUES
-- Solid fuels
('Solid fuels', 'Anthracite', 'tonne (metric ton) (t)', '2360.492768', NULL, 'scope1', '2360.492768', '2360.492768', '2360.492768', NULL),
('Solid fuels', 'Bituminous coal', 'tonne (metric ton) (t)', '2109.2028', '1', 'scope1', '2109.2028', '2109.2028', '2109.2028', NULL),
('Solid fuels', 'Coal (domestic)', 'tonne (metric ton) (t)', '2632', NULL, 'scope1', '2632', '2632', '2632', NULL),
('Solid fuels', 'Coal (electricity generation)', 'tonne (metric ton) (t)', '2184.02', NULL, 'scope1', '2184.02', '2184.02', '2184.02', NULL),
('Solid fuels', 'Coal (industrial)', 'tonne (metric ton) (t)', '2371.91', NULL, 'scope1', '2371.91', '2371.91', '2371.91', NULL),
('Solid fuels', 'Coking coal', 'tonne (metric ton) (t)', '3144.16', NULL, 'scope1', '3144.16', '3144.16', '3144.16', NULL),

-- Liquid fuels
('Liquid fuels', 'Aviation spirit', 'Litres (L)', '2.28297', NULL, 'scope1', '2.28297', '2.28297', '2.28297', NULL),
('Liquid fuels', 'Aviation turbine fuel', 'Litres (L)', '2.51973', NULL, 'scope1', '2.51973', '2.51973', '2.51973', NULL),
('Liquid fuels', 'Burning oil', 'Litres (L)', '2.52782', NULL, 'scope1', '2.52782', '2.52782', '2.52782', NULL),
('Liquid fuels', 'Diesel (100% mineral diesel)', 'Litres (L)', '2.626', NULL, 'scope1', '2.626', '2.626', '2.626', NULL),
('Liquid fuels', 'Diesel (average biofuel blend)', 'Litres (L)', '2.47887', NULL, 'scope1', '2.47887', '2.47887', '2.47887', NULL),
('Liquid fuels', 'Fuel oil', 'Litres (L)', '3.16262', NULL, 'scope1', '3.16262', '3.16262', '3.16262', NULL),
('Liquid fuels', 'Gas oil', 'Litres (L)', '2.72417', NULL, 'scope1', '2.72417', '2.72417', '2.72417', NULL),
('Liquid fuels', 'Lubricants', 'Litres (L)', '2.74972', NULL, 'scope1', '2.74972', '2.74972', '2.74972', NULL),
('Liquid fuels', 'Marine fuel oil', 'Litres (L)', '3.06194', NULL, 'scope1', '3.06194', '3.06194', '3.06194', NULL),
('Liquid fuels', 'Marine gas oil', 'Litres (L)', '2.73782', NULL, 'scope1', '2.73782', '2.73782', '2.73782', NULL),
('Liquid fuels', 'Naphtha', 'Litres (L)', '2.11926', NULL, 'scope1', '2.11926', '2.11926', '2.11926', NULL),

-- Gaseous fuels
('Gaseous fuels', 'CNG', 'Litres (L)', '0.44757', NULL, 'scope1', '0.44757', '0.44757', '0.44757', NULL),
('Gaseous fuels', 'LNG', 'Litres (L)', '1.16604', NULL, 'scope1', '1.16604', '1.16604', '1.16604', NULL),
('Gaseous fuels', 'LPG', 'Litres (L)', '1.55491', NULL, 'scope1', '1.55491', '1.55491', '1.55491', NULL),
('Gaseous fuels', 'Natural gas', 'Cubic metres (m3)', '2.03437', NULL, 'scope1', '2.03437', '2.03437', '2.03437', NULL),
('Gaseous fuels', 'Natural gas (100% mineral blend)', 'Cubic metres (m3)', '2.04981', NULL, 'scope1', '2.04981', '2.04981', '2.04981', NULL),
('Gaseous fuels', 'Other petroleum gas', 'Litres (L)', '0.94348', NULL, 'scope1', '0.94348', '0.94348', '0.94348', NULL);

-- Additional specialty fuels
INSERT INTO switch.fuels VALUES
('Liquid fuels', 'Light Diesel Oil (LDO)', 'Litres (L)', '2.868462629', NULL, 'scope1', '2.868462629', '2.868462629', '2.868462629', NULL),
('Gaseous fuels', 'Liquid O2 MBC Cylinder 99.5% Purity', 'Cubic metres (m3)', '1', NULL, 'scope1', '1', '1', '1', NULL),
('Gaseous fuels', 'Argon Gas Cylinder', 'Cubic metres (m3)', '1', NULL, 'scope1', '1', '1', '1', NULL),
('Solid fuels', 'Imported coal', 'tonne (metric ton) (t)', '395.806', NULL, 'scope1', '395.806', '395.806', '395.806', NULL),
('Solid fuels', 'Mixed industrial waste', 'tonne (metric ton) (t)', '347.272', NULL, 'scope1', '347.272', '347.272', '347.272', NULL);
```

---

## 🎯 ASSIGNMENT 1: EPA FUEL MANAGEMENT SYSTEM
**Assigned to:** Intern 1  
**Your Standard:** **EPA (Environmental Protection Agency - USA)**

### 📝 YOUR SPECIFIC TASK:
You will work **ONLY** with fuels that have EPA emission factors. Your job is to:

1. **Filter the master table** for fuels where `epa_factor` is NOT NULL
2. **Build APIs** to manage EPA fuel inventory and usage
3. **Calculate emissions** using EPA methodology
4. **Create practical tools** for US operations

### 🎯 YOUR FUEL FOCUS:
From the master table, you'll work with these fuel types:
```sql
-- This is YOUR data - fuels with EPA factors
SELECT fuel, unit, epa_factor 
FROM switch.fuels 
WHERE epa_factor IS NOT NULL;

-- Results you'll work with:
-- Aviation spirit (2.28297 kg CO2/L)
-- Aviation turbine fuel (2.51973 kg CO2/L)
-- Diesel (100% mineral diesel) (2.626 kg CO2/L)
-- Natural gas (2.03437 kg CO2/m3)
-- LPG (1.55491 kg CO2/L)
-- ... and all other EPA fuels
```

### 🗃️ YOUR DATABASE TABLES TO CREATE:

#### Table 1: EPA Fuel Inventory
```sql
CREATE TABLE epa_fuel_inventory (
    id SERIAL PRIMARY KEY,
    facility_name VARCHAR(255) NOT NULL,
    fuel_name VARCHAR(255) NOT NULL, -- References switch.fuels(fuel)
    quantity_available DECIMAL(12,2) NOT NULL,
    unit VARCHAR(100) NOT NULL,
    storage_location VARCHAR(255),
    purchase_date DATE,
    supplier_name VARCHAR(255),
    cost_per_unit DECIMAL(10,4),
    reorder_level DECIMAL(10,2),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Ensure fuel exists in master table with EPA factor
    CONSTRAINT fk_epa_fuel FOREIGN KEY (fuel_name) REFERENCES switch.fuels(fuel)
);
```

#### Table 2: EPA Fuel Usage Tracking
```sql
CREATE TABLE epa_fuel_usage (
    id SERIAL PRIMARY KEY,
    inventory_id INTEGER NOT NULL REFERENCES epa_fuel_inventory(id),
    equipment_name VARCHAR(255) NOT NULL,
    quantity_used DECIMAL(12,2) NOT NULL,
    usage_date DATE NOT NULL,
    purpose VARCHAR(255), -- 'Heating', 'Transportation', 'Power Generation'
    operator_name VARCHAR(100),
    shift VARCHAR(20), -- 'Day', 'Night', 'Weekend'
    maintenance_due BOOLEAN DEFAULT FALSE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Table 3: EPA Emissions Results
```sql
CREATE TABLE epa_emissions_calculated (
    id SERIAL PRIMARY KEY,
    usage_id INTEGER NOT NULL REFERENCES epa_fuel_usage(id),
    fuel_name VARCHAR(255) NOT NULL,
    epa_emission_factor DECIMAL(10,6) NOT NULL,
    quantity_used DECIMAL(12,2) NOT NULL,
    unit VARCHAR(100) NOT NULL,
    co2_emissions_kg DECIMAL(15,3) NOT NULL,
    calculation_method VARCHAR(100) DEFAULT 'EPA Standard',
    calculated_by VARCHAR(100),
    calculation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    verified BOOLEAN DEFAULT FALSE
);
```

### 🔨 YOUR REQUIRED APIs:

#### 1. EPA Fuel Discovery APIs
```
GET    /api/epa/available-fuels     - Get all fuels with EPA factors
GET    /api/epa/fuel-details/:name  - Get specific fuel EPA information
GET    /api/epa/fuel-types         - Get fuel types (Liquid, Gaseous, Solid)
```

**Example Response:**
```json
GET /api/epa/available-fuels
{
    "success": true,
    "data": [
        {
            "fuel": "Diesel (100% mineral diesel)",
            "type": "Liquid fuels",
            "unit": "Litres (L)",
            "epa_factor": "2.626",
            "emission_scope": "scope1"
        },
        {
            "fuel": "Natural gas",
            "type": "Gaseous fuels", 
            "unit": "Cubic metres (m3)",
            "epa_factor": "2.03437",
            "emission_scope": "scope1"
        }
    ]
}
```

#### 2. Fuel Inventory Management APIs
```
POST   /api/epa/inventory          - Add fuel to inventory
GET    /api/epa/inventory          - Get all inventory
GET    /api/epa/inventory/:id      - Get specific inventory item
PUT    /api/epa/inventory/:id      - Update inventory
DELETE /api/epa/inventory/:id      - Remove from inventory
GET    /api/epa/inventory/alerts   - Get low stock alerts
GET    /api/epa/inventory/facility/:name - Get facility inventory
```

**Example Request:**
```json
POST /api/epa/inventory
{
    "facility_name": "Houston Manufacturing Plant",
    "fuel_name": "Diesel (100% mineral diesel)",
    "quantity_available": 5000.00,
    "unit": "Litres (L)",
    "storage_location": "Tank A-1",
    "supplier_name": "Shell Energy USA",
    "cost_per_unit": 1.25,
    "reorder_level": 1000.00
}
```

#### 3. Fuel Usage Tracking APIs
```
POST   /api/epa/usage              - Record fuel usage
GET    /api/epa/usage              - Get all usage records
GET    /api/epa/usage/:id          - Get specific usage
PUT    /api/epa/usage/:id          - Update usage record
GET    /api/epa/usage/equipment/:name - Get usage by equipment
GET    /api/epa/usage/period       - Get usage by date range
```

#### 4. EPA Emission Calculation APIs
```
POST   /api/epa/calculate-emissions - Calculate emissions for usage
GET    /api/epa/emissions/:id      - Get emission calculation
GET    /api/epa/emissions/total    - Get total facility emissions
GET    /api/epa/emissions/monthly  - Get monthly emissions report
GET    /api/epa/emissions/verify/:id - Verify emission calculation
```

**Calculation Logic:**
```
EPA CO2 Emissions (kg) = Fuel Quantity × EPA Emission Factor

Example:
150.5 Litres of Diesel × 2.626 kg CO2/L = 395.213 kg CO2
```

### 💡 SAMPLE DATA TO INSERT:
```sql
-- Sample EPA fuel inventory
INSERT INTO epa_fuel_inventory (facility_name, fuel_name, quantity_available, unit, storage_location, supplier_name, cost_per_unit, reorder_level) VALUES
('Houston Plant', 'Diesel (100% mineral diesel)', 5000.00, 'Litres (L)', 'Tank A-1', 'Shell Energy USA', 1.25, 1000.00),
('Houston Plant', 'Natural gas', 15000.00, 'Cubic metres (m3)', 'Gas Line 1', 'Texas Gas Corp', 0.45, 3000.00),
('Dallas Office', 'Natural gas', 2500.00, 'Cubic metres (m3)', 'Building Main', 'Dallas Gas & Electric', 0.48, 500.00),
('Austin Warehouse', 'LPG', 800.00, 'Litres (L)', 'LPG Tank 1', 'AmeriGas', 0.85, 200.00);

-- Sample fuel usage
INSERT INTO epa_fuel_usage (inventory_id, equipment_name, quantity_used, usage_date, purpose, operator_name, shift) VALUES
(1, 'Generator Unit 3', 150.5, '2024-08-10', 'Emergency Power', 'John Smith', 'Day'),
(2, 'Boiler System A', 450.0, '2024-08-10', 'Steam Generation', 'Maria Garcia', 'Day'),
(1, 'Forklift Fleet', 85.2, '2024-08-11', 'Material Handling', 'Mike Johnson', 'Night');
```

### 🎮 SPECIAL FEATURES TO BUILD:

1. **Smart Alerts System:**
   - Low stock warnings when inventory < reorder_level
   - High usage alerts (when daily usage > average × 1.5)
   - Equipment maintenance reminders

2. **EPA Cost Calculator:**
   - Calculate fuel costs vs emissions
   - Compare different EPA fuels for same purpose
   - Monthly cost analysis

3. **Usage Efficiency Tracker:**
   - Track fuel efficiency by equipment
   - Identify high-consumption equipment
   - Suggest optimization opportunities

4. **EPA Compliance Reports:**
   - Monthly EPA emissions summary
   - Facility-wise emission breakdown
   - Year-over-year comparison

---

## 🎯 ASSIGNMENT 2: DEFRA FUEL MANAGEMENT SYSTEM
**Assigned to:** Intern 2  
**Your Standard:** **DEFRA (UK Department for Environment, Food & Rural Affairs)**

### 📝 YOUR SPECIFIC TASK:
You will work **ONLY** with fuels that have DEFRA emission factors. Your job is to:

1. **Filter the master table** for fuels where `defra_factor` is NOT NULL
2. **Build APIs** to manage DEFRA fuel operations for UK facilities
3. **Calculate emissions** using DEFRA methodology
4. **Create UK-specific reporting tools**

### 🎯 YOUR FUEL FOCUS:
```sql
-- This is YOUR data - fuels with DEFRA factors
SELECT fuel, unit, defra_factor 
FROM switch.fuels 
WHERE defra_factor IS NOT NULL;

-- You'll work with the same fuels but focus on DEFRA methodology
-- Gas oil (2.72417 kg CO2/L) - UK specific
-- Burning oil (2.52782 kg CO2/L) - UK heating
-- Natural gas (2.03437 kg CO2/m3) - UK commercial
```

### 🗃️ YOUR DATABASE TABLES TO CREATE:

#### Table 1: UK Facility Fuel Management
```sql
CREATE TABLE defra_facility_fuel (
    id SERIAL PRIMARY KEY,
    uk_site_name VARCHAR(255) NOT NULL,
    site_postcode VARCHAR(10) NOT NULL, -- UK postcodes
    fuel_name VARCHAR(255) NOT NULL REFERENCES switch.fuels(fuel),
    storage_capacity DECIMAL(12,2) NOT NULL,
    current_stock DECIMAL(12,2) NOT NULL,
    unit VARCHAR(100) NOT NULL,
    storage_type VARCHAR(100), -- 'Underground Tank', 'Above Ground', 'Gas Connection'
    last_delivery_date DATE,
    next_delivery_scheduled DATE,
    supplier_name VARCHAR(255),
    contract_rate DECIMAL(10,4), -- £ per unit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Table 2: DEFRA Fuel Consumption
```sql
CREATE TABLE defra_fuel_consumption (
    id SERIAL PRIMARY KEY,
    facility_fuel_id INTEGER NOT NULL REFERENCES defra_facility_fuel(id),
    consumption_date DATE NOT NULL,
    meter_reading_start DECIMAL(10,2),
    meter_reading_end DECIMAL(10,2),
    quantity_consumed DECIMAL(12,2) NOT NULL,
    department VARCHAR(100), -- 'Production', 'Office Heating', 'Fleet'
    invoice_number VARCHAR(100),
    consumption_purpose VARCHAR(255),
    recorded_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Table 3: DEFRA Carbon Footprint Results
```sql
CREATE TABLE defra_carbon_footprint (
    id SERIAL PRIMARY KEY,
    consumption_id INTEGER NOT NULL REFERENCES defra_fuel_consumption(id),
    fuel_name VARCHAR(255) NOT NULL,
    defra_emission_factor DECIMAL(10,6) NOT NULL,
    quantity_consumed DECIMAL(12,2) NOT NULL,
    co2_emissions_kg DECIMAL(15,3) NOT NULL,
    reporting_period VARCHAR(20), -- 'Q1-2024', 'Apr-2024'
    calculated_by VARCHAR(100),
    calculation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    defra_compliant BOOLEAN DEFAULT TRUE
);
```

### 💡 SAMPLE DATA FOR DEFRA:
```sql
-- UK facility data
INSERT INTO defra_facility_fuel (uk_site_name, site_postcode, fuel_name, storage_capacity, current_stock, unit, storage_type, supplier_name, contract_rate) VALUES
('Manchester Office', 'M1 1AA', 'Natural gas', 10000.00, 7500.00, 'Cubic metres (m3)', 'Gas Connection', 'British Gas', 0.52),
('Birmingham Plant', 'B1 1BB', 'Gas oil', 3000.00, 2100.00, 'Litres (L)', 'Underground Tank', 'Shell UK', 1.45),
('London Headquarters', 'SW1 1CC', 'Natural gas', 5000.00, 3800.00, 'Cubic metres (m3)', 'Gas Connection', 'EDF Energy', 0.48);

-- UK consumption data
INSERT INTO defra_fuel_consumption (facility_fuel_id, consumption_date, quantity_consumed, department, consumption_purpose, recorded_by) VALUES
(1, '2024-08-10', 125.5, 'Office Heating', 'Building Climate Control', 'Sarah Wilson'),
(2, '2024-08-10', 245.0, 'Production', 'Manufacturing Process', 'James Brown'),
(1, '2024-08-11', 98.2, 'Canteen', 'Kitchen Operations', 'Mary Jones');
```

---

## 🎯 ASSIGNMENT 3: IPCC FUEL MANAGEMENT SYSTEM
**Assigned to:** Intern 3  
**Your Standard:** **IPCC (Intergovernmental Panel on Climate Change)**

### 📝 YOUR SPECIFIC TASK:
You will work **ONLY** with fuels that have IPCC emission factors. Your job is to:

1. **Filter the master table** for fuels where `ipcc_factor` is NOT NULL
2. **Build APIs** for international fuel operations and climate reporting
3. **Calculate emissions** using IPCC methodology
4. **Create global climate impact tools**

### 🗃️ YOUR DATABASE TABLES TO CREATE:

#### Table 1: Global Operations Fuel Management
```sql
CREATE TABLE ipcc_global_operations (
    id SERIAL PRIMARY KEY,
    country_code CHAR(3) NOT NULL, -- 'USA', 'GBR', 'DEU', 'IND'
    facility_name VARCHAR(255) NOT NULL,
    fuel_name VARCHAR(255) NOT NULL REFERENCES switch.fuels(fuel),
    annual_consumption DECIMAL(15,3) NOT NULL,
    unit VARCHAR(100) NOT NULL,
    operation_type VARCHAR(100), -- 'Manufacturing', 'Transport', 'Energy'
    reporting_year INTEGER NOT NULL,
    data_quality VARCHAR(50) DEFAULT 'Measured', -- 'Measured', 'Estimated', 'Calculated'
    local_currency CHAR(3), -- 'USD', 'GBP', 'EUR'
    fuel_cost_local DECIMAL(12,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 💡 SAMPLE DATA FOR IPCC:
```sql
-- Global operations data
INSERT INTO ipcc_global_operations (country_code, facility_name, fuel_name, annual_consumption, unit, operation_type, reporting_year, local_currency, fuel_cost_local) VALUES
('USA', 'Houston Manufacturing', 'Aviation turbine fuel', 25000.00, 'Litres (L)', 'Corporate Aviation', 2024, 'USD', 31250.00),
('GBR', 'London Office Fleet', 'Diesel (100% mineral diesel)', 12000.00, 'Litres (L)', 'Transportation', 2024, 'GBP', 16800.00),
('DEU', 'Berlin Plant', 'Natural gas', 45000.00, 'Cubic metres (m3)', 'Manufacturing', 2024, 'EUR', 23400.00),
('IND', 'Mumbai Operations', 'LPG', 8500.00, 'Litres (L)', 'Process Heat', 2024, 'INR', 765000.00);
```

---

## 🎯 ASSIGNMENT 4: GHG PROTOCOL FUEL MANAGEMENT SYSTEM
**Assigned to:** Intern 4  
**Your Standard:** **GHG Protocol (Greenhouse Gas Protocol)**

### 📝 YOUR SPECIFIC TASK:
You will work **ONLY** with fuels that have GHG Protocol emission factors. Your job is to:

1. **Filter the master table** for fuels where `ghg_factor` is NOT NULL
2. **Build APIs** for corporate sustainability reporting
3. **Calculate emissions** using GHG Protocol methodology
4. **Create corporate sustainability tools**

### 🗃️ YOUR DATABASE TABLES TO CREATE:

#### Table 1: Corporate Sustainability Programs
```sql
CREATE TABLE ghg_corporate_programs (
    id SERIAL PRIMARY KEY,
    program_name VARCHAR(255) NOT NULL,
    business_unit VARCHAR(255) NOT NULL,
    fuel_name VARCHAR(255) NOT NULL REFERENCES switch.fuels(fuel),
    annual_target_consumption DECIMAL(12,2),
    co2_reduction_target_kg DECIMAL(15,2), -- Annual reduction target
    program_manager VARCHAR(255),
    start_date DATE,
    end_date DATE,
    budget_allocated DECIMAL(12,2),
    program_status VARCHAR(50) DEFAULT 'Active', -- 'Active', 'Paused', 'Completed'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 💡 SAMPLE DATA FOR GHG PROTOCOL:
```sql
-- Corporate programs data
INSERT INTO ghg_corporate_programs (program_name, business_unit, fuel_name, annual_target_consumption, co2_reduction_target_kg, program_manager, start_date, budget_allocated) VALUES
('Green Fleet Initiative', 'Operations', 'Diesel (100% mineral diesel)', 10000.00, 2626.00, 'Sarah Johnson', '2024-01-01', 125000.00),
('Facility Heating Optimization', 'Facilities', 'Natural gas', 20000.00, 4068.74, 'Mike Chen', '2024-01-01', 85000.00),
('Aviation Carbon Offset', 'Corporate', 'Aviation turbine fuel', 5000.00, 1259.865, 'Lisa Wang', '2024-01-01', 200000.00);
```

---

## 🚀 GETTING STARTED - STEP BY STEP GUIDE

### Step 1: Database Connection & Exploration
1. **Connect to PostgreSQL** using the credentials I'll provide
2. **Explore the master fuel table:**
   ```sql
   -- See all available data
   SELECT * FROM switch.fuels LIMIT 10;
   
   -- Find YOUR standard's fuels (replace 'epa' with your standard)
   SELECT fuel, unit, epa_factor 
   FROM switch.fuels 
   WHERE epa_factor IS NOT NULL
   ORDER BY "_type", fuel;
   ```

### Step 2: Create Your Additional Tables
3. **Create your tables** using the DDL provided for your assignment
4. **Insert sample data** using the DML provided
5. **Test your tables** with basic SELECT queries

### Step 3: Build Basic CRUD APIs
6. **Start with fuel discovery APIs** - let users see available fuels
7. **Build inventory management** - add, view, update fuel records
8. **Add usage tracking** - record when fuels are consumed
9. **Implement emission calculations** - the core business logic

### Step 4: Test Your APIs
10. **Use Postman or similar** to test each endpoint
11. **Verify calculations** - make sure math is correct
12. **Handle errors gracefully** - what happens with bad data?

### Step 5: Add Special Features
13. **Build the fun extras** mentioned in your assignment
14. **Create useful reports** that managers would actually want
15. **Add data validations** to prevent errors

## ✅ SUCCESS CRITERIA

You'll know you've succeeded when:
- ✅ You can filter the master fuel table for YOUR standard
- ✅ All your CRUD APIs work correctly  
- ✅ Emission calculations produce accurate results
- ✅ Your system can handle realistic data volumes
- ✅ Error handling works properly
- ✅ You can generate useful business reports
- ✅ Other developers can understand your code

## 📊 FINAL DELIVERABLES

1. **Working APIs** - All endpoints functioning correctly
2. **Database Setup** - Tables created with sample data
3. **Documentation** - README with setup and usage instructions  
4. **Test Cases** - Basic tests proving functionality
5. **Demo** - Working demonstration of key features

## 💬 Questions & Support

If you get stuck:
1. **Check the master fuel table** - make sure you understand the data structure
2. **Verify your filtering** - ensure you're only working with your standard's fuels
3. **Test calculations manually** - verify emission math before coding
4. **Ask for help** - don't spend hours stuck on one issue

Remember: You're building something **REAL** that our company will actually use. Focus on making it practical and useful! 🌟
