
# Data Preparation & Modeling Approach

I manually analysed all the files in Excel to see if I can find any patterns, any data consistency issues and ideas that may help me design the data model and the dashboard. Overall, I observed a lot of date conversions to be done, null exclusions, data de-duplication requirements. This step is important to ensure that a clean, stable and consistent data model is connected to the dashboard

### Reviewing Source Files and Recording Observations

#### `Player.csv`

Table includes additional fields like `wantsBonus`, `wantsCommunication`, `wantsSMS`, and `welcomeOffer`. Although not relevant to my analysis right away, it would eb good to retain them for future applications.

#### `PaymentTransaction.csv`

This table contains deposit and withdrawal activity. The transactions appear in multiple currencies, so exchange conversion should be done before loading to teh dashboard. As mentioned, `PENDINGWITHDRAWAL` and `CANCELWITHDRAWAL` will be excluded for the assignment.

#### `GameTransaction.csv`

This will become the main gameplay fact table and used for calculating most gaming KPIs. Upon inspection, I observed that a single `BetID` may appear in multiple rows, likely due to the way some bets are recorded. The table also contains mixed data-types for `BetID` values, and is missing some `ChannelUID` values.

#### `Game.csv`, `GameProvider.csv`, `GameCategory.csv`

These tables contain game metadata. Since they are all dimensional data, it would make sense to combine these into a single game dimension table. Upon review, blank rows were identified in `GameProvider.csv`, and duplicate rows were identified in `GameCategory.csv`.


### Data Cleaning and Preprocessing

#### `Player.csv`

-   Converted `RandomizedBirthDate` from numeric format into a valid date field.
    

#### `PaymentTransaction.csv`

-   Converted `txDateTime` into a valid DateTime field.
    
-   Excluded rows where `txType` was `PENDINGWITHDRAWAL` or  `CANCELWITHDRAWAL`
        
-   Joined the table for exchange-rate conversion to EUR on Date + Currency Key.
    

#### `GameTransaction.csv`

-   Removed rows where `BetID` was null.
    
-   Removed rows where `ChannelUID` was null.
    
-   Removed rows where both wager cash and wager bonus were zero. Assuming that a valid wager must contain some non-zero stake, whether cash or bonus component.

    
-   Joined the table for exchange-rate conversion to EUR on Date + Currency Key.
    

#### `GameProvider.csv`

-   Removed blank rows.
    

#### `GameCategory.csv`

-   Removed duplicate rows.
    

#### `Game.csv`

-   Joined `GameCategory.csv` and `GameProvider.csv` tables to consolidate
    
![image.avif](https://user4551.na.imgto.link/public/20260314/image.avif)
----------

# Data ETL & Design Decisions

### Building the ETL Dataflow

I used KNIME, a free, open-source ETL and Business Intelligence tool to load the csv files and perform the cleaning, joins aggregations etc. I chose this over a locally hosted SQL Database because of my system OS Limitations and the flexibility that comes with using a tool like KNIME, where all operations are broken down into their individual steps - making it easier for me to make changes and regenerate multiple csv files instead of doing a manual export anytime I had to retrace my steps. KNIME combined some SQL-like operations and some operations I may have ended up doing in Python, making it a quicker and better choice for me to design in for this project.

#### The KNIME ETL Model

Below, you can see a screenshot of the ETL Model I built in KNIME to load all the 7 csv files, perform all operations and then export them into 4 csv files at the end.

![workflow.svg](https://user4551.na.imgto.link/public/20260314/workflow.svg)

#### Early currency standardization

Both gameplay and payment data required currency conversion. To avoid repeating conversion logic later in the reporting layer, I standardised monetary values to EUR early in the ETL process. I joined the tables on a key of Date and Currency so that only the currency exchange rate relevant to the date on the transaction was considered.

#### Bet-level consolidation in gameplay data

In `GameTransaction.csv` it seemed that the same `BetID` could appeared across multiple result rows. To simplify calcualtions, wager and result activity was aggregated at `betID` level so that each bet could be represented by the same number of rows. Additionally, I also **pivoted** the wager and result data after aggregation and currency conversion. This was done so that **every row represented a unique `betID` in the table.** So now we had one row with w `betID` and columns for `wagerCash`, `wagerBonus`, `wagerDate` and similarly `resultCash`, `resultBonus`, `resultDate`.

#### Enriched game dimension table

Instead of keeping game, category, and provider data separate in Tableau, `Game.csv`, `GameProvider.csv`, and `GameCategory.csv` were combined into a single game dimension table. 

### Other Design  Decisions

#### Pre-Caluclating some KPIs
Some KPIs were calculated in the ETL Stage before loading to Tableau itself,  such as:
- Cash turnover = Wagered amount in cash
- Bonus turnover = Wagered amount in bonus
-  Cash winnings = Result amount in cash
-  Bonus winnings = Result amount in bonus
-  Turnover = Cash turnover + Bonus turnover
-  Winnings = Cash winnings + Bonus winnings
-  Cash result = Cash turnover - Cash winnings
-  Bonus result = Bonus turnover - Bonus winnings
-  Gross result = Turnover - Winnings
-  Deposit
-  Withdrawal
-  Net deposit = Deposit – Withdrawal

#### `BetID` treated as a string

`BetID` was treated as a string rather than a numeric field because several valid records contained non-numeric or mixed-format values. These records were retained rather than discarded.

#### Timestamp handling for aggregated bets

Where multiple rows existed for the same `BetID`, the maximum timestamp was taken as the final timestamp for those related rows.


# Data Model for Tableau

This resulted in the following data model : 

![image.avif](https://user4551.na.imgto.link/public/20260314/image-1.avif)

Which looks like this in Tableau:

![image.avif](https://user4551.na.imgto.link/public/20260314/image-2.avif)

-   **fact_Game**: bet-level KPIs such as turnover, winnings, and gross result standardized to EUR
    
-   **fact_payment**: deposit and withdrawal activity standardized to EUR
    
-   **dim_player**: player attributes such as country, birth date, and status
    
-   **dim_game**: game metadata including game name, provider, and category
    

This structure made it easier to build dashboards that could answer business questions from both a player perspective and a game-performance perspective, while keeping the Tableau model clean and easy to understand.

----------

----------
