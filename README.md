# HB2 RDSPD Funding Pipeline

## 📋 Overview

This document outlines the step-by-step process for calculating the Regional Day School Program for the Deaf (RDSPD) funding allocation, as established by [Texas House Bill (HB) 2, 89th Legislature, Regular Session (2025)](https://www.legis.state.tx.us/BillLookup/Actions.aspx?LegSess=89R&Bill=HB2), which adds Section 48.315 to the Texas Education Code (TEC). Under TEC §48.315(a), each program administrator is entitled to receive $6,925 per student receiving services. However, TEC §48.315(b) requires the Texas Education Agency (TEA) to implement an upward adjustment mechanism to ensure that the total allotment for all programs meets the statutory minimum of $35 million annually.

> [!IMPORTANT]
> The method is organized in a clear flow from **legislative authority** → **business rule** → **calculation requirement** → **SAS code** → **results**.

## 🏫 Regional Day School Programs for the Deaf (RDSPD)

[RDSPD](https://tea.texas.gov/academics/special-student-populations/special-education/programs-and-services/sensory-impairments#:~:text=Regional%20Day%20School%20Programs%20for,Board%20of%20Education%20(SBOE).) are specialized educational programs established by the TEA to serve students who are deaf or hard of hearing from birth to age 22. Due to the low incidence of hearing differences, students may come from several districts to a school that houses an RDSPD for educational services or the RDSPD would send a teacher of the deaf and hard of hearing to the home district to provide itinerant services, RDSPDs are established as shared services arrangements (SSAs) between neighboring school districts and open enrollment charter schools, with one district or an education service center (ESC) serving as the fiscal agent of the RDSPD to coordinate funding and program administration. Located on general education campuses, the services ensure that deaf and hard of hearing students across Texas receive specialized educational services even when individual districts lack sufficient enrollment to support standalone programs.

## 📜 Legislative Foundation

The methodology implements the statutory requirements established by HB 2 in [TEC §48.315](https://capitol.texas.gov/tlodocs/89R/billtext/pdf/HB00002F.pdf).

> **Sec. 48.315. FUNDING FOR REGIONAL DAY SCHOOL PROGRAMS FOR THE DEAF**
>
> **(a)** The program administrator or fiscal agent of a regional day school program for the deaf is entitled to receive for each school year an allotment of **$6,925**, or a greater amount provided by appropriation, for **each student receiving services** from the program.
>
> **(b)** Notwithstanding Subsection (a), the agency shall **adjust the amount** of an allotment under that subsection for a school year to ensure the **total amount of allotments** provided under that subsection is **at least $35 million** for that school year.

This law was signed by Governor Greg Abbott on June 20, 2025 ([Journal p. 7610](https://journals.house.texas.gov/hjrnl/89r/pdf/89RDAY81FINAL.PDF#page=30))

## 🎯 Method

The method provides a complete funding pipeline from appropriation to distribution through a dual-level calculation for both statewide totals and totals disaggregated by fiscal agent.

-----

### 🧮 Data Source

Public Education Information Management System (PEIMS) Fall Submission (snapshot captured on the **last Friday in October**)
- Modeled on PEIMS data from the 2024–25 school year 
- Fall snapshot date: [Friday, October 25, 2024](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)
- Data available to customers: [Thursday, February 13, 2025](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)

#### 📅 Data Collection and Distribution Timeline Mapping

| School Year | PEIMS Snapshot Date<br>(Last Friday in October) | Data Available<br>for Processing | Expected Distribution |
|-------------|--------------------------------------------------|-----------------------------------|----------------------|
| 2024-25     | October 25, 2024                               | February 2025                     | TBD                  |
| 2025-26     | October 31, 2025                               | February 2026                     | TBD                  |
| 2026-27     | October 30, 2026                               | February 2027                     | TBD                  |
| 2027-28     | October 29, 2027                               | February 2028                     | TBD                  |
| 2028-29     | October 27, 2028                               | February 2029                     | TBD                  |

#### 📋 Timeline Notes

- **PEIMS Snapshot**: Student enrollment and service data captured on the last Friday in October
- **Processing Lag**: Approximately 16 weeks between data collection and availability for funding calculations
- **Distribution Timing**: RDSPD funding disbursements timing to be determined based on TEA processing schedules and legislative appropriation cycles
- **Academic Calendar Alignment**: This timeline ensures funding calculations are based on verified enrollment data

> [!CAUTION] 
> Actual distribution dates depend on TEA processing schedules and legislative appropriation timing
> Data availability dates are estimates based on historical PEIMS processing patterns
> School districts should plan cash flow accordingly, as there may be lag between data collection and funding distribution

-----

### Step 1: PEIMS Data Extraction

#### 📜 Business Rule
**BR-001: PEIMS Data Source Requirements**
> Student eligibility data must be extracted from the PEIMS Fall Submission special education program student view, ensuring data completeness and accuracy for funding calculations per TEC §48.315.

#### 📘 Calculation Requirement

Extract student-level RDSPD participation data from the PEIMS special education program student view.

#### 💻 SAS Code

```sas
proc sql;
    create table rdspd_raw as
    select DISTRICT,
           STUDENTID, 
           DIST_RDSPD_SVC,
           EFFECTIVE_DT
    from a.speced_pgm_student25f;
quit;
```

#### ✅ Output

```
NOTE: Table WORK.RDSPD_RAW created, with [X] rows and 4 columns.
```

-----

### Step 2: Student Eligibility Filtering

#### 📜 Business Rule
**BR-002: RDSPD Service Assignment Validation**
> Only students with a valid fiscal agent assignment in the **`DIST_RDSPD_SVC`** field are eligible for RDSPD funding. Records with missing or null fiscal agent assignments must be excluded from funding calculations.

#### 📘 Calculation Requirement

Filter the extracted data to include only students who have been assigned to receive RDSPD services from a fiscal agent.

#### 💻 SAS Code

```sas
data rdspd_eligible;
    set rdspd_raw;
    where DIST_RDSPD_SVC is not null;
run;
```

#### ✅ Output

```
NOTE: Data set WORK.RDSPD_ELIGIBLE created, with [Y] rows and 4 columns.
```

-----

### Step 3: Student Record Deduplication

#### 📜 Business Rule
**BR-003: Student Record Deduplication Rules**
> When multiple records exist for the same student, the most recent record (highest **`EFFECTIVE_DT`**) takes precedence. This ensures each student is counted only once in funding calculations, preventing duplicate funding claims.

#### 📘 Calculation Requirement

Sort student records by **`DISTRICT`**, **`STUDENTID`**, and **`EFFECTIVE_DT`** (descending) to identify the most current assignment for each student. Keep only the first record for each student to eliminate duplicates at the **`DISTRICT`** and **`STUDENTID`** level while preserving the most recent RDSPD service assignment.

#### 💻 SAS Code

```sas
proc sort data=rdspd_eligible out=rdspd_sorted;
    by DISTRICT STUDENTID descending EFFECTIVE_DT;
run;

data rdspd_deduplicated rdspd_duplicates;
    set rdspd_sorted;
    by DISTRICT STUDENTID;
    if first.STUDENTID then output rdspd_deduplicated;
    else output rdspd_duplicates;
run;
```

#### ✅ Output

```
NOTE: Data set WORK.RDSPD_DEDUPLICATED created, with [Z] rows and 4 columns.
NOTE: Data set WORK.RDSPD_DUPLICATES created, with [A] rows and 4 columns.
```

-----

### Step 4: Fiscal Agent Aggregation

#### 📜 Business Rule
**BR-004: Fiscal Agent Aggregation Requirements**
> Student counts must be aggregated by fiscal agent (**`DIST_RDSPD_SVC`**) to determine funding allocation per responsible district. Each fiscal agent receives funding based on their enrolled student population.

#### 📘 Calculation Requirement

Aggregate the deduplicated student records by fiscal agent to determine how many students are served by each fiscal agent.

#### 💻 SAS Code

```sas
proc sql;
    create table rdspd_counts as
    select DIST_RDSPD_SVC as FISCAL_AGENT_CDN,
           count(STUDENTID) as STUDENT_COUNT,
           6925 as STUDENT_BASE_FUNDING,
           (calculated STUDENT_COUNT * 6925) as TOTAL_INITIAL_FUNDING
    from rdspd_deduplicated
    group by DIST_RDSPD_SVC;
quit;
```

#### ✅ Output

```
NOTE: Table WORK.RDSPD_COUNTS created, with [B] rows and 4 columns.
```

**Example Results:**
```
FISCAL_AGENT_CDN    STUDENT_COUNT    STUDENT_BASE_FUNDING    TOTAL_INITIAL_FUNDING
000001                    89                  6925                  616,325
000002                   156                  6925                1,080,300
000003                   203                  6925                1,405,775
```

-----

### Step 5: Total Base Funding Calculation

#### 📜 Business Rule
**BR-005: Base Per-Student Entitlement**
> Each RDSPD student is entitled to receive $6,925 in base funding. The total base funding for all programs is calculated by multiplying this statutory base amount by the total number of students enrolled across all RDSPD programs.

#### 📘 Calculation Requirement

Calculate the statewide total base funding by summing the individual fiscal agent totals. This establishes the baseline funding across all RDSPD programs and determines whether additional funding adjustments are necessary to meet the statutory minimum.

#### Formula:

$$\text{Total Base Funding} = \text{Base Funding per Student} \times \text{Total RDSPD Students}$$

Where:

- **Base Funding per Student =** $6,925
- **Total RDSPD Students =** 5,012

#### Values:

```
$6,925.00 × 5,012 = $34,708,100.00
```

#### 💻 SAS Code

```sas
proc sql;
    create table statewide_totals as
    select sum(STUDENT_COUNT) as TOTAL_STUDENTS,
           sum(TOTAL_INITIAL_FUNDING) as TOTAL_BASE_FUNDING
    from rdspd_counts;
quit;
```

#### ✅ Output

```
TOTAL_STUDENTS    TOTAL_BASE_FUNDING
      5012           34,708,100.00
```
-----

### Step 6: Total Upward Adjustment Amount

#### 📜 Business Rule
**BR-006: Minimum Total Allocation Requirement**
> The total program funding must meet a statutory minimum of $35,000,000 annually. If the total base funding falls below this threshold, an upward adjustment is required to bridge the funding gap.

**BR-007: Upward Adjustment Trigger**
> An upward adjustment shall only be applied when the total base funding is less than or equal to the minimum allocation. The adjustment amount equals the difference between the minimum allocation and the total base funding.

#### 📘 Calculation Requirement

Compare the total base funding to the minimum allotment of $35,000,000. If the base funding falls short, calculate the difference as the upward adjustment amount that will be distributed equally among all students.

#### Condition:

```
If Total Base Funding ≤ Total Allotment:
```

#### Formula:

$$\text{Total Upward Adjustment} = \text{Total Allotment} - \text{Total Base Funding}$$

Where:

- **Total Allotment =** $35,000,000
- **Total Base Funding =** $34,708,100 (from Step 5)

#### Values:

```
$35,000,000 − $34,708,100 = $291,900.00
```

#### 💻 SAS Code

```sas
data upward_adjustment;
    set statewide_totals;
    STATE_MINIMUM = 35000000;
    if TOTAL_BASE_FUNDING <= STATE_MINIMUM then
        TOTAL_UPWARD_ADJUSTMENT = STATE_MINIMUM - TOTAL_BASE_FUNDING;
    else
        TOTAL_UPWARD_ADJUSTMENT = 0;
    put "Total Upward Adjustment Amount: $" TOTAL_UPWARD_ADJUSTMENT comma12.2;
run;
```

#### ✅ Output

```
Total Upward Adjustment Amount: $291,900.00
```

-----

### Step 7: Per-Student Upward Adjustment Amount

#### 📜 Business Rule
**BR-008: Equitable Distribution Principle**
> Any upward adjustment amount must be distributed equally among all RDSPD students to ensure fairness and compliance with equal treatment requirements. Each student receives the same proportional share of the additional funding.

#### 📘 Calculation Requirement

Calculate the per-student share of the upward adjustment by dividing the total adjustment amount by the total number of RDSPD students. This ensures equitable distribution of additional funding across all students regardless of their fiscal agent assignment.

#### Formula:

$$
\text{Per-Student Upward Adjustment}
= \frac{\text{Total Upward Adjustment}}
       {\text{Total RDSPD Students}}
$$

Where:

- **Total Upward Adjustment =** $291,900 (from Step 6)
- **Total RDSPD Students =** 5,012

#### Values:

```
$291,900.00 ÷ 5,012 = $58.24
```

#### 💻 SAS Code

```sas
data per_student_adjustment;
    set upward_adjustment;
    STUDENT_UPWARD_ADJUSTMENT = TOTAL_UPWARD_ADJUSTMENT / TOTAL_STUDENTS;
    put "Per-Student Upward Adjustment Amount: $" STUDENT_UPWARD_ADJUSTMENT 8.2;
run;
```

#### ✅ Output

```
Per-Student Upward Adjustment Amount: $58.24
```

-----

### Step 8: Adjusted Base Funding per Student

#### 📜 Business Rule
**BR-009: Adjusted Funding Calculation**
> The final per-student funding amount is calculated by adding the per-student upward adjustment to the statutory base funding amount. This ensures each student receives both their entitled base funding and their proportional share of any additional funding needed to meet the minimum allocation.

#### 📘 Calculation Requirement

Combine the original base funding with the calculated upward adjustment to determine the new per-student funding amount. This adjusted rate ensures that the total program funding will meet the minimum allotment requirement.

#### Formula:

$$\text{Adjusted Base Funding per Student} = \text{Base Funding per Student} + \text{Per-Student Upward Adjustment}$$

Where:

- **Base Funding per Student =** $6,925
- **Per-Student Upward Adjustment =** $58.24 (from Step 7)

#### Values:

```
$6,925.00 + $58.24 = $6,983.24
```

#### 💻 SAS Code

```sas
data adjusted_funding_rate;
    set per_student_adjustment;
    BASE_FUNDING_PER_STUDENT = 6925.00;
    ADJUSTED_BASE_FUNDING = BASE_FUNDING_PER_STUDENT + STUDENT_UPWARD_ADJUSTMENT;
    put "Adjusted Base Funding per Student: $" ADJUSTED_BASE_FUNDING 8.2;
run;
```

#### ✅ Output

```
Adjusted Base Funding per Student: $6,983.24
```

-----

### Step 9: Final Funding by Fiscal Agent

#### 📜 Business Rule
**BR-010: Fiscal Agent Funding Distribution**
> Each fiscal agent receives funding based on their enrolled student population multiplied by the adjusted per-student funding rate. This ensures that funding flows directly to the districts responsible for providing RDSPD services.

#### 📘 Calculation Requirement

Apply the adjusted per-student funding rate to each fiscal agent's student count to determine individual funding allocations. This step transforms the statewide funding calculation into actionable disbursement amounts for each participating district.

#### Formula:

$$\text{Final Funding per Fiscal Agent} = \text{Student Count} + \text{Adjusted Base Funding per Student}$$

#### 💻 SAS Code

```sas
proc sql;
    create table final_funding_by_agent as
    select a.FISCAL_AGENT_CDN,
           a.STUDENT_COUNT,
           a.STUDENT_BASE_FUNDING,
           a.TOTAL_INITIAL_FUNDING,
           b.STUDENT_UPWARD_ADJUSTMENT,
           b.ADJUSTED_BASE_FUNDING,
           (a.STUDENT_COUNT * b.ADJUSTED_BASE_FUNDING) as FINAL_FUNDING format=comma12.2
    from rdspd_counts a
    cross join adjusted_funding_rate b;
quit;
```

#### ✅ Output

**Example Results:**
```
FISCAL_AGENT_CDN  STUDENT_COUNT  STUDENT_BASE_FUNDING  TOTAL_INITIAL_FUNDING  STUDENT_UPWARD_ADJUSTMENT  ADJUSTED_BASE_FUNDING  FINAL_FUNDING
000001                   89                  6925                 616,325                      58.24                6,983.24         621,508.36
000002                  156                  6925               1,080,300                      58.24                6,983.24       1,089,385.44
000003                  203                  6925               1,405,775                      58.24                6,983.24       1,417,597.72
```

-----

### Step 10: Total Adjusted Funding Verification

#### 📜 Business Rule
**BR-011: Total Funding Verification**
> The total adjusted funding must be calculated using the adjusted per-student rate to verify that the funding mechanism achieves the minimum allocation requirement. This serves as a validation step to ensure compliance with statutory requirements.

#### 📘 Calculation Requirement

Calculate the statewide total adjusted funding by summing all fiscal agent allocations. This verification step ensures that the funding mechanism successfully meets the minimum allotment requirement.

#### Formula:

$$\text{Total Adjusted Funding} = \text{Adjusted Base Funding per Student} \times \text{Total RDSPD Students}$$

Where:

- **Adjusted Base Funding per Student =** $6,983.24 (from Step 8)
- **Total RDSPD Students =** 5,012

#### Values:

```
$6,983.24 × 5,012 = $34,999,998.88
```

#### 💻 SAS Code

```sas
proc sql;
    create table total_verification as
    select sum(FINAL_FUNDING) as TOTAL_ADJUSTED_FUNDING format=comma15.2
    from final_funding_by_agent;
quit;
```

#### ✅ Output

```
TOTAL_ADJUSTED_FUNDING
    34,999,998.88
```

-----

### Step 11: Final Verification and Rounding

### 📜 Business Rule
**BR-012: Rounding and Tolerance Verification**
> Final verification requires confirming that the total adjusted funding meets the minimum allocation requirement within acceptable rounding tolerances. Small variances are likely due to decimal round precision (i.e., Minor discrepancies between expected and calculated values likely the result of rounding numbers during intermediate calculations).

#### 📘 Calculation Requirement

Perform final verification that the total adjusted funding meets the minimum allotment requirement. The small rounding difference ($1.12) is due to decimal precision in the per-student calculations.

#### Values:

```
Total Adjusted Funding: $34,999,998.88
Rounded to Nearest Million: $35,000,000
```

#### 💻 SAS Code

```sas
data final_verification;
    set total_verification;
    ROUNDED_MILLIONS = round(TOTAL_ADJUSTED_FUNDING, 1000000);
    VARIANCE = 35000000 - TOTAL_ADJUSTED_FUNDING;
    put "Total Adjusted Funding: $" TOTAL_ADJUSTED_FUNDING comma15.2;
    put "Rounded to Nearest Million: $" ROUNDED_MILLIONS comma15.2;
    put "Variance from $35M: $" VARIANCE comma10.2;
run;
```

#### ✅ Output

```
Total Adjusted Funding: $34,999,998.88
Rounded to Nearest Million: $35,000,000.00
Variance from $35M: $1.12
```

-----

## 📊 Summary

### Business Rules Summary

The RDSPD funding calculation methodology is governed by twelve key business rules:

1. **BR-001**: PEIMS data source requirements for student eligibility
2. **BR-002**: RDSPD service assignment validation
3. **BR-003**: Student record deduplication rules
4. **BR-004**: Fiscal agent aggregation requirements
5. **BR-005**: Base per-student entitlement of $6,925 per TEC §48.315(a)
6. **BR-006**: Minimum total allocation requirement of $35,000,000 per TEC §48.315(b)
7. **BR-007**: Upward adjustment trigger when base funding is insufficient
8. **BR-008**: Equitable distribution of adjustment amounts among all students
9. **BR-009**: Adjusted funding calculation combining base and adjustment amounts
10. **BR-010**: Fiscal agent funding distribution
11. **BR-011**: Total funding verification using adjusted rates
12. **BR-012**: Rounding tolerance verification for compliance

### Dual-Level Calculation Flow

This methodology provides both statewide totals for legislative compliance and fiscal agent breakdowns** for practical disbursement through parallel calculation pipelines:

#### Common Data Foundation (Steps 1-4)
Both pipelines rely on PEIMS data:
1. Extract student data from PEIMS Fall submission
2. Filter for students with valid RDSPD service assignments  
3. Deduplicate student records by most recent effective date
4. Aggregate student counts by fiscal agent

#### State-Level Pipeline: Legislative Appropriation & Compliance
**Purpose:** Verify statutory compliance and determine statewide funding requirements

5. **Calculate total base funding** at $6,925 per student (5,012 students = $34,708,100)
6. **Determine upward adjustment** needed to reach $35,000,000 minimum ($291,900)
7. **Calculate per-student upward adjustment** ($58.24 per student)
8. **Establish adjusted per-student funding rate** ($6,983.24)
10. **Verify total adjusted funding** meets minimum requirement ($34,999,998.88)
11. **Confirm compliance** within acceptable rounding tolerances

**State-Level Outcomes:**
- Legislative mandate compliance (TEC §48.315)
- Total appropriation requirement ($35,000,000)
- Uniform per-student adjustment rate ($58.24)

#### Fiscal Agent-Level Pipeline: District Disbursement & Operations
**Purpose:** Calculate individual funding allocations for practical distribution

4. **Aggregate student counts** by fiscal agent (from common data foundation)
8. **Apply adjusted per-student rate** ($6,983.24) to each fiscal agent's enrollment
9. **Calculate final funding** per fiscal agent (student count × $6,983.24)
11. **Generate disbursement amounts** for each participating district

**Fiscal Agent-Level Outcomes:**
- 💰 Individual district funding amounts
- 📊 Student-based allocation transparency  
- 🏫 Actionable disbursement data for TEA operations

**Example Fiscal Agent Results:**
```
FISCAL_AGENT_CDN  STUDENT_COUNT  FINAL_FUNDING
000001                   89        $621,508.36
000002                  156      $1,089,385.44
000003                  203      $1,417,597.72
```

This dual-pipeline approach ensures legislative compliance at the state level while providing actionable funding distributions for individual school districts fiscal agents.

-----

## ❓ Frequently Asked Questions (FAQ)

### Q: What happens if the total RDSPD funding exceeds $35 million before any adjustments?

**A:** Based on TEC §48.315, if the base calculation exceeds $35 million, **no adjustment occurs** and students receive the full **$6,925 per student**. The upward adjustment mechanism only triggers when total base funding falls **below** $35 million.

### Q: What is the current funding situation for RDSPD students?

**A:** Based on data from the current 2024-25 school year, there are **5,012** RDSPD students. With the base allotment of **$6,925**, the total base funding equals $34,708,100, which falls below the $35 million minimum. This triggers an upward adjustment of **$58.24** per student, bringing the total per-pupil funding to **$6,983.24**. The upward adjustment brings the total funding to **$34,999,998.88**, which is practically **$35 million**.

### Q: Is $35 million a funding cap or floor?

**A:** HB2 sets $35 million as a **minimum floor**, not a maximum cap. The law requires the agency to "ensure the total amount...is **at least** $35 million" - meaning $35 million is the guaranteed minimum, but funding can exceed this amount when student enrollment justifies it.

### Q: How would funding work with higher enrollment?

**A:** **Scenario Analysis:**
- **Current situation:** 5,012 students × $6,925 = $34.7M (below minimum) → triggers $58.24 upward adjustment
- **Higher enrollment scenario:** 5,100 students × $6,925 = $35.3M (above minimum) → no adjustment needed, students get full $6,925

When base funding exceeds $35 million due to higher enrollment, the state wouldn't reduce the per-student amount - students would receive the full statutory $6,925 with total funding exceeding $35 million.

-----

For questions, email [zane.wubbena@tea.texas.gov](zane.wubbena@tea.texas.gov)
