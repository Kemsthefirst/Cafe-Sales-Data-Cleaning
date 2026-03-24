### Cafe-sales-data-cleaning

## Dataset Overview
The dataset contains café sales transactions with the following columns:

| Column | Description |
|---|---|
| `TRANSACTION_ID` | Unique identifier for each sale |
| `ITEM` | Product sold (8 categories) |
| `QUANTITY` | Number of items purchased (range 1–5) |
| `PRICE_PER_UNIT` | Price of a single item |
| `TOTAL_SPENT` | Total transaction value |
| `PAYMENT_METHOD` | How the customer paid |
| `LOCATION` | Where the transaction happened |
| `TRANSACTION_DATE` | Date of transaction |


## The Mess We Started With

| Column | Missing/Dirty Values | % of Dataset |
|---|---|---|
| TRANSACTION_ID | 0 | 0% |
| ITEM | 333 | 3.3% |
| QUANTITY | 138 | 1.4% |
| PRICE_PER_UNIT | 179 | 1.8% |
| TOTAL_SPENT | 173 | 1.7% |
| PAYMENT_METHOD | 2,579 | 25.8% |
| LOCATION | 3,265 | 32.7% |
| TRANSACTION_DATE | 460 | 4.6% |

Missing values weren't just `NaN` — they also appeared as `"Error"` and `"Unknown"` strings scattered across multiple columns.

---

## Cleaning Strategy & Thought Process

### ITEM & PRICE_PER_UNIT — Bidirectional Lookup
Rather than filling blindly, a price-to-item dictionary was built to work in both directions:
- If `ITEM` was missing but `PRICE_PER_UNIT` was present → reverse lookup to recover item
- If `PRICE_PER_UNIT` was missing but `ITEM` was present → forward lookup to recover price
- If both were missing but `TOTAL_SPENT` and `QUANTITY` were present → derived price mathematically, then recovered item via lookup
- Only the truly unrecoverable rows (where all anchor columns were missing) received a random fill

### QUANTITY — Derive First, Fill Last
- Where `TOTAL_SPENT` and `PRICE_PER_UNIT` were both present → derived `quantity = total_spent / price_per_unit`
- Remaining missing values filled with **mode (5)** since quantity is a discrete 1–5 range — mean would produce decimals that aren't valid quantities
- A rogue value of `6` was also discovered and corrected to `5`

### TOTAL_SPENT — Fully Derived
- Recalculated for all rows as `quantity × price_per_unit` after both columns were cleaned
- Ensures mathematical consistency across every row

### LOCATION & PAYMENT_METHOD — Proportional Fill
- Rather than inflating the most common category with a mode fill, missing and dirty values were distributed **proportionally** across valid categories based on their existing frequency in the dataset

### TRANSACTION_DATE — Intentional NaN
- 460 rows had missing or dirty date values
- Dates cannot be logically inferred from any other column
- A deliberate decision was made to **leave these as NaN** rather than delete the rows — they still carry valid transaction, revenue and product data useful for non-time-based analysis

---

## Tools Used
- Python 3
- Pandas
- NumPy
- Collections (defaultdict)
- Jupyter Notebook


## Key Takeaways
- Missing data isn't always `NaN` — always check for string placeholders
- Delete rows as a last resort — most gaps can be recovered through logic and derivation
- Understand your data's relationships before reaching for mean or mode
- Sometimes leaving a value as `NaN` is the most honest and analytically sound decision


## Author
Built as a personal project to sharpen data cleaning skills and document real analytical thinking.

## Data Source
The dataset was obtained from Kaggle.
🔗 [Café Sales Dirty Data - Kaggle](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training))
