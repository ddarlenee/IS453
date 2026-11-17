# First Property Purchase Loans: Credit Scorecard

A credit scorecard for a bank's first-home loan, built for the IS453 Financial Analytics group assignment (Team 4) at Singapore Management University. It scores applicants aged 21 to 55 who own no property, sets an approval cutoff and interest rates, and measures the result on applicants the model never saw.

Everything uses the methods taught in the course: fine and coarse classing, Weight of Evidence (WOE) and Information Value (IV), logistic regression, scorecard scaling, and cutoff selection by profit.

## Results

| | |
|---|---|
| Segment | 72,292 applicants (23.5% of 307,511), 9.20% bad rate against 8.07% overall |
| Train / test split | 50,604 / 21,688, stratified on the outcome |
| Scorecards | Two on one scale: 13 characteristics for applicants with a bureau file, 8 for those without |
| Scale | 600 points at 1:19 odds of default, 20 points to double the odds; scores run 429 to 623 |
| Recommended cutoff | 494 (the most profitable cutoff is 505) |
| AUC, train / test | 0.697 / 0.692 |
| Test approval, recall, specificity | 89.1%, 90.5%, 25.2% |
| Test profit per 10,000 applicants | S$41.4m at a flat 2.00% rate; S$32.6m if everyone were approved |
| Expected vs actual defaults among approved test applicants | 1,477 vs 1,493 |
| Interest rate | 2.08% portfolio rate; 1.87% to 2.33% by score band |

A test AUC of 0.692 is in the "Poor" band of the course's scale (0.6 to 0.7). The scorecard leaves out education, region, housing, family status, age and loan size on purpose, which costs predictive power but keeps it fair and usable for real mortgage applicants.

## Design choices

- **No segment left behind.** The recommended cutoff is the most profitable one at which every customer group (age, gender, education, bureau file and seven crossings of them) is approved at no less than 80% of its best-treated peer's rate. This costs S$2.59m per 10,000 applicants against the most profitable cutoff. On the test set, 0 of 17 group tables fall below 80%, though two sit at exactly 0.800.
- **A separate scorecard for applicants with no bureau file** (about 14%), so they are not scored on history they do not have.
- **Class weights.** `class_weight='balanced'` is used throughout. It ranks applicants as well as an unweighted model, but its probabilities are not real default rates, so each score's probability of default is read from the actual bad rate of training applicants in the same score group.
- **Monotonic bins.** Every numerical characteristic has WOE and points moving in one direction, and every bin holds at least 5% of its applicants. Merges are set by hand.

## Repository

| Path | Contents |
|---|---|
| `IS453_Component_1_and_2.ipynb` | Business case, selecting the segment, merging the application and bureau data, data exploration |
| `IS453_Component_3.ipynb` | Fine classing, screening by IV, coarse classing, monotonic WOE, redundancy check |
| `IS453_Component_4_to_7.ipynb` | Logistic regression, tuning, scaled scorecard, cutoff, interest rates and profit, test-set evaluation, segment audit, applicant walkthrough |
| `component3_outputs/` | Bins, WOE tables, screening decisions and the train/test split |
| `component4_outputs/` | The final scorecard, cutoff and pricing tables, profit by score band, charts and summary numbers |

The notebooks include their saved outputs, so they can be read on GitHub without rerunning.

## Running it

1. Install Python and the packages. The notebooks were run with Python 3.13, pandas 2.2.3, NumPy 2.2.3, scikit-learn 1.6.1, scorecardpy 0.1.9.7, seaborn 0.13.2 and matplotlib 3.10:

   ```
   pip install pandas numpy scikit-learn scorecardpy seaborn matplotlib jupyter
   ```

2. Put the two raw data files in the same folder as the notebooks, under the names they were supplied with (`IS453 Group Assignment - Application Data.csv`, `IS453 Group Assignment - Bureau Data.csv`) or as `App_Data.csv` and `Bureau_Data.csv`. The data is not included in this repository.

3. Run the notebooks in order, each from top to bottom: Components 1-2, then 3, then 4-7. Component 3 reads the Components 1-2 notebook by its file name, and Components 4-7 read `component3_outputs/`, so keep the file names and folders as they are. The full run takes a few minutes.

## Business assumptions

These are the team's assumptions for one first-property loan, set in the settings cell of the last notebook, and are not facts from the data.

| Input | Value |
|---|---|
| Property price | S$630,000, about the median 4-room HDB resale flat |
| Loan | 75% of the price (MAS limit on a first housing loan), S$472,500 |
| Loss given default | 20% of the loan |
| Holding period | 5 years |
| Funding cost / operating cost | 1.20% / 0.25% a year |
| Reference rate / target margin | 2.00% / 0.30% |

A sensitivity table in the notebook shows how the cutoff moves with loss given default and the interest spread. At the recommended cutoff, profit turns negative if loss given default exceeds about 33%.

## Limitations

- **The outcome is a proxy.** `TARGET` records payment difficulty on short consumer loans, not default on a mortgage, so the scorecard is assumed to rank first-time buyers correctly rather than to give true mortgage default rates.
- **Sample bias.** Only approved applicants appear in the data, so risk among the weakest applicants may be understated.
- **Applicants with no bureau file.** Their risk is read low: 8.9% expected against 12.0% actual on the test set. A side effect of balanced weights, it is reported but not corrected.
- **Profit at risk-based rates** assumes that everyone approved still takes the loan, and gives the same total as a flat 2.08% rate. What risk-based pricing changes is who pays what, not the total.
- **The test set was looked at during development.** Final decisions were made on training data only, but earlier iterations of the project also inspected test results, so treat the test figures as optimistic.
- **No application dates.** Stability over time cannot be tested; the notebook includes a Population Stability Index calculation ready for monitoring after launch.

## Data and course material

The application and bureau data were supplied with the course and are not redistributed here. Business context comes from MAS, HDB and LTA web pages, listed in the References sections of the notebooks. Method references use the form FA*module* s*slide* (for example FA10.3 s11), pointing to the IS453 lecture slides and labs.
