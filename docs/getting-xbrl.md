## Overview

The `edgar.xbrl` module provides a powerful yet user-friendly API for processing **XBRL (eXtensible Business Reporting Language)** financial data from SEC filings. 


## Key Features

- **Intuitive API**: Access financial statements with simple, readable method calls
- **Multi-period Analysis**: Compare financial data across quarters and years with statement stitching
- **Standardized Concepts**: View company-specific terms or standardized labels for cross-company comparison
- **Rich Rendering**: Display beautifully formatted financial statements in console or notebooks
- **Smart Period Selection**: Automatically identify and select relevant periods for meaningful comparisons
- **DataFrame Export**: Convert any statement to pandas DataFrames for further analysis

## Getting Started

You can get the XBRL from a single filing, or stitch together multiple filings.

### Getting XBRL from a single filing

For a single filing you can use `filing.xbrl()` to get the XBRL data, and then access the financial and other statements.

```python
from edgar import Company
from edgar.xbrl.xbrl import XBRL

# Get a company's latest 10-K filing
company = Company('AAPL')
filing = company.latest("10-K")

# Parse XBRL data
xb = filing.xbrl()

# Access statements through the user-friendly API
statements = xb.statements

# Display financial statements
balance_sheet = statements.balance_sheet()
income_statement = statements.income_statement()
cash_flow = statements.cashflow_statement()
```

### Getting XBRL from multiple filings

You can also stitch together multiple filings to create a multi-period view of financial statements. This uses the `edgar.XBRLS` class to combine data across multiple filings.
Each filing should be of the same type (e.g., all 10-Ks or all 10-Qs) and from the same company.

```python
from edgar import Company
from edgar.xbrl import XBRLS

# Get multiple filings for trend analysis
company = Company('AAPL')
filings = company.get_filings(form="10-K").head(3)  # Get the last 3 annual reports

# Create a stitched view across multiple filings
xbrls = XBRLS.from_filings(filings)

# Access stitched statements
stitched_statements = xbrls.statements

# Display multi-period statements
income_trend = stitched_statements.income_statement()
balance_sheet_trend = stitched_statements.balance_sheet()
cashflow_trend = stitched_statements.cashflow_statement()
```

## User-Friendly Features

### Simple Statement Access

Access common financial statements with intuitive methods:

```python
# Get basic statements
balance_sheet = statements.balance_sheet()
income_statement = statements.income_statement()
cash_flow = statements.cashflow_statement()
statement_of_equity = statements.statement_of_equity()

# Access any statement by type
comprehensive_income = statements["ComprehensiveIncome"]
```

### Smart Period Views

Choose from intelligent period selection views:

```python
# See available period views
period_views = statements.get_period_views("IncomeStatement")
for view in period_views:
    print(f"- {view['name']}: {view['description']}")

# Render with specific view
annual_comparison = statements.income_statement(period_view="Annual Comparison")
quarter_comparison = statements.income_statement(period_view="Quarterly Comparison")
```

### Easy Conversion to DataFrames

Transform any statement into a pandas DataFrame for further analysis:

```python
# Get DataFrame of income statement
df = income_statement.to_dataframe()
```

## Statement Stitching for Trend Analysis

The XBRLS class combines data from multiple periods with intelligent handling of concept changes:

```python
# Create stitched statements across multiple filings
xbrls = XBRLS.from_filings(filings)
stitched = xbrls.statements

# Get a three-year comparison of income statements
income_trend = stitched.income_statement(max_periods=3)

# Convert to DataFrame for time series analysis
trend_df = income_trend.to_dataframe()
```

## Rendering Options

The XBRL2 module provides flexible output options for financial statements:

```python
# Display with default styling as Rich tables in console/notebooks
print(statements.balance_sheet())

# Show full date ranges for duration periods
print(statements.income_statement(show_date_range=True))

# Customize period view
print(statements.income_statement(period_view="Annual Comparison"))

# Convert to pandas DataFrame for analysis
df = statements.to_dataframe("BalanceSheet")

# Export the statement to markdown
income_statement = statements.income_statement()
markdown_text = income_statement.render().to_markdown()
```

### Statement Display Options

The rendering system offers several customization options:

| Option | Description |
| ------ | ----------- |
| `standard=True` | Use standardized labels for cross-company comparison (default) |
| `standard=False` | Use company-specific labels as reported in the filing |
| `show_date_range=True` | Show complete date ranges for duration periods (e.g., "Jan 1 - Mar 31, 2023") |
| `show_date_range=False` | Show only end dates for cleaner presentation (default) |
| `period_view="Name"` | Select a predefined period view ("Annual Comparison", "Quarterly Comparison", etc.) |
| `period_filter="duration_..."` | Filter to a specific period by period key |

### The `RenderedStatement` Class

The `render_statement()` function returns a `RenderedStatement` object, which provides multiple output formats:

```python
# Get a rendered statement
statement = xbrl.render_statement("BalanceSheet")

# Display as Rich table (default)
print(statement)

# Convert to pandas DataFrame 
df = statement.to_dataframe()

# Export to markdown
markdown = statement.to_markdown()
```

### Customizing Statement Appearance

The rendering engine automatically handles:

- Proper monetary formatting with scale indicators (thousands, millions, billions)
- Appropriate indentation for statement hierarchy
- Formatting of section headers and dimension items
- Correct display of share counts and per-share values
- Fiscal period indicators in statement titles
- Unit notes (e.g., "In millions, except per share data")

For stitched multi-period statements, you can control the number of periods and date formatting:

```python
# Get 3-year comparison with full date ranges
annual_trend = stitched_statements.income_statement(
    max_periods=3, 
    show_date_range=True
)
```

## Advanced Features

### Custom Period Selection

```python
# Get specific periods from available options
available_periods = xbrl.reporting_periods
latest_period = available_periods[0]

# Render with specific period
if latest_period['type'] == 'instant':
    period_filter = f"instant_{latest_period['date']}"
    latest_balance_sheet = statements.balance_sheet().render(period_filter=period_filter)
```

### Statement Data Exploration

```python
# Get raw statement data for custom processing
raw_data = statements.balance_sheet().get_raw_data()

# Extract specific information
assets = [item for item in raw_data if 'assets' in item['label'].lower()]
```

## Design Philosophy

The XBRL2 module is designed with these principles:

1. **User-First API**: Simple methods that match how financial analysts think about statements
2. **Intelligent Defaults**: Smart period selection and formatting that "just works" out of the box
3. **Flexible Output Options**: Rich tables for display, DataFrames for analysis, and raw data for custom processing
4. **Consistency Across Companies**: Standardized concepts that enable cross-company comparison

## Period Selection Logic

The XBRL2 module implements sophisticated period selection logic to ensure appropriate periods are displayed for financial statements:

### Quarterly Statement Period Selection

When rendering quarterly statements (when fiscal_period_focus is Q1, Q2, Q3, or Q4):

1. The system identifies true quarterly periods by filtering duration periods to those with 80-100 day durations
2. If quarterly periods are found, the most recent one is selected as the current quarter
3. For comparison, the system looks for periods with similar duration from approximately 1-2 years prior
4. If no quarterly periods are found, it falls back to the most recent period with a warning

### Annual Statement Period Selection

For annual reports (when fiscal_period_focus is FY):

1. Annual periods are identified by looking for ~365 day durations or fiscal year markers
2. The system prioritizes periods that align with the entity's fiscal year end
3. Up to three most recent fiscal years are displayed in chronological order

This intelligent period selection ensures appropriate periods are displayed for statements, with robust fallbacks when ideal periods aren't available.

## Enhanced Facts API

The XBRL2 module includes a powerful facts query interface for direct access to individual XBRL facts:

```python
from edgar import Company
from edgar.xbrl import XBRL

# Parse XBRL data
company = Company('AAPL')
filing = company.get_filings(form='10-K').latest()

xbrl = XBRL.from_filing(filing)

# Access the facts view
facts = xbrl.facts

# Query facts by various attributes
revenue = facts.query().by_concept('Revenue').to_dataframe()
balance_sheet_facts = facts.query().by_statement_type('BalanceSheet').to_dataframe()

# Use predefined period views - returns important metadata including available periods
income_views = facts.get_available_period_views('IncomeStatement')
for view in income_views:
    print(f"- {view['name']}: {view['description']} ({view['facts_count']} facts)")

# Get facts filtered by period view
annual_comparison = facts.get_facts_by_period_view('IncomeStatement', 'Annual Comparison')

# Flexible text search across all text fields (concept, label, element name)
earnings_facts = facts.search_facts("Earnings Per Share")

# Filter by period keys - useful for custom period selection
facts.query().by_period_keys(['duration_2023-01-01_2023-12-31',
                              'duration_2022-01-01_2022-12-31']).to_dataframe()

# Query dimensional data
facts_by_segment = facts.query().by_dimension('Segment').to_dataframe()

# Safe numeric value filtering with proper None handling
large_income_items = (facts.query()
    .by_statement_type('IncomeStatement')
    .by_value(lambda v: v > 1_000_000_000)
    .sort_by('numeric_value', ascending=False)
    .to_dataframe())

# Time series analysis
revenue_over_time = facts.time_series('Revenue')
```

## XBRL Calculation Support

The XBRL2 module properly handles calculation relationships from XBRL calculation linkbases:

```python
# Values are automatically adjusted according to calculation weights
# For example, elements with negative weights (-1.0) like "IncreaseDecreaseInInventories"
# are automatically negated to maintain proper calculation relationships
cash_flow_statement = statements.cashflow_statement()

# The calculation trees are accessible for inspection
for role_uri, calc_tree in xbrl.calculation_trees.items():
    print(f"Calculation tree: {calc_tree.definition}")
    for element_id, node in calc_tree.all_nodes.items():
        if node.weight != 1.0:
            print(f"- {element_id}: weight={node.weight}")
```

The parser intelligently applies calculation weights to ensure consistent financial data presentation:

1. **Expense Concept Consistency**: Major expense categories (R&D, SG&A, Marketing, etc.) are consistently positive across companies, matching SEC CompanyFacts API behavior
2. **Cash Flow Integrity**: Elements with negative weights (-1.0) in cash flow statements maintain proper sign relationships for accurate calculations
3. **Legitimate Negatives Preserved**: Concepts that should be negative (tax benefits, foreign exchange gains/losses) retain their intended signs
4. **Cross-Company Comparability**: Eliminates inconsistencies where MSFT showed R&D as negative while AAPL showed positive values

```python
# Example: R&D expenses are now consistently positive across companies
msft_statements = msft_xbrl.statements.income_statement()
aapl_statements = aapl_xbrl.statements.income_statement()

# Both show R&D as positive values for proper comparison
msft_rnd = msft_statements.get_concept_value("ResearchAndDevelopmentExpense")  # $32.5B (positive)
aapl_rnd = aapl_statements.get_concept_value("ResearchAndDevelopmentExpense")  # $31.4B (positive)
```

## Future Enhancements

- Enhanced support for non-standard financial statements
- Interactive visualization options
- Expanded dimensional analysis capabilities
- Automatic footnote association
- Financial ratio calculations
- Advanced calculation validation and reconciliation
