# Bank Client Risk Analysis (Power BI)

A four-page Power BI dashboard, plus a hidden data quality page, that analyses credit risk across 3,000 bank client records.

**Headline finding:** the risk ratings do not track leverage. Six in ten clients owe more than they hold in financial assets, and that share is almost the same in every rating band. No branch, customer group or banking contact explains it, so the issue is portfolio-wide.

## Dataset

- 3,000 client records, 25 columns: demographics, income, deposits, loans, business lending, credit card balances, a 1-5 risk weighting, branch and banking contact.
- Source: the sample dataset from the Udemy course *Master Data Analysis: Learn Python, EDA, Stats, Excel, SQL, Power BI/Tableau, Fabric, Predictive Analytics & ETL Basics* by Dr. Satyajit Pattnaik and Zep Tech Solutions. Credit for the data goes to the course.
- The analysis, dashboard design, measures and written report are my own work.

## Business questions

| Page | Question | Audience |
|---|---|---|
| 1. Executive Overview | How big is the book, and how risky is it? | Senior leadership |
| 2. Credit and Leverage Risk | Who is over-borrowed, and where is the exposure? | Credit and risk team |
| 3. Customer Segmentation | Which customer groups drive exposure and risk? | Marketing and product |
| 4. Branch and Relationship Performance | Which branches and banking contacts need attention? | Operations |
| Data Quality (hidden) | What should a reader know before trusting the numbers? | Data owner and risk team |

## Key findings

- **Exposure and assets:** $4.38bn of credit exposure against $3.79bn of financial assets. The average risk weighting is 2.25 on a 1-5 scale.
- **Ratings skew low:** 68.6% of clients are rated Very Low or Low. The 482 High and Very High clients (16.1%) hold 23.8% of exposure.
- **Debt above assets:** 1,836 clients (61.2%) owe more than they hold, with $2.96bn of exposure (67.5% of the total). The share is 57-63% in every rating band.
- **Ratings do not track leverage:** total debt is 8.5 times annual income. Very Low rated clients carry the highest multiple (10.6x) and High and Very High carry 7.4x.
- **Debt above income:** 1,273 clients (42.4%) owe more than ten times their annual income and hold 53.7% of exposure.
- **Branch 3 is large, not riskier:** it holds 45.4% of exposure with 45.1% of clients. Per-client borrowing, DTI and risk are close across all four branches.
- **Segments look the same:** average exposure per client and average risk weighting are nearly identical across age band, tenure, loyalty tier, fee structure and nationality.
- **Banking contacts:** 49 contacts each serve clients in all four branches, with 43-77 clients each. High-risk share varies from 7% to 26%, but the groups are small, so this is a review list and not a ranking.

## Definitions and assumptions

| Item | Definition |
|---|---|
| Risk category | From the risk weighting: 1 Very Low, 2 Low, 3 Medium, 4 High, 5 Very High |
| Total credit exposure | Bank loans + business lending + credit card balances. Card balances are treated as separate debt (0.2% of exposure; excluding them changes no conclusion) |
| Total assets | Bank deposits + checking + savings + foreign currency + superannuation for clients aged 60 and over |
| Retirement age threshold | Default 60, adjustable with a slicer on pages 1 and 2. Superannuation of younger clients is excluded because it cannot be accessed to repay debt |
| Foreign currency | Assumed to be already converted to the reporting currency |
| Debt to income (DTI) | Total exposure divided by annual income. This is a leverage multiple, not a payment ratio, so it should not be compared with the usual lending percentage benchmarks |
| Over-leveraged client | A client whose exposure is above their total assets |
| High-risk share | Share of clients with a risk weighting of 4 or 5 |
| Currency | Amounts are shown in AUD. The source file does not state a currency |

## Data quality and limitations

- **Duplicate client IDs:** 3,000 records share only 2,940 unique IDs (119 records are affected). The duplicates are different people, so I created a unique customer key and count clients with it.
- **Shared names:** 174 records share a name with another client, so names cannot identify a client.
- **Clients aged 17:** 39 records show an age under 18 while holding credit products ($56.6M, 1.3% of exposure). They stay in all figures and are flagged with an `Age Check` column, so they can be filtered. They are listed on the Data Quality page.
- **Property is not factored in:** the data records how many properties each client owns (about 76% own at least one), but not their value, whether they were pledged as collateral, or which loan they would secure. Loans do not appear to be sized to property ownership, and ownership has no relationship with the risk rating. Assets are therefore financial holdings only, and debt above assets may be overstated for property owners. Property values and collateral details would be needed to refine this.
- **Currency not stated** in the source file (see above).

## How it was built

**Power Query**
- Added an index column as a unique customer key
- Converted the `Joined Bank` text dates (dd/mm/yyyy) using the English (United Kingdom) locale

**DAX (selected measures)**

```dax
Total Credit Exposure =
SUM(Clients[Bank Loans]) + SUM(Clients[Business Lending]) + SUM(Clients[Credit Card Balance])

Total Assets =
VAR ThresholdAge = 'Retirement Age Threshold'[Retirement Age Threshold Value]
RETURN
    SUM(Clients[Bank Deposits]) + SUM(Clients[Checking Accounts]) + SUM(Clients[Saving Accounts])
        + SUM(Clients[Foreign Currency Account])
        + CALCULATE(
            SUM(Clients[Superannuation Savings]),
            KEEPFILTERS(Clients[Age] >= ThresholdAge)
        )

DTI Ratio = DIVIDE([Total Credit Exposure], SUM(Clients[Estimated Income]))

High Risk Share =
DIVIDE(CALCULATE([Customers], Clients[Risk Weighting] >= 4), [Customers])

Over-Leveraged Clients =
VAR T = 'Retirement Age Threshold'[Retirement Age Threshold Value]
VAR OverLev =
    FILTER(
        Clients,
        (Clients[Bank Loans] + Clients[Business Lending] + Clients[Credit Card Balance])
            > (Clients[Bank Deposits] + Clients[Checking Accounts] + Clients[Saving Accounts]
                + Clients[Foreign Currency Account]
                + IF(Clients[Age] >= T, Clients[Superannuation Savings], 0))
    )
RETURN COUNTROWS(OverLev)
```

**Power BI features used:** calculated columns and sort-by-column, a what-if parameter, symmetry shading on a scatter chart, Top N filters, conditional formatting, edit interactions, synced slicers, drill-through, a hidden page and page navigation buttons.

## Opening the dashboard

1. Download `Bank_Risk_Analysis.pbix` from this repository.
2. Open it in Power BI Desktop (Windows). The data is embedded, so no refresh is needed.
3. Use the home button and arrows to navigate. Right-click a branch on page 4 and choose **Drill through** for its client list. The (i) icon opens the hidden Data Quality page.

## Repository contents

```
Bank_Risk_Analysis.pbix     the dashboard
Executive_Report.pdf        written report for senior leadership
README.md
```

## Possible next steps

- Add property values and collateral details to measure secured and unsecured exposure properly
- Rebuild the rating so that leverage (exposure relative to income and assets) is part of it
- Obtain the data owner's confirmation on the under-18 records and the duplicate IDs

## Author

**Collins Chidubem Nweze**, Data and Business Intelligence Analyst

[LinkedIn](https://linkedin.com/in/collins-nweze) | [Portfolio](https://sites.google.com/view/collins-nweze-portfolio/home)

## Acknowledgements

Dataset and course by Dr. Satyajit Pattnaik and Zep Tech Solutions on Udemy.
