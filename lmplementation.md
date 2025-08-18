# DEFRA API 5: UK Carbon Report - Complete Architecture & Implementation Guide

## 🎯 API Overview
**Endpoint:** `GET /api/defra/report`
**Purpose:** Generate comprehensive carbon footprint report for UK site and time period
**Core Logic:** Aggregate emission data across multiple dimensions (fuel, department, equipment)

## 📊 Required Query Parameters
```
GET /api/defra/report?site=Manchester Office&start_date=2024-08-01&end_date=2024-08-31
```

### Parameter Validation
- `site` (required): UK site name - must exist in defra_inventory
- `start_date` (required): Format YYYY-MM-DD
- `end_date` (required): Format YYYY-MM-DD, must be >= start_date

## 🏗️ Database Architecture & Query Strategy

### Main Data Flow
1. **Input Validation** → Validate parameters
2. **Site Verification** → Check if site exists
3. **Data Aggregation** → Multiple complex queries
4. **Response Assembly** → Structure the final JSON

### Core Queries You Need

#### Query 1: Main Report Data
```sql
SELECT 
    di.site_name,
    di.site_postcode,
    dc.consumption_date,
    dc.quantity_consumed,
    dc.department,
    dc.equipment_name,
    mf.fuel_name,
    mf.fuel_type,
    mf.unit,
    de.defra_factor,
    de.co2_emissions_kg,
    di.cost_per_unit,
    (dc.quantity_consumed * di.cost_per_unit) as consumption_cost
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
WHERE di.site_name = ? 
    AND dc.consumption_date BETWEEN ? AND ?
ORDER BY dc.consumption_date DESC;
```

#### Query 2: Summary Totals
```sql
SELECT 
    COUNT(dc.id) as total_activities,
    SUM(de.co2_emissions_kg) as total_emissions_kg,
    SUM(dc.quantity_consumed) as total_fuel_consumed,
    SUM(dc.quantity_consumed * di.cost_per_unit) as total_cost_gbp,
    AVG(de.co2_emissions_kg) as avg_emissions_per_activity
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
WHERE di.site_name = ? 
    AND dc.consumption_date BETWEEN ? AND ?;
```

#### Query 3: Fuel Breakdown
```sql
SELECT 
    mf.fuel_name,
    mf.fuel_type,
    mf.unit,
    SUM(dc.quantity_consumed) as total_quantity,
    de.defra_factor,
    SUM(de.co2_emissions_kg) as total_emissions,
    SUM(dc.quantity_consumed * di.cost_per_unit) as total_cost
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
WHERE di.site_name = ? 
    AND dc.consumption_date BETWEEN ? AND ?
GROUP BY mf.fuel_name, mf.fuel_type, mf.unit, de.defra_factor
ORDER BY total_emissions DESC;
```

#### Query 4: Department Breakdown
```sql
SELECT 
    dc.department,
    SUM(de.co2_emissions_kg) as total_emissions,
    SUM(dc.quantity_consumed * di.cost_per_unit) as total_cost,
    COUNT(dc.id) as activity_count
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
WHERE di.site_name = ? 
    AND dc.consumption_date BETWEEN ? AND ?
GROUP BY dc.department
ORDER BY total_emissions DESC;
```

#### Query 5: Equipment Breakdown
```sql
SELECT 
    dc.equipment_name,
    mf.fuel_name,
    SUM(de.co2_emissions_kg) as total_emissions,
    COUNT(dc.id) as usage_count
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
WHERE di.site_name = ? 
    AND dc.consumption_date BETWEEN ? AND ?
    AND dc.equipment_name IS NOT NULL
GROUP BY dc.equipment_name, mf.fuel_name
ORDER BY total_emissions DESC;
```

## 🔧 Implementation Steps

### Step 1: Controller Setup
```javascript
// GET /api/defra/report
async function getUKCarbonReport(req, res) {
    try {
        // 1. Extract and validate query parameters
        const { site, start_date, end_date } = req.query;
        
        // 2. Validate required parameters
        const validation = validateReportParams(site, start_date, end_date);
        if (!validation.isValid) {
            return res.status(400).json({
                success: false,
                message: "Validation failed",
                errors: validation.errors
            });
        }
        
        // 3. Execute report generation
        const reportData = await generateCarbonReport(site, start_date, end_date);
        
        // 4. Return formatted response
        return res.status(200).json({
            success: true,
            message: "UK carbon report generated successfully",
            data: reportData
        });
        
    } catch (error) {
        console.error('Report generation error:', error);
        return res.status(500).json({
            success: false,
            message: "Failed to generate carbon report",
            error: error.message
        });
    }
}
```

### Step 2: Validation Function
```javascript
function validateReportParams(site, start_date, end_date) {
    const errors = [];
    
    // Check required parameters
    if (!site) errors.push({ field: "site", message: "Site name is required" });
    if (!start_date) errors.push({ field: "start_date", message: "Start date is required" });
    if (!end_date) errors.push({ field: "end_date", message: "End date is required" });
    
    // Validate date formats
    if (start_date && !isValidDate(start_date)) {
        errors.push({ field: "start_date", message: "Invalid date format. Use YYYY-MM-DD" });
    }
    if (end_date && !isValidDate(end_date)) {
        errors.push({ field: "end_date", message: "Invalid date format. Use YYYY-MM-DD" });
    }
    
    // Validate date logic
    if (start_date && end_date && new Date(start_date) > new Date(end_date)) {
        errors.push({ field: "date_range", message: "Start date must be before end date" });
    }
    
    return {
        isValid: errors.length === 0,
        errors: errors
    };
}

function isValidDate(dateString) {
    const regex = /^\d{4}-\d{2}-\d{2}$/;
    if (!regex.test(dateString)) return false;
    
    const date = new Date(dateString);
    return date instanceof Date && !isNaN(date);
}
```

### Step 3: Main Report Generation Function
```javascript
async function generateCarbonReport(site, start_date, end_date) {
    const client = await pool.connect();
    
    try {
        // 1. Check if site exists
        const siteCheck = await client.query(
            'SELECT DISTINCT site_name, site_postcode FROM defra_inventory WHERE site_name = $1',
            [site]
        );
        
        if (siteCheck.rows.length === 0) {
            throw new Error(`Site '${site}' not found in inventory`);
        }
        
        const siteInfo = siteCheck.rows[0];
        
        // 2. Get summary data
        const summary = await getSummaryData(client, site, start_date, end_date);
        
        // 3. Get fuel breakdown
        const fuelBreakdown = await getFuelBreakdown(client, site, start_date, end_date, summary.total_emissions_kg);
        
        // 4. Get department breakdown
        const departmentBreakdown = await getDepartmentBreakdown(client, site, start_date, end_date, summary.total_emissions_kg);
        
        // 5. Get equipment breakdown
        const equipmentBreakdown = await getEquipmentBreakdown(client, site, start_date, end_date);
        
        // 6. Calculate average daily emissions
        const daysDiff = Math.ceil((new Date(end_date) - new Date(start_date)) / (1000 * 60 * 60 * 24)) + 1;
        const averageDailyEmissions = summary.total_emissions_kg / daysDiff;
        
        // 7. Assemble final report
        return {
            site_name: siteInfo.site_name,
            site_postcode: siteInfo.site_postcode,
            period: {
                start_date: start_date,
                end_date: end_date
            },
            summary: {
                total_emissions_kg: parseFloat(summary.total_emissions_kg) || 0,
                total_fuel_consumed: parseFloat(summary.total_fuel_consumed) || 0,
                total_activities: parseInt(summary.total_activities) || 0,
                average_daily_emissions: parseFloat(averageDailyEmissions.toFixed(2)),
                total_cost_gbp: parseFloat(summary.total_cost_gbp) || 0
            },
            fuel_breakdown: fuelBreakdown,
            department_breakdown: departmentBreakdown,
            equipment_breakdown: equipmentBreakdown
        };
        
    } finally {
        client.release();
    }
}
```

### Step 4: Helper Functions

#### Summary Data Function
```javascript
async function getSummaryData(client, site, start_date, end_date) {
    const query = `
        SELECT 
            COUNT(dc.id) as total_activities,
            COALESCE(SUM(de.co2_emissions_kg), 0) as total_emissions_kg,
            COALESCE(SUM(dc.quantity_consumed), 0) as total_fuel_consumed,
            COALESCE(SUM(dc.quantity_consumed * di.cost_per_unit), 0) as total_cost_gbp
        FROM defra_inventory di
        JOIN defra_consumption dc ON di.id = dc.inventory_id
        JOIN defra_emissions de ON dc.id = de.consumption_id
        WHERE di.site_name = $1 
            AND dc.consumption_date BETWEEN $2 AND $3
    `;
    
    const result = await client.query(query, [site, start_date, end_date]);
    return result.rows[0];
}
```

#### Fuel Breakdown Function
```javascript
async function getFuelBreakdown(client, site, start_date, end_date, totalEmissions) {
    const query = `
        SELECT 
            mf.fuel_name,
            mf.fuel_type,
            mf.unit,
            SUM(dc.quantity_consumed) as quantity_consumed,
            de.defra_factor,
            SUM(de.co2_emissions_kg) as emissions_kg,
            SUM(dc.quantity_consumed * di.cost_per_unit) as cost_gbp
        FROM defra_inventory di
        JOIN defra_consumption dc ON di.id = dc.inventory_id
        JOIN defra_emissions de ON dc.id = de.consumption_id
        JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
        WHERE di.site_name = $1 
            AND dc.consumption_date BETWEEN $2 AND $3
        GROUP BY mf.fuel_name, mf.fuel_type, mf.unit, de.defra_factor
        ORDER BY emissions_kg DESC
    `;
    
    const result = await client.query(query, [site, start_date, end_date]);
    
    return result.rows.map(row => ({
        fuel_name: row.fuel_name,
        fuel_type: row.fuel_type,
        quantity_consumed: parseFloat(row.quantity_consumed),
        unit: row.unit,
        defra_factor: parseFloat(row.defra_factor),
        emissions_kg: parseFloat(row.emissions_kg),
        percentage_of_total: totalEmissions > 0 ? parseFloat(((row.emissions_kg / totalEmissions) * 100).toFixed(1)) : 0,
        cost_gbp: parseFloat(row.cost_gbp)
    }));
}
```

#### Department Breakdown Function
```javascript
async function getDepartmentBreakdown(client, site, start_date, end_date, totalEmissions) {
    const query = `
        SELECT 
            dc.department,
            SUM(de.co2_emissions_kg) as emissions_kg,
            SUM(dc.quantity_consumed * di.cost_per_unit) as cost_gbp
        FROM defra_inventory di
        JOIN defra_consumption dc ON di.id = dc.inventory_id
        JOIN defra_emissions de ON dc.id = de.consumption_id
        WHERE di.site_name = $1 
            AND dc.consumption_date BETWEEN $2 AND $3
        GROUP BY dc.department
        ORDER BY emissions_kg DESC
    `;
    
    const result = await client.query(query, [site, start_date, end_date]);
    
    return result.rows.map(row => ({
        department: row.department,
        emissions_kg: parseFloat(row.emissions_kg),
        percentage_of_total: totalEmissions > 0 ? parseFloat(((row.emissions_kg / totalEmissions) * 100).toFixed(1)) : 0,
        cost_gbp: parseFloat(row.cost_gbp)
    }));
}
```

#### Equipment Breakdown Function
```javascript
async function getEquipmentBreakdown(client, site, start_date, end_date) {
    const query = `
        SELECT 
            dc.equipment_name,
            mf.fuel_name,
            SUM(de.co2_emissions_kg) as emissions_kg
        FROM defra_inventory di
        JOIN defra_consumption dc ON di.id = dc.inventory_id
        JOIN defra_emissions de ON dc.id = de.consumption_id
        JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
        WHERE di.site_name = $1 
            AND dc.consumption_date BETWEEN $2 AND $3
            AND dc.equipment_name IS NOT NULL
        GROUP BY dc.equipment_name, mf.fuel_name
        ORDER BY emissions_kg DESC
    `;
    
    const result = await client.query(query, [site, start_date, end_date]);
    
    return result.rows.map(row => ({
        equipment_name: row.equipment_name,
        fuel_used: row.fuel_name,
        emissions_kg: parseFloat(row.emissions_kg)
    }));
}
```

## 🧪 Testing Strategy

### Test Cases to Implement

#### 1. Happy Path Test
```javascript
// Test with valid data
GET /api/defra/report?site=Manchester Office&start_date=2024-08-01&end_date=2024-08-31
```

#### 2. Error Handling Tests
```javascript
// Missing parameters
GET /api/defra/report?site=Manchester Office

// Invalid date format
GET /api/defra/report?site=Manchester Office&start_date=2024-13-01&end_date=2024-08-31

// Invalid date range
GET /api/defra/report?site=Manchester Office&start_date=2024-08-31&end_date=2024-08-01

// Non-existent site
GET /api/defra/report?site=Unknown Site&start_date=2024-08-01&end_date=2024-08-31
```

#### 3. Edge Case Tests
```javascript
// No data in date range
GET /api/defra/report?site=Manchester Office&start_date=2025-01-01&end_date=2025-01-31

// Single day report
GET /api/defra/report?site=Manchester Office&start_date=2024-08-10&end_date=2024-08-10
```

## 🔍 Common Issues & Solutions

### Issue 1: Complex JOIN Performance
**Solution:** Add proper indexes
```sql
CREATE INDEX idx_consumption_site_date ON defra_consumption(inventory_id, consumption_date);
CREATE INDEX idx_emissions_consumption ON defra_emissions(consumption_id);
```

### Issue 2: Decimal Precision
**Solution:** Use parseFloat() and toFixed() consistently
```javascript
emissions_kg: parseFloat(row.emissions_kg.toFixed(3))
```

### Issue 3: Division by Zero
**Solution:** Always check for zero before calculating percentages
```javascript
percentage_of_total: totalEmissions > 0 ? parseFloat(((row.emissions_kg / totalEmissions) * 100).toFixed(1)) : 0
```

### Issue 4: Null/Undefined Values
**Solution:** Use COALESCE in SQL and null checks in JavaScript
```sql
COALESCE(SUM(de.co2_emissions_kg), 0) as total_emissions_kg
```

## 📝 Implementation Checklist

- [ ] Set up route handler with parameter extraction
- [ ] Implement comprehensive parameter validation
- [ ] Create database connection with proper error handling
- [ ] Write and test all 5 core SQL queries
- [ ] Implement summary data aggregation
- [ ] Implement fuel breakdown with percentages
- [ ] Implement department breakdown with percentages
- [ ] Implement equipment breakdown
- [ ] Add proper number formatting and rounding
- [ ] Implement error handling for all scenarios
- [ ] Test with sample data
- [ ] Verify calculations manually
- [ ] Add performance indexes
- [ ] Test edge cases (no data, single record, etc.)
- [ ] Document API usage examples

## 🎯 Success Metrics

Your API is successful when:
1. ✅ Returns accurate aggregated data across all dimensions
2. ✅ Handles all error cases gracefully
3. ✅ Calculates percentages correctly
4. ✅ Performs well with 100+ records
5. ✅ Validates all input parameters properly
6. ✅ Matches the exact JSON structure from requirements
7. ✅ Handles edge cases (no data, single day, etc.)

This comprehensive architecture will help you build a robust and accurate UK carbon reporting system! 🇬🇧
