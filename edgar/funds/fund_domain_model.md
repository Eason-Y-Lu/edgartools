# Understanding Investment Fund Structure and Relationships

## Introduction for Users and Product Managers

Investment funds can be confusing with their complex organizational structure and terminology. This document explains the key entities in our fund data model, how they relate to each other, and how users interact with fund information.

## Domain Entities and Relationships

### Key Entities

1. **Fund Company**
   - The legal entity that creates and manages investment funds
   - Examples: Vanguard, Fidelity, BlackRock, PIMCO
   - Files SEC disclosures under a single CIK (Central Index Key)

2. **Fund Series**
   - A specific investment product/strategy offered by the fund company
   - Examples: "Vanguard 500 Index Fund", "Fidelity Contrafund"
   - Has a unique Series ID (starts with 'S' followed by numbers)
   - Has its own investment objective, portfolio, and performance history

3. **Share Class**
   - Different versions of the same fund with varying fee structures, minimum investments, etc.
   - Examples: Investor shares, Admiral shares, Institutional shares
   - Has a unique Class ID (starts with 'C' followed by numbers)
   - Has its own ticker symbol and pricing

### Entity Relationships

```
Fund Company (1) ---< Fund Series (Many) ---< Share Classes (Many)
```

- A **Fund Company** can offer **multiple Fund Series** (one-to-many)
- Each **Fund Series** can have **multiple Share Classes** (one-to-many)
- Each **Share Class** belongs to exactly one **Fund Series** (many-to-one)
- All entities ultimately roll up to a single **Fund Company** that makes SEC filings

## User Perspective: Finding Fund Information

### Starting Points for Users

Users typically start their search using one of these identifiers:

1. **Ticker Symbol** (most common)
   - Example: "VFINX" for Vanguard 500 Index Fund Investor Shares
   - Uniquely identifies a specific share class
   - Easy to remember and widely used by investors

2. **Fund Name**
   - Example: "Vanguard 500 Index Fund"
   - Identifies a fund series, but not a specific share class
   - Natural language way users think about funds

3. **Fund Company Name**
   - Example: "Vanguard" 
   - Top-level entity that manages multiple funds
   - Often users know the company but not specific fund details

4. **Identifiers for Advanced Users**
   - Series ID (S000xxxxx) - identifies a specific fund series
   - Class ID (C000xxxxx) - identifies a specific share class
   - CIK - identifies the fund company in SEC database

### Common User Scenarios

1. **"I have a ticker symbol and want fund information"**
   - Most direct path: Ticker → Share Class → Fund Series → Fund Company
   - User needs: Performance data, fees, minimum investment, etc.

2. **"I want to see all share classes of a fund"**
   - Path: Fund Series → Multiple Share Classes
   - User needs: Compare expense ratios, minimums, eligibility requirements

3. **"I want to see all funds offered by a company"**
   - Path: Fund Company → Multiple Fund Series
   - User needs: Browse investment options, compare strategies

4. **"I want to see a fund's holdings and derivatives"**
   - Path: Fund Series → Portfolio Holdings (Traditional + Derivatives)
   - User needs: See what securities the fund invests in, sector breakdown, hedging strategies

## Product Management Perspective

From a product management viewpoint, the key considerations are:

### User Journeys

1. **Discovery**
   - Users search for funds by ticker, name, or company
   - Need efficient ways to find the right fund among thousands

2. **Comparison**
   - Users compare multiple share classes of the same fund
   - Users compare similar funds across companies

3. **Analysis**
   - Users analyze fund holdings, performance, and characteristics
   - Need aggregated data from multiple SEC filings

### Data Relationships for Features

1. **Fund Profile Pages**
   - Should clearly show hierarchy: "Fund X (Series) by Company Y with Classes A, B, C"
   - Should link related entities for easy navigation

2. **Search Functionality**
   - Should support all entry points: ticker, name, company
   - Should present results in a hierarchy that makes sense

3. **Portfolio Analysis**
   - Should tie portfolio holdings to specific fund series
   - Should allow time-series analysis of changing holdings

## Implementation Considerations and Gaps

### Current Implementation

In our `edgar.funds` package:

- `FundCompany` class represents a fund company (top-level entity)
- `FundSeries` represents a specific fund product/strategy
- `FundClass` represents a specific share class of a fund

The package provides a smart finder function `find_fund()` that returns the appropriate entity type based on the identifier provided (ticker, series ID, class ID, or CIK).

### Previous Implementation Gaps (Now Addressed)

1. **Terminology Mismatch (RESOLVED)**
   - We've renamed the `Fund` class to `FundCompany` to better reflect its role
   - This change aligns with how users typically think of a "fund" as the investment product (series), not the company
   - The old `Fund` class is still available for backward compatibility

2. **Entity Resolution Challenges**
   - SEC filings don't always clearly distinguish between series and classes
   - Fund naming is inconsistent across sources (may need normalization)

3. **Incomplete Data Model**
   - Some funds have sub-funds or are part of a "fund of funds" structure
   - ETFs and closed-end funds have different structures than mutual funds

4. **User Intent vs. Data Structure**
   - Users often want "the fund" without caring about class distinctions
   - Implementation needs to abstract away complexity while maintaining accuracy

5. **Data Freshness**
   - Fund information changes (new classes, mergers, name changes)
   - Need to balance caching vs. real-time accuracy

6. **Navigation Between Related Entities**
   - Current implementation excels at drill-down (company → series → class)
   - Limited support for lateral movement (compare classes, find similar funds)

### Future Improvements

1. **Enhanced Entity Resolution**
   - More robust mapping of tickers to classes
   - Better handling of fund name variations

2. **Expanded Support for Fund Types**
   - ETFs, closed-end funds, interval funds
   - Private funds and hedge funds (Form D filers)

3. **Temporal Aspects**
   - Track changes in fund structure over time
   - Handle fund mergers, splits, and renames

4. **User-Focused Abstraction**
   - Create a unified "Fund" interface that hides complexity
   - Present the right level of detail based on user needs

### Implementation Status (updated August 2025)

### User Journey Assessment

#### 1. Discovery Journey
**Progress: 90% Complete**

✓ **Implemented:**
- Smart entity lookup (via `find_fund()`) that returns appropriate entity type
- Specialized getters for each entity type: `get_fund_company()`, `get_fund_series()`, `get_fund_class()`
- Clear entity type distinction with renamed `FundCompany` class
- Ticker symbol lookup (via `find_fund(ticker)` or `get_fund_class(ticker)`. Internal reference data uses `get_class_by_ticker()`)
- Fund Series and Share Class identification
- Series-class association to connect the hierarchy
- Robust mechanisms for retrieving all series for a fund company
- Inference for associating classes with series even with incomplete data

⚠️ **Gaps:**
- Fund company search by name (could be enhanced)
- Handling of fund name variations and aliases

#### 2. Comparison Journey
**Progress: 70% Complete**

✓ **Implemented:**
- Ability to get all share classes for a specific fund series
- Basic class attributes (ticker, name) for comparison
- Hierarchical organization that groups classes by series
- Rich display formatting that shows relationships

⚠️ **Gaps:**
- Limited fee and performance data for true comparison
- No direct class-to-class comparison features
- Missing comparison metrics (expense ratios, minimums, performance)

#### 3. Portfolio Holdings (N-PORT)
**Progress: 80% Complete**

✓ **Implemented:**
- `FundReport` class for parsing N-PORT filings
- Extraction of detailed portfolio holdings into a pandas DataFrame via `FundReport.investment_data()`
- Access to general fund information, metrics, and returns from the report

⚠️ **Gaps:**
- Full mapping of all N-PORT XML fields
- Comparison capabilities between different N-PORT filings

### Feature Status

#### 1. Fund Profile Pages
**Progress: 75% Complete**

✓ **Implemented:**
- Clear hierarchy display in `__rich__` representation
- Entity relationships are properly represented
- Navigation from class to series to fund

⚠️ **Gaps:**
- Could improve the depth of information displayed
- Limited performance and fee data

#### 2. Search Functionality
**Progress: 65% Complete**

✓ **Implemented:**
- Multiple entry points (ticker, CIK, series/class IDs)
- Resolution of various identifiers to appropriate entity types

⚠️ **Gaps:**
- Limited natural language search
- No fuzzy matching for fund names
- Limited search by fund attributes

#### 3. Portfolio Analysis
**Progress: 75% Complete (Previously 55%)**   
Users want to understand what funds actually own - both traditional securities and derivatives. This has become significantly more sophisticated with enhanced N-PORT parsing.
#### Common User Scenarios
Traditional Securities Analysis:

- "What stocks/bonds does this fund hold?"
- "Show me the fund's sector allocation"
- "What are the top 10 holdings by weight?"

Derivatives Analysis (NEW CAPABILITIES):

- "Does this fund use derivatives for hedging?"
- "What's the fund's interest rate swap exposure?"
- "Show me all the fund's options positions"
- "How much leverage does this fund have through derivatives?"   

✓ **Implemented:**

Basic portfolio holdings extraction via investment_data()
Comprehensive derivatives parsing with six specialized methods:

- `derivatives_data()` - Unified view of all derivatives  
- `swaps_data()` - Interest rate, credit, and total return swaps
- `swaptions_data()` - Options on swaps with nested details
- `options_data()` - Equity/index options including nested derivatives
- `forwards_data()` - FX forwards and other forward contracts
- `futures_data()` - Future contracts with reference entities

Index reference parsing (S&P 500 INDEX, etc.)    
Nested derivative support (options on forwards/futures/swaps)  
Dynamic column structures that adapt to instrument type  

⚠️ Partial Implementation:

Warrant derivatives (WAR) - parsed but no dedicated method     
Complex structured products    
Multi-level derivative nesting (>1 level)    
Portfolio-level analytics and aggregations    

❌ Work to be done:

Sector/industry breakdown views   
Geographic allocation analysis   
Temporal portfolio comparison   
Risk analytics (VaR, duration, etc.)   
Performance attribution 

**Data Model Enhancement**  
The portfolio analysis now supports a comprehensive investment hierarchy:   
```bash
Portfolio Holdings
├── Traditional Securities
│   ├── Equity Securities
│   ├── Fixed Income Securities
│   └── Other Securities
└── Derivative Instruments (NEW)
    ├── Interest Rate Derivatives
    │   ├── Swaps (IRS, CDS, TRS)
    │   ├── Swaptions
    │   └── Interest Rate Futures
    ├── Equity Derivatives
    │   ├── Equity Options
    │   ├── Index Options
    │   └── Equity Futures
    ├── FX Derivatives
    │   ├── FX Forwards
    │   ├── FX Options
    │   └── Currency Swaps
    └── Other Derivatives
        ├── Commodity Derivatives
        └── Credit Derivatives
```

### Overall Assessment

Recent improvements have significantly enhanced the **Discovery Journey** by introducing a more intuitive API with clearer entity naming. The new `find_fund()` smart factory function and specialized getters like `get_fund_company()`, `get_fund_series()`, and `get_fund_class()` greatly improve the user experience. Renaming the top-level entity from `Fund` to `FundCompany` addresses a major terminology mismatch that was previously confusing.

The enhanced inference logic in entity relationships directly supports key user scenarios like "I want to see all share classes of a fund" and the improved navigation between entities makes it easier to explore the fund hierarchy.

**Current State: ~78% Complete Across All Journeys**

The core relationships between entities are now robust and intuitive, providing a solid foundation for building more advanced features to fulfill these user journeys completely. The API is now more predictable and better aligned with user expectations.