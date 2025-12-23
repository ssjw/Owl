# Specification: FERS Pension Enhancement with Diet-COLA

## 1. Objective
Add support for Federal Employees Retirement System (FERS) pensions, incorporating the unique "Diet-COLA" inflation adjustment logic and the temporary FERS Supplemental Annuity.

## 2. User Interface Changes (`ui/Fixed_Income.py`)
Add a new section titled **"FERS Pension"** with the following inputs:
- **High-3 Average Salary**: Numeric input for the average of the highest three consecutive years of pay.
- **Years of Service**: Numeric input for creditable service years.
- **Commencement Age & Month**: Age and month when the annuity starts.
- **Survivor Benefit Selection**:
    - **None (0%)**: 0% reduction to the retiree's annuity.
    - **Partial (25%)**: 5% reduction to the retiree's annuity.
    - **Full (50%)**: 10% reduction to the retiree's annuity.
- **Supplemental Annuity (SRS)**: Monthly estimate of the Social Security supplement.

## 3. Backend Logic (`src/owlplanner/plan.py`)

### 3.1 Base Annuity Calculation
- **Multiplier**: 1.0% per year of service (or 1.1% if the retiree is age 62 or older with at least 20 years of service at retirement).
- **Annuity Formula**: `(Multiplier * High-3 * Years_of_Service) / 12`.
- **Survivor Reduction**: Apply the 0%, 5%, or 10% reduction to the calculated monthly amount.

### 3.2 Supplemental Annuity (SRS)
- **Logic**: The SRS amount is added to the monthly pension income from the start of retirement until the month the retiree turns 62.
- **Indexing**: The SRS is not subject to COLAs.

### 3.3 Diet-COLA Inflation Adjustment
Modify the indexing logic in the `Plan` class (specifically within or related to `_adjustParameters`):
- **Ages < 62**: No COLA is applied to the base annuity (it remains fixed in nominal dollars).
- **Ages >= 62**: Apply the "Diet-COLA" formula based on the annual inflation rate ($i$):
    - If $i < 2\%$: COLA = $i$.
    - If $2\% \le i \le 3\%$: COLA = $2\%$.
    - If $i > 3\%$: COLA = $i - 1\%$.

## 4. Proposed Implementation Steps
1.  **Update `Plan.__init__`**: Initialize variables to store FERS parameters for each individual.
2.  **Add `Plan.setFersPension`**: Create a method to receive FERS inputs and calculate the base nominal amounts.
3.  **Update `Plan._adjustParameters`**: 
    - Implement a conditional loop for FERS pensions that calculates the year-over-year adjustment using the Diet-COLA formula once the individual reaches age 62.
    - Ensure the Supplemental Annuity is only included in years where `age < 62`.
4.  **Update `ui/Fixed_Income.py`**: Add the Streamlit components to capture the new FERS-specific inputs and call `setFersPension`.
