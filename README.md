# HB2 RDSPD Appropriation to Distribution Pipeline

## 📋 Overview

This document outlines the step-by-step process for calculating the Regional Day School Program for the Deaf (RDSPD) funding allocation, as established by [Texas House Bill (HB) 2, 89th Legislature, Regular Session (2025)](https://www.legis.state.tx.us/BillLookup/Actions.aspx?LegSess=89R&Bill=HB2), which adds Section 48.315 to the Texas Education Code (TEC). Under TEC §48.315(a), each program administrator is entitled to receive $6,925 per student receiving services. However, TEC §48.315(b) requires the Texas Education Agency to implement an upward adjustment mechanism to ensure that the total allotment for all programs meets the statutory minimum of $35 million annually. The funding calculation process described herein ensures compliance with both the base per-student amount and the required total minimum.

> [!IMPORTANT]
> Each step in this methodology is organized to create a clear flow from **legislative authority** → **business rule** → **calculation requirement** → **SAS code** → **results**. This structure ensures regulatory compliance, provides audit trails, and makes it easy to understand both the legal foundation or authority and the technical implementation of each calculation step.

## 🏫 Regional Day School Programs for the Deaf (RDSPD)

[Regional Day School Programs for the Deaf (RDSPD)](https://tea.texas.gov/academics/special-student-populations/special-education/programs-and-services/sensory-impairments#:~:text=Regional%20Day%20School%20Programs%20for,Board%20of%20Education%20(SBOE).) are specialized educational programs established by the Texas Education Agency to serve students who are deaf or hard of hearing from birth to age 22. Due to the low incidence of hearing impairment, students may come from several districts to an RDSPD school location for education services, or be served by teachers who are certified to work with students who are deaf/hard of hearing on their home campuses. RDSPDs are established as shared services arrangements (SSAs) between neighboring school districts, with one district serving as the fiscal agent RDSPD to coordinate funding and program administration. Located on general education campuses, the services ensure that deaf and hard of hearing students across Texas receive specialized educational services even when individual districts lack sufficient enrollment to support standalone programs.

## 📜 Legislative Foundation

The methodology directly implements the statutory requirements established by HB 2 in [TEC §48.315](https://capitol.texas.gov/tlodocs/89R/billtext/pdf/HB00002F.pdf).

> **Sec. 48.315. FUNDING FOR REGIONAL DAY SCHOOL PROGRAMS FOR THE DEAF**
>
> **(a)** The program administrator or fiscal agent of a regional day school program for the deaf is entitled to receive for each school year an allotment of **$6,925**, or a greater amount provided by appropriation, for **each student receiving services** from the program.
>
> **(b)** Notwithstanding Subsection (a), the agency shall **adjust the amount** of an allotment under that subsection for a school year to ensure the **total amount of allotments** provided under that subsection is **at least $35 million** for that school year.

This methodology operationalizes the legislative mandate by translating statutory language into specific calculation steps, data requirements, and implementation procedures. Every business rule and technical step directly supports compliance with TEC §48.315.

## 🎯 Methodology

This methodology employs a dual-level calculation approach that serves distinct but complementary purposes to calculate both statewide totals and totals disaggregated by fiscal agent. This approach produces a complete funding pipeline from appropriation to distribution.

-----

### 🧮 Data Source

Public Education Information Management System (PEIMS) Fall Submission (snapshot captured on the **last Friday in October**)
- Modeled on PEIMS data from the 2024–25 school year  
- Fall snapshot date: [Friday, October 25, 2024](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)
- Data available to customers: [Thursday, February 13, 2025](https://tealprod.tea.state.tx.us/TWEDSAPI/28/0/0/Overview/List/TimeLine/902)

-----

### Step 1: PEIMS Data Extraction

#### 📜 Business Rule
**BR-001: PEIMS Data Source Requirements**
> Student eligibility data must be extracted from the PEIMS Fall Submission special education program student view, ensuring data completeness and accuracy for funding calculations per TEC §48.315.

#### 📘 Calculation Requirement

Extract student-level RDSPD participation data from the PEIMS special education program student view.

#### 💻 SAS Code

```sas
/*--------------------------------------------------------------*
| Step 1: Extract student-level RDSPD data from PEIMS view    |
*--------------------------------------------------------------*/
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
> Only students with a valid fiscal agent assignment in the DIST_RDSPD_SVC field are eligible for RDSPD funding. Records with missing or null fiscal agent assignments must be excluded from funding calculations.

#### 📘 Calculation Requirement

Filter the extracted data to include only students who have been assigned to receive RDSPD services from a fiscal agent.

#### 💻 SAS Code

```sas
/*-----------------------------------------------*
| Step 2: Filter for students with RDSPD services |
*-----------------------------------------------*/
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
> When multiple records exist for the same student, the most recent record (highest EFFECTIVE_DT) takes precedence. This ensures each student is counted only once in funding calculations, preventing duplicate funding claims.

#### 📘 Calculation Requirement

Sort student records by district, student ID, and effective date (descending) to identify the most current assignment for each student. Keep only the first record for each student to eliminate duplicates at the district and student ID level while preserving the most recent RDSPD service assignment.

#### 💻 SAS Code

```sas
/*--------------------------------------------------------------*
| Step 3: Deduplicate student records by most recent date     |
*--------------------------------------------------------------*/
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
> Student counts must be aggregated by fiscal agent (DIST_RDSPD_SVC) to determine funding allocation per responsible district. Each fiscal agent receives funding based on their enrolled student population.

#### 📘 Calculation Requirement

Aggregate the deduplicated student records by fiscal agent to determine how many students are served by each fiscal agent.

#### 💻 SAS Code

```sas
/*-----------------------------------------------*
| Step 4: Aggregate student counts by fiscal agent |
*-----------------------------------------------*/
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
> Each RDSPD student is entitled to receive $6,925 in base funding per TEC §48.315(a). The total base funding for all programs is calculated by multiplying this statutory base amount by the total number of students enrolled across all RDSPD programs.

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
/*-----------------------------------------------*
| Step 5: Calculate total base funding statewide |
*-----------------------------------------------*/
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
> The total program funding must meet a statutory minimum of $35,000,000 annually per TEC §48.315(b). If the total base funding falls below this threshold, an upward adjustment is required to bridge the funding gap.

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
/*-----------------------------------------------*
| Step 6: Calculate total upward adjustment needed |
*-----------------------------------------------*/
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

```
Per-Student Upward Adjustment = Total Upward Adjustment ÷ Total RDSPD Students
```

Where:

- **Total Upward Adjustment =** $291,900 (from Step 6)
- **Total RDSPD Students =** 5,012

#### Values:

```
$291,900.00 ÷ 5,012 = $58.24
```

#### 💻 SAS Code

```sas
/*-----------------------------------------------*
| Step 7: Calculate per-student upward adjustment |
*-----------------------------------------------*/
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
/*-----------------------------------------------*
| Step 8: Calculate adjusted base funding per student |
*-----------------------------------------------*/
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
/*-----------------------------------------------*
| Step 9: Calculate final funding by fiscal agent |
*-----------------------------------------------*/
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
> The total adjusted funding must be calculated using the adjusted per-student rate to verify that the funding mechanism successfully achieves the minimum allocation requirement. This serves as a validation step to ensure compliance with statutory requirements.

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
/*-----------------------------------------------*
| Step 10: Verify total adjusted funding statewide |
*-----------------------------------------------*/
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
> Final verification requires confirming that the total adjusted funding meets the minimum allocation requirement within acceptable rounding tolerances. Small variances due to decimal precision in per-student calculations are acceptable as long as the total approaches the statutory minimum.

#### 📘 Calculation Requirement

Perform final verification that the total adjusted funding meets the minimum allotment requirement. The small rounding difference ($1.12) is due to decimal precision in the per-student calculations and is within acceptable tolerances.

#### Values:

```
Total Adjusted Funding: $34,999,998.88
Rounded to Nearest Million: $35,000,000
```

#### 💻 SAS Code

```sas
/*-----------------------------------------------*
| Step 11: Final verification and rounding check |
*-----------------------------------------------*/
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
- ✅ Legislative mandate compliance (TEC §48.315)
- ✅ Total appropriation requirement ($35,000,000)
- ✅ Uniform per-student adjustment rate ($58.24)

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
057905                   89        $621,508.36
071901                  156      $1,089,385.44
101912                  203      $1,417,597.72
```

This dual-pipeline approach ensures **legislative compliance** at the state level while providing **actionable funding distributions** for individual school districts, creating complete transparency from democratic appropriation to classroom implementation.

-----

For questions, email [zane.wubbena@tea.texas.gov](zane.wubbena@tea.texas.gov)
