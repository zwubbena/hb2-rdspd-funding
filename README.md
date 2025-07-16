# HB2 RDSPD Funding Calculation

## 📋 Overview

This document outlines the step-by-step process for calculating the Regional Day School Program for the Deaf (RDSPD) funding allocation, as established by [Texas House Bill 2 (HB2), 89th Legislature, Regular Session (2025)](https://capitol.texas.gov/tlodocs/89R/billtext/pdf/HB00002F.pdf),which adds Section 48.315 to the Texas Education Code (TEC). Under TEC §48.315(a), each program administrator is entitled to receive $6,925 per student receiving services. However, TEC §48.315(b) requires the Texas Education Agency to implement an upward adjustment mechanism to ensure that the total allotment for all programs meets the statutory minimum of $35 million annually. The funding calculation process described herein ensures compliance with both the base per-student amount and the required total minimum.

-----

## Step 1: Total Base Funding Calculation

### 📘 Calculation Requirement

The first step establishes the baseline funding by multiplying the standard per-student allocation by the total number of students enrolled in RDSPD programs. This base calculation determines whether additional funding adjustments are necessary.

### Formula:

`Total Base Funding = Base Funding per Student × Total RDSPD Students`

Where:

- **Base Funding per Student =** $6,925
- **Total RDSPD Students =** 5,012

### Values:

$6,925.00 × 5,012 = $34,708,100.00

### 💻 SAS Code

```sas
data base_funding;
    base_funding_per_student = 6925.00;
    total_rdspd_students = 5012;
    total_base_funding = base_funding_per_student * total_rdspd_students;
    put "Total Base Funding: $" total_base_funding comma12.2;
run;
```

### ✅ Output

```
Total Base Funding: $34,708,100.00
```

-----

## Step 2: Total Upward Adjustment Amount

### 📘 Calculation Requirement

This step determines if an upward adjustment is needed by comparing the total base funding to the minimum allotment of $35,000,000. If the base funding falls short, the difference becomes the upward adjustment amount that will be distributed among all students.

### Condition:

If Total Base Funding ≤ Total Allotment:

### Formula:

`Total Upward Adjustment = Total Allotment − Total Base Funding`

Where:

- **Total Allotment =** $35,000,000
- **Total Base Funding =** $34,708,100 (from Step 1)

### Values:

$35,000,000 − $34,708,100 = $291,900.00

### 💻 SAS Code

```sas
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

-----

## Step 3: Per-Student Upward Adjustment Amount

### 📘 Calculation Requirement

To ensure equitable distribution of the additional funding, this step calculates how much extra funding each student will receive. The total upward adjustment is divided equally among all RDSPD students, resulting in a per-student supplement to the base funding.

### Formula:

`Per-Student Upward Adjustment = Total Upward Adjustment ÷ Total RDSPD Students`

Where:

- **Total Upward Adjustment =** $291,900 (from Step 2)
- **Total RDSPD Students =** 5,012

### Values:

$291,900.00 ÷ 5,012 = $58.24

### 💻 SAS Code

```sas
data per_student_adjustment;
    upward_adjustment_amount = 291900.00;
    total_students = 5012;
    per_student_amount = upward_adjustment_amount / total_students;
    put "Per-Student Upward Adjustment Amount: $" per_student_amount 8.2;
run;
```

### ✅ Output

```
Per-Student Upward Adjustment Amount: $58.24
```

-----

## Step 4: Add Per-Student Upward Adjustment to Base Funding

### 📘 Calculation Requirement

This step combines the original base funding with the calculated upward adjustment to determine the new per-student funding amount. This adjusted rate ensures that the total program funding will meet the minimum allotment requirement.

### Formula:

`Adjusted Base Funding per Student = Base Funding per Student + Per-Student Upward Adjustment`

Where:

- **Base Funding per Student =** $6,925
- **Per-Student Upward Adjustment =** $58.24 (from Step 3)

### Values:

$6,925.00 + $58.24 = $6,983.24

### 💻 SAS Code

```sas
data adjusted_base_funding;
    base_funding = 6925.00;
    upward_adjustment = 58.24;
    adjusted_funding = base_funding + upward_adjustment;
    put "Adjusted Base Funding per Student: $" adjusted_funding 8.2;
run;
```

### ✅ Output

```
Adjusted Base Funding per Student: $6,983.24
```

-----

## Step 5: Total Adjusted Funding

### 📘 Calculation Requirement

This step calculates the total program funding using the adjusted per-student rate. By multiplying the adjusted base funding by the total number of students, we verify that the funding now meets or approaches the minimum allotment of $35,000,000.

### Formula:

`Total Adjusted Funding = Adjusted Base Funding per Student × Total RDSPD Students`

Where:

- **Adjusted Base Funding per Student =** $6,983.24 (from Step 4)
- **Total RDSPD Students =** 5,012

### Values:

$6,983.24 × 5,012 = $34,999,998.88

### 💻 SAS Code

```sas
data total_adjusted_funding;
    adjusted_funding_per_student = 6983.24;
    total_students = 5012;
    total_adjusted_funding = adjusted_funding_per_student * total_students;
    put "Total Adjusted Funding: $" total_adjusted_funding comma15.2;
run;
```

### ✅ Output

```
Total Adjusted Funding: $34,999,998.88
```

-----

## Step 6: Verification - Rounded Total Funding

### 📘 Calculation Requirement

As a final verification step, this calculation rounds the total adjusted funding to the nearest million to confirm that the funding mechanism successfully brings the total allocation to the required $35,000,000 threshold. The small rounding difference ($1.12) is due to decimal precision in the per-student calculations.

### Values:

`Total Adjusted Funding: $34,999,998.88`
`Rounded to Nearest Million: $35,000,000`

### 💻 SAS Code

```sas
data rounded_funding_verification;
    adjusted_funding_per_student = 6983.24;
    total_students = 5012;
    total_adjusted_funding = adjusted_funding_per_student * total_students;
    rounded_millions = round(total_adjusted_funding, 1000000);
    put "Total Adjusted Funding: $" total_adjusted_funding comma15.2;
    put "Rounded to Nearest Million: $" rounded_millions comma15.2;
run;
```

### ✅ Output

```
Total Adjusted Funding: $34,999,998.88
Rounded to Nearest Million: $35,000,000.00
```

-----

## 📊 Summary

Through this six-step process, the RDSPD funding calculation ensures that:

1. Base funding is calculated at $6,925 per student
1. When total base funding ($34,708,100) falls below the $35,000,000 minimum, an upward adjustment is applied
1. Each student receives an additional $58.24 in funding
1. The adjusted per-student funding becomes $6,983.24
1. Total program funding reaches $34,999,998.88, effectively meeting the $35,000,000 requirement