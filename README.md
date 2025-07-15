# hb2-rdspd-funding

# Step 1: Total Base Funding Calculation
## 📘 Calculation Requirement
### Formula:
**Total Base Funding** = **Base Funding per Student** × **Total RDSPD Students**  

### Values:
$6,925.00 × 5,012 = $34,708,100.00

## 💻 SAS Code
```
data base_funding;
    base_funding_per_student = 6925.00;
    total_rdspd_students = 5012;
    total_base_funding = base_funding_per_student * total_rdspd_students;
    put "Total Base Funding: $" total_base_funding comma12.2;
run;
```
## ✅ Output
```
Total Base Funding: $34,708,100.00
```

# Step 2: Total Upward Adjustment Amount
Calculate the upward adjustment amount if the total base funding ≤ $35,000,000.

## 📘 Calculation Requirement
### Condition:

If Total Base Funding ≤ Total Allotment:

### Formula:

**Total Upward Adjustment = Total Allotment − Total Base Funding**

### Values:
Total Upward Adjustment = $35,000,000 − $34,708,100 = $291,900.00

## 💻 SAS Code
```
data upward_adjustment;
    total_allotment = 35000000;
    total_base_funding = 6925.00 * 5012;
    if total_base_funding <= total_allotment then
        total_upward_adjustment = total_allotment - total_base_funding;
    else
        total_upward_adjustment = 0;
    put "Total Upward Adjustment Amount: $" total_upward_adjustment comma12.2;
run;
```
### ✅ Output
```
Total Upward Adjustment Amount: $291,900.00
```

# Step 3: Per-Student Upward Adjustment Amount
## 📘 Calculation Requirement
### Formula:
**Per-Student Upward Adjustment = Total Upward Adjustment ÷ Total RDSPD Students**

### Values:
$291,900.00 ÷ 5,012 = $58.24

### 💻 SAS Code
```
data per_student_adjustment;
    upward_adjustment_amount = 291900.00;
    total_students = 5012;
    per_student_amount = upward_adjustment_amount / total_students;
    put "Per-Student Upward Adjustment Amount: $" per_student_amount 8.2;
run;`
```
### ✅ Output
```
Per-Student Upward Adjustment Amount: $58.24
```

# Step 4: Add Per-Student Upward Adjustment to Base Funding

## 📘 Calculation Requirement
### Formula:
**Adjusted Base Funding per Student = Base Funding per Student + Per-Student Upward Adjustment**
### Values:
$6,925.00 + $58.24 = $6,983.24

## 💻 SAS Code
```
data adjusted_base_funding;
    base_funding = 6925.00;
    upward_adjustment = 58.24;
    adjusted_funding = base_funding + upward_adjustment;
    put "Adjusted Base Funding per Student: $" adjusted_funding 8.2;
run;
```
## ✅ Output
```
Adjusted Base Funding per Student: $6983.24
```

# Step 5: Total Adjusted Funding 
Multiply the base funding by the total RDSPD students.

## 📘 Calculation Requirement
### Formula:
**Total Adjusted Funding = Adjusted Base Funding per Student × Total RDSPD Students**
### Values:
Total Adjusted Funding = $6,983.24 × 5,012 = $34,999,998.88

## 💻 SAS Code
```
data total_adjusted_funding;
    adjusted_funding_per_student = 6983.24;
    total_students = 5012;
    total_adjusted_funding = adjusted_funding_per_student * total_students;
    put "Total Adjusted Funding: $" total_adjusted_funding comma15.2;
run;
```
## ✅ Output
```
Total Adjusted Funding: $34,999,998.88
```

# Step 6: Rounded Total Base Funding
Round the total base funding to the nearest million.

## 📘 Calculation Requirement

### Rounded to Nearest Million:
Total Adjusted Base Funding: $34,999,998.88

Rounded Total Base Funding = $35,000,000

## 💻 SAS Code
```
data rounded_base_funding;
    base_funding = 6925.00;
    total_students = 5012;
    total_base_funding = base_funding * total_students;
    rounded_millions = round(total_base_funding, 1000000);
    put "Total Base Funding: $" total_base_funding comma15.2;
    put "Rounded to Nearest Million: $" rounded_millions comma15.2;
run;
```
## ✅ Output
```
Total Base Funding: $34,999,998.88
Rounded to Nearest Million: $35,000,000.00
```

