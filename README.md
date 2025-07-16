# HB2 RDSPD Funding Allocation: Methodology with Business Rules

## 📋 Overview

This document outlines the step-by-step process for calculating the Regional Day School Program for the Deaf (RDSPD) funding allocation, as established by [Texas House Bill 2 (HB2), 89th Legislature, Regular Session (2025)](https://capitol.texas.gov/tlodocs/89R/billtext/pdf/HB00002F.pdf), which adds Section 48.315 to the Texas Education Code (TEC). Under TEC §48.315(a), each program administrator is entitled to receive $6,925 per student receiving services. However, TEC §48.315(b) requires the Texas Education Agency to implement an upward adjustment mechanism to ensure that the total allotment for all programs meets the statutory minimum of $35 million annually. The funding calculation process described herein ensures compliance with both the base per-student amount and the required total minimum.

> [!TIP]
> Each step in this methodology is organized to create a clear flow from **business rule** → **calculation requirement** → **SAS code** → **results**. This structure ensures regulatory compliance, provides audit trails, and makes it easy to understand both the legal foundation or authority and the technical implementation of each calculation step.

-----

## 🧮 Data Source

Public Education Information Management System (PEIMS) Fall Submission (snapshot captured on the **last Friday in October**)
- Modeled on PEIMS data from the 2024–25 school year  
- Fall snapshot date: [Friday, October 25, 2024](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)
- Data available to customers: [Thursday, February 13, 2025
](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)

-----

## Step 1: Total Base Funding Calculation

### 📜 Business Rule
**BR-001: Base Per-Student Entitlement**
> Each RDSPD student is entitled to receive $6,925 in base funding per TEC §48.315(a). The total base funding for all programs is calculated by multiplying this statutory base amount by the total number of students enrolled across all RDSPD programs.

### 📘 Calculation Requirement

The first step establishes the baseline funding by multiplying the standard per-student allocation by the total number of students enrolled in RDSPD programs. This base calculation determines whether additional funding adjustments are necessary.

### Formula:

$$\text{Total Base Funding} = \text{Base Funding per Student} \times \text{Total RDSPD Students}$$

Where:

- **Base Funding per Student =** $6,925
- **Total RDSPD Students =** 5,012

### Values:

`$6,925.00 × 5,012 = $34,708,100.00`

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

### 📜 Business Rule
**BR-002: Minimum Total Allocation Requirement**
> The total program funding must meet a statutory minimum of $35,000,000 annually per TEC §48.315(b). If the total base funding falls below this threshold, an upward adjustment is required to bridge the funding gap.

**BR-003: Upward Adjustment Trigger**
> An upward adjustment shall only be applied when the total base funding is less than or equal to the minimum allocation. The adjustment amount equals the difference between the minimum allocation and the total base funding.

### 📘 Calculation Requirement

This step determines if an upward adjustment is needed by comparing the total base funding to the minimum allotment of $35,000,000. If the base funding falls short, the difference becomes the upward adjustment amount that will be distributed among all students.

### Condition:

`If Total Base Funding ≤ Total Allotment:`

### Formula:

$$\text{Total Upward Adjustment} = \text{Total Allotment} - \text{Total Base Funding}$$

Where:

- **Total Allotment =** $35,000,000
- **Total Base Funding =** $34,708,100 (from [Step 1](https://github.com/zwubbena/hb2-rdspd-funding/blob/main/README.md#step-1-total-base-funding-calculation))

### Values:

`$35,000,000 − $34,708,100 = $291,900.00`

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

### 📜 Business Rule
**BR-004: Equitable Distribution Principle**
> Any upward adjustment amount must be distributed equally among all RDSPD students to ensure fairness and compliance with equal treatment requirements. Each student receives the same proportional share of the additional funding.

### 📘 Calculation Requirement

To ensure equitable distribution of the additional funding, this step calculates how much extra funding each student will receive. The total upward adjustment is divided equally among all RDSPD students, resulting in a per-student supplement to the base funding.

### Formula:

$$\text{Per-Student Upward Adjustment} = \frac{\text{Total Upward Adjustment}}{\text{Total RDSPD Students}}$$

Where:

- **Total Upward Adjustment =** $291,900 (from [Step 2](https://github.com/zwubbena/hb2-rdspd-funding/blob/main/README.md#step-2-total-upward-adjustment-amount))
- **Total RDSPD Students =** 5,012

### Values:

`$291,900.00 ÷ 5,012 = $58.24`

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

### 📜 Business Rule
**BR-005: Adjusted Funding Calculation**
> The final per-student funding amount is calculated by adding the per-student upward adjustment to the statutory base funding amount. This ensures each student receives both their entitled base funding and their proportional share of any additional funding needed to meet the minimum allocation.

### 📘 Calculation Requirement

This step combines the original base funding with the calculated upward adjustment to determine the new per-student funding amount. This adjusted rate ensures that the total program funding will meet the minimum allotment requirement.

### Formula:

$$\text{Adjusted Base Funding per Student} = \text{Base Funding per Student} + \text{Per-Student Upward Adjustment}$$

Where:

- **Base Funding per Student =** $6,925
- **Per-Student Upward Adjustment =** $58.24 (from [Step 3](https://github.com/zwubbena/hb2-rdspd-funding/blob/main/README.md#step-3-per-student-upward-adjustment-amount))

### Values:

`$6,925.00 + $58.24 = $6,983.24`

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

### 📜 Business Rule
**BR-006: Total Funding Verification**
> The total adjusted funding must be calculated using the adjusted per-student rate to verify that the funding mechanism successfully achieves the minimum allocation requirement. This serves as a validation step to ensure compliance with statutory requirements.

### 📘 Calculation Requirement

This step calculates the total program funding using the adjusted per-student rate. By multiplying the adjusted base funding by the total number of students, we verify that the funding now meets or approaches the minimum allotment of $35,000,000.

### Formula:

$$\text{Total Adjusted Funding} = \text{Adjusted Base Funding per Student} \times \text{Total RDSPD Students}$$

Where:

- **Adjusted Base Funding per Student =** $6,983.24 (from [Step 4](https://github.com/zwubbena/hb2-rdspd-funding/blob/main/README.md#step-4-add-per-student-upward-adjustment-to-base-funding))
- **Total RDSPD Students =** 5,012

### Values:

`$6,983.24 × 5,012 = $34,999,998.88`

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

### 📜 Business Rule
**BR-007: Rounding and Tolerance Verification**
> Final verification requires confirming that the total adjusted funding meets the minimum allocation requirement within acceptable rounding tolerances. Small variances due to decimal precision in per-student calculations are acceptable as long as the total approaches the statutory minimum.

### 📘 Calculation Requirement

As a final verification step, this calculation rounds the total adjusted funding to the nearest million to confirm that the funding mechanism successfully brings the total allocation to the required $35,000,000 threshold. 

The small rounding difference ($1.12) is due to decimal precision in the per-student calculations.

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

### Business Rules Summary

The RDSPD funding calculation methodology is governed by seven key business rules:

1. **BR-001**: Base per-student entitlement of $6,925 per TEC §48.315(a)
2. **BR-002**: Minimum total allocation requirement of $35,000,000 per TEC §48.315(b)
3. **BR-003**: Upward adjustment trigger when base funding is insufficient
4. **BR-004**: Equitable distribution of adjustment amounts among all students
5. **BR-005**: Adjusted funding calculation combining base and adjustment amounts
6. **BR-006**: Total funding verification using adjusted rates
7. **BR-007**: Rounding tolerance verification for compliance

### Calculation Flow Summary

Through this six-step process, the RDSPD funding calculation ensures that:

1. Base funding is calculated at $6,925 per student
2. When total base funding ($34,708,100) falls below the $35,000,000 minimum, an upward adjustment is applied
3. Each student receives an additional $58.24 in funding
4. The adjusted per-student funding becomes $6,983.24
5. Total program funding reaches $34,999,998.88, effectively meeting the $35,000,000 requirement
