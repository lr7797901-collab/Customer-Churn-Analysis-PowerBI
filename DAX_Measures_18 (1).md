# 📊 Customer Churn Analysis — DAX Measures (18)

Table: `customer_churn_1M`

---

## 🧮 Core Customer Counts

**1. Total Customers**
```dax
Total Customers = DISTINCTCOUNT(customer_churn_1M[customerid])
```

**2. Active Customers**
```dax
Active Customers = CALCULATE([Total Customers], customer_churn_1M[churn] = "No")
```

**3. Churn Customers**
```dax
Churn Customers = CALCULATE([Total Customers], customer_churn_1M[churn] = "Yes")
```

---

## 📉 Churn & Retention Rates

**4. Churn Rate**
```dax
Churn Rate = DIVIDE([Churn Customers], [Total Customers], 0)
```

**5. Retention Rate**
```dax
Retention Rate = DIVIDE([Active Customers], [Total Customers], 0)
```

---

## 💰 Revenue Metrics

**6. Total Revenue**
```dax
Total Revenue = SUM(customer_churn_1M[totalcharges])
```

**7. Revenue Lost**
```dax
Revenue Lost = CALCULATE([Total Revenue], customer_churn_1M[churn] = "Yes")
```

**8. ARPU (Average Revenue Per User)**
```dax
ARPU = DIVIDE([Total Revenue], [Total Customers], 0)
```

**9. Average Monthly Charges**
```dax
Average Monthly Charges = AVERAGE(customer_churn_1M[monthlycharges])
```

**10. Average Total Charges**
```dax
Average Total Charges = AVERAGE(customer_churn_1M[totalcharges])
```

---

## 👤 Customer Profile Averages

**11. Average Age**
```dax
Average Age = AVERAGE(customer_churn_1M[age])
```

**12. Average Customer Satisfaction**
```dax
Average Customer Satisfaction = AVERAGE(customer_churn_1M[satisfaction])
```

---

## ⚠️ Risk Segmentation

**13. High Risk Customers**
```dax
High Risk Customers = 
CALCULATE(
    [Total Customers],
    customer_churn_1M[churn] = "Yes",
    customer_churn_1M[late_payments] > 2
)
```

---

## 🏠 Demographic & Household %

**14. Partner %**
```dax
Partner % = 
DIVIDE(
    CALCULATE([Total Customers], customer_churn_1M[partner] = "Yes"),
    [Total Customers],
    0
)
```

**15. Dependents %**
```dax
Dependents % = 
DIVIDE(
    CALCULATE([Total Customers], customer_churn_1M[dependents] = "Yes"),
    [Total Customers],
    0
)
```

**16. Senior Citizen %**
```dax
Senior Citizen % = 
DIVIDE(
    CALCULATE([Total Customers], customer_churn_1M[senior_citizen] = 1),
    [Total Customers],
    0
)
```

**17. Family Customer %**
```dax
Family Customer % = 
DIVIDE(
    CALCULATE([Total Customers], customer_churn_1M[marital_status] = "Married"),
    [Total Customers],
    0
)
```

**18. Male / Female Split**
```dax
Male / Female Split = 
DIVIDE(
    CALCULATE([Total Customers], customer_churn_1M[gender] = "Male"),
    [Total Customers],
    0
)
```

