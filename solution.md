# DEFRA API 5

## 📊 Sample Test Data Setup

Before showing query outputs, here's the test data we'll use:

### Sample Inventory Data
```sql
-- Sample data already in your database
INSERT INTO defra_inventory (id, site_name, site_postcode, fuel_name, quantity, unit, cost_per_unit, supplier, storage_location) VALUES
(1, 'Manchester Office', 'M1 1AA', 'Natural Gas', 10000.0, 'Cubic metres (m3)', 0.52, 'British Gas', 'Gas Connection Main'),
(2, 'Manchester Office', 'M1 1AA', 'LPG', 500.0, 'Litres (L)', 0.78, 'Calor Gas', 'LPG Storage Bay'),
(3, 'Manchester Office', 'M1 1AA', 'Diesel', 1000.0, 'Litres (L)', 1.45, 'Shell UK', 'Underground Tank A');
```

### Sample Consumption Data
```sql
INSERT INTO defra_consumption (id, inventory_id, consumption_date, quantity_consumed, department, equipment_name, recorded_by) VALUES
(1, 1, '2024-08-10', 700.0, 'Office Heating', 'Boiler System A', 'Sarah Wilson'),
(2, 2, '2024-08-11', 150.0, 'Catering', 'Kitchen Equipment', 'Mary Jones'),
(3, 3, '2024-08-12', 200.0, 'Production', 'Generator Unit', 'James Brown'),
(4, 1, '2024-08-15', 300.0, 'Office Heating', 'HVAC System', 'David Smith'),
(5, 2, '2024-08-20', 50.0, 'Production', 'Forklift Fleet', 'Lisa Taylor');
```

### Sample Emissions Data (Calculated using DEFRA factors)
```sql
INSERT INTO defra_emissions (id, consumption_id, fuel_name, quantity_consumed, defra_factor, co2_emissions_kg, reporting_period) VALUES
(1, 1, 'Natural Gas', 700.0, 2.0344, 1424.08, 'Q3-2024'),
(2, 2, 'LPG', 150.0, 1.5549, 233.24, 'Q3-2024'),
(3, 3, 'Diesel', 200.0, 2.6850, 537.00, 'Q3-2024'),
(4, 4, 'Natural Gas', 300.0, 2.0344, 610.32, 'Q3-2024'),
(5, 5, 'LPG', 50.0, 1.5549, 77.75, 'Q3-2024');
```

---

## 🔍 Query Examples with Expected Outputs

### Query 1: Site Verification
```sql
SELECT DISTINCT site_name, site_postcode 
FROM defra_inventory 
WHERE site_name = 'Manchester Office';
```

**Expected Output:**
```json
{
  "rows": [
    {
      "site_name": "Manchester Office",
      "site_postcode": "M1 1AA"
    }
  ]
}
```

---

### Query 2: Summary Data
```sql
SELECT 
    COUNT(dc.id) as total_activities,
    COALESCE(SUM(de.co2_emissions_kg), 0) as total_emissions_kg,
    COALESCE(SUM(dc.quantity_consumed), 0) as total_fuel_consumed,
    COALESCE(SUM(dc.quantity_consumed * di.cost_per_unit), 0) as total_cost_gbp
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
WHERE di.site_name = 'Manchester Office' 
    AND dc.consumption_date BETWEEN '2024-08-01' AND '2024-08-31';
```

**Expected Output:**
```json
{
  "rows": [
    {
      "total_activities": "5",
      "total_emissions_kg": "2882.39",
      "total_fuel_consumed": "1400.00",
      "total_cost_gbp": "729.00"
    }
  ]
}
```

**Calculation Verification:**
- Activities: 5 consumption records
- Emissions: 1424.08 + 233.24 + 537.00 + 610.32 + 77.75 = 2882.39 kg
- Fuel: 700 + 150 + 200 + 300 + 50 = 1400 units
- Cost: (700×0.52) + (150×0.78) + (200×1.45) + (300×0.52) + (50×0.78) = 364 + 117 + 290 + 156 + 39 = 966 GBP

---

### Query 3: Fuel Breakdown
```sql
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
WHERE di.site_name = 'Manchester Office' 
    AND dc.consumption_date BETWEEN '2024-08-01' AND '2024-08-31'
GROUP BY mf.fuel_name, mf.fuel_type, mf.unit, de.defra_factor
ORDER BY emissions_kg DESC;
```

**Expected Output:**
```json
{
  "rows": [
    {
      "fuel_name": "Natural Gas",
      "fuel_type": "Gaseous fuels",
      "unit": "Cubic metres (m3)",
      "quantity_consumed": "1000.00",
      "defra_factor": "2.0344",
      "emissions_kg": "2034.40",
      "cost_gbp": "520.00"
    },
    {
      "fuel_name": "Diesel",
      "fuel_type": "Liquid fuels",
      "unit": "Litres (L)",
      "quantity_consumed": "200.00",
      "defra_factor": "2.6850",
      "emissions_kg": "537.00",
      "cost_gbp": "290.00"
    },
    {
      "fuel_name": "LPG",
      "fuel_type": "Gaseous fuels",
      "unit": "Litres (L)",
      "quantity_consumed": "200.00",
      "defra_factor": "1.5549",
      "emissions_kg": "310.98",
      "cost_gbp": "156.00"
    }
  ]
}
```

**After JavaScript Processing (with percentages):**
```json
[
  {
    "fuel_name": "Natural Gas",
    "fuel_type": "Gaseous fuels",
    "quantity_consumed": 1000.0,
    "unit": "Cubic metres (m3)",
    "defra_factor": 2.0344,
    "emissions_kg": 2034.40,
    "percentage_of_total": 70.6,
    "cost_gbp": 520.00
  },
  {
    "fuel_name": "Diesel",
    "fuel_type": "Liquid fuels",
    "quantity_consumed": 200.0,
    "unit": "Litres (L)",
    "defra_factor": 2.6850,
    "emissions_kg": 537.00,
    "percentage_of_total": 18.6,
    "cost_gbp": 290.00
  },
  {
    "fuel_name": "LPG",
    "fuel_type": "Gaseous fuels",
    "quantity_consumed": 200.0,
    "unit": "Litres (L)",
    "defra_factor": 1.5549,
    "emissions_kg": 310.98,
    "percentage_of_total": 10.8,
    "cost_gbp": 156.00
  }
]
```

---

### Query 4: Department Breakdown
```sql
SELECT 
    dc.department,
    SUM(de.co2_emissions_kg) as emissions_kg,
    SUM(dc.quantity_consumed * di.cost_per_unit) as cost_gbp
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
WHERE di.site_name = 'Manchester Office' 
    AND dc.consumption_date BETWEEN '2024-08-01' AND '2024-08-31'
GROUP BY dc.department
ORDER BY emissions_kg DESC;
```

**Expected Output:**
```json
{
  "rows": [
    {
      "department": "Office Heating",
      "emissions_kg": "2034.40",
      "cost_gbp": "520.00"
    },
    {
      "department": "Production",
      "emissions_kg": "614.75",
      "cost_gbp": "329.00"
    },
    {
      "department": "Catering",
      "emissions_kg": "233.24",
      "cost_gbp": "117.00"
    }
  ]
}
```

**After JavaScript Processing (with percentages):**
```json
[
  {
    "department": "Office Heating",
    "emissions_kg": 2034.40,
    "percentage_of_total": 70.6,
    "cost_gbp": 520.00
  },
  {
    "department": "Production",
    "emissions_kg": 614.75,
    "percentage_of_total": 21.3,
    "cost_gbp": 329.00
  },
  {
    "department": "Catering",
    "emissions_kg": 233.24,
    "percentage_of_total": 8.1,
    "cost_gbp": 117.00
  }
]
```

---

### Query 5: Equipment Breakdown
```sql
SELECT 
    dc.equipment_name,
    mf.fuel_name,
    SUM(de.co2_emissions_kg) as emissions_kg
FROM defra_inventory di
JOIN defra_consumption dc ON di.id = dc.inventory_id
JOIN defra_emissions de ON dc.id = de.consumption_id
JOIN master_fuels mf ON di.fuel_name = mf.fuel_name
WHERE di.site_name = 'Manchester Office' 
    AND dc.consumption_date BETWEEN '2024-08-01' AND '2024-08-31'
    AND dc.equipment_name IS NOT NULL
GROUP BY dc.equipment_name, mf.fuel_name
ORDER BY emissions_kg DESC;
```

**Expected Output:**
```json
{
  "rows": [
    {
      "equipment_name": "Boiler System A",
      "fuel_name": "Natural Gas",
      "emissions_kg": "1424.08"
    },
    {
      "equipment_name": "HVAC System",
      "fuel_name": "Natural Gas",
      "emissions_kg": "610.32"
    },
    {
      "equipment_name": "Generator Unit",
      "fuel_name": "Diesel",
      "emissions_kg": "537.00"
    },
    {
      "equipment_name": "Kitchen Equipment",
      "fuel_name": "LPG",
      "emissions_kg": "233.24"
    },
    {
      "equipment_name": "Forklift Fleet",
      "fuel_name": "LPG",
      "emissions_kg": "77.75"
    }
  ]
}
```

**After JavaScript Processing:**
```json
[
  {
    "equipment_name": "Boiler System A",
    "fuel_used": "Natural Gas",
    "emissions_kg": 1424.08
  },
  {
    "equipment_name": "HVAC System",
    "fuel_used": "Natural Gas",
    "emissions_kg": 610.32
  },
  {
    "equipment_name": "Generator Unit",
    "fuel_used": "Diesel",
    "emissions_kg": 537.00
  },
  {
    "equipment_name": "Kitchen Equipment",
    "fuel_used": "LPG",
    "emissions_kg": 233.24
  },
  {
    "equipment_name": "Forklift Fleet",
    "fuel_used": "LPG",
    "emissions_kg": 77.75
  }
]
```

---

## 🎯 Final API Response Example

Here's what your complete API response should look like:

```json
{
  "success": true,
  "message": "UK carbon report generated successfully",
  "data": {
    "site_name": "Manchester Office",
    "site_postcode": "M1 1AA",
    "period": {
      "start_date": "2024-08-01",
      "end_date": "2024-08-31"
    },
    "summary": {
      "total_emissions_kg": 2882.39,
      "total_fuel_consumed": 1400.0,
      "total_activities": 5,
      "average_daily_emissions": 93.0,
      "total_cost_gbp": 966.0
    },
    "fuel_breakdown": [
      {
        "fuel_name": "Natural Gas",
        "fuel_type": "Gaseous fuels",
        "quantity_consumed": 1000.0,
        "unit": "Cubic metres (m3)",
        "defra_factor": 2.0344,
        "emissions_kg": 2034.40,
        "percentage_of_total": 70.6,
        "cost_gbp": 520.00
      },
      {
        "fuel_name": "Diesel",
        "fuel_type": "Liquid fuels",
        "quantity_consumed": 200.0,
        "unit": "Litres (L)",
        "defra_factor": 2.6850,
        "emissions_kg": 537.00,
        "percentage_of_total": 18.6,
        "cost_gbp": 290.00
      },
      {
        "fuel_name": "LPG",
        "fuel_type": "Gaseous fuels",
        "quantity_consumed": 200.0,
        "unit": "Litres (L)",
        "defra_factor": 1.5549,
        "emissions_kg": 310.98,
        "percentage_of_total": 10.8,
        "cost_gbp": 156.00
      }
    ],
    "department_breakdown": [
      {
        "department": "Office Heating",
        "emissions_kg": 2034.40,
        "percentage_of_total": 70.6,
        "cost_gbp": 520.00
      },
      {
        "department": "Production",
        "emissions_kg": 614.75,
        "percentage_of_total": 21.3,
        "cost_gbp": 329.00
      },
      {
        "department": "Catering",
        "emissions_kg": 233.24,
        "percentage_of_total": 8.1,
        "cost_gbp": 117.00
      }
    ],
    "equipment_breakdown": [
      {
        "equipment_name": "Boiler System A",
        "fuel_used": "Natural Gas",
        "emissions_kg": 1424.08
      },
      {
        "equipment_name": "HVAC System",
        "fuel_used": "Natural Gas",
        "emissions_kg": 610.32
      },
      {
        "equipment_name": "Generator Unit",
        "fuel_used": "Diesel",
        "emissions_kg": 537.00
      },
      {
        "equipment_name": "Kitchen Equipment",
        "fuel_used": "LPG",
        "emissions_kg": 233.24
      },
      {
        "equipment_name": "Forklift Fleet",
        "fuel_used": "LPG",
        "emissions_kg": 77.75
      }
    ]
  }
}
```

---

## 🔧 Testing Commands

Use these exact curl commands to test your API:

```bash
# Happy path test
curl -X GET "http://localhost:3000/api/defra/report?site=Manchester%20Office&start_date=2024-08-01&end_date=2024-08-31"

# Error test - missing parameters
curl -X GET "http://localhost:3000/api/defra/report?site=Manchester%20Office"

# Error test - invalid date range
curl -X GET "http://localhost:3000/api/defra/report?site=Manchester%20Office&start_date=2024-08-31&end_date=2024-08-01"

# Error test - non-existent site
curl -X GET "http://localhost:3000/api/defra/report?site=Unknown%20Site&start_date=2024-08-01&end_date=2024-08-31"
```

---

## ✅ Verification Checklist

Compare your outputs with these examples:

- [ ] Site verification returns correct site info
- [ ] Summary totals match calculated values
- [ ] Fuel breakdown is ordered by emissions (highest first)
- [ ] Percentages add up to 100% (allowing for rounding)
- [ ] Department breakdown shows all departments
- [ ] Equipment breakdown includes fuel_used field
- [ ] All decimal numbers are properly formatted
- [ ] Costs are calculated correctly
- [ ] Average daily emissions = total_emissions / days_in_period

Use these examples to debug and verify your implementation step by step! 🎯
