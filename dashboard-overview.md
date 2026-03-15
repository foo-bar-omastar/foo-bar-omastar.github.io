
# Dashboard Overview

#### General Structure

This dashboard was created as a daily overview dashboard. The idea was to give a snapshot of the health of the business at a daily-level granularity. 

 - The Dashboard begins with a set of slicers that enable the user to filter by different attributes such as age group, game category, country, date/time, platform etc.
 - The number cards give a summarized overview of the main KPIs of interest
 - The graph gives an hourly overview of the performance of the business
 - The bottom-half of the report is split by player and games allowing the user to drill-down into specific attributes, players, games etc to view more detailed information
 - Throughout the dashboard, I have also experimented with a couple of different techniques for ranking players/games which is explained in the sections below.

All visuals can be used as filters, enablingt he suer to click on any part of the dashbaord and reloading the dashboard with the specific slice of data.


## Reporting Requirements

As specified in the material provided, I tried to include the requested reporting requirements in teh dashboard. Below is more information on what I did/how I worked with each of them.

#### No. of Wagers
This was achieved by a title card on the top of the dashboard that counts the distinct number of BetIDs :  `COUNTD([Bet ID])`

#### No. of Players
This was achieved by a title card on the top of the dashboard that counts the distinct number of PlayerIDs in the fact_game table :  `COUNTD([playerID])`

We are counting from the fact_game table because we want to look at players that are active on the paltform. In addition, I have also displayed the active players as a %age of the total players from the dim_player table. This allowed me to udnerstand how many of the registered users are actually active on our products.

#### No. of Deposits
This was achieved by a title card on the top of the dashboard that counts the distinct number of IDs in the fact_payment table :  `COUNTD([ID])` .  In addition, I have also displayed the monetary value of the deposits in EUR by summing the total deposits as `SUM([Deposit])`. 

In this scenario, it was okay to count all players for deposits irrespective of whether they have been active on the platform or not. 

#### No. Of Cancelled Withdrawals
This KPI was treated a little differently, since we excluded all the transactions that were `CANCEL_WITHDRAWAL` during the ETL Stage itself. So I have calculated it spearately, manually in Excel, and sharing the result below:
`Cancelled Withdrawals = 10 across 7 Distinct Players`

#### Top List with regards to Turnover
A toplist allows the user to view a ranked list of players/games etc based on a specific metric, in this case - Turnover. In order to build a more comprehensive visual, I built an entire Toplist dedicated to players with a few key metrics that would be of interest such as `Cash Turnover`, `Turnover`, `Turnover`,`Profit (Gross Result)`, `# of Bets`, `Return to Player`.

This enables the user to sort in any order they require for any of those metrics that they may require. This way, one single toplist catered to different ways of viewing the data.

I have provided a visual-specific filter incase the user wants to look at, say, turnover for players with minimum 5 bets. This way we can have thresholds for the toplist and have it remain unaffected by low-volume players. 

##### Ranking Players in Players Toplist
In this visual, I have also expreimented with an `Overall Player Rank` which takes into account  `Cash Turnover`, `Net Deposit`, `Turnover Per Spin`,`Profit (Gross Result)`, `Return to Player`. The Calculation ranks the players across each of these KPIs individually and then creates a consolidated rank by multiplying the ranks of each of thse KPIs. The Player with the lowest product of ranks is ranked the higest overall player and so on.

This is a rough approach to ranking players overall where each KPI contributes equally towards the overall rank. This ranking was built just to give a more holistic indication to the users with regards to the top players. In further sections, I have explored another method of ranking and placing players into tiers.

#### Toplist with the most Profitable Customers
This leverages the same toplist that I have built and mentioned above. The user may sort the toplist in descending order of Profit to view the players that are the most profitable. 

#### Show the turnover development over time (hourly resolution) for customers in Sweden
This was achieved by the line and bar graph in the upper half of the dashboard. 
This visual allows the user to see `Hourly Turnover & Profit vs No. of Players by Country`. So based on the filter selected, one can view the hourly turnover for one or more countries together. As with the other visuals, this visual can be filtered by any of the other slicers or visuals enabling a more dynamic view of turnover development over time.

I also wanted to include the number of active players at every hour as a bar graph and add another line graph that shows the profit at an hourly level as well. This helps put things in perspective visually where one may see high turnover and high/low profits or players giving a direct indication of the quality of activity.

#### Which country was most profitable
This has been highlighted by a Profit table in the upper half of the dashboard. Since my focus was to drill-down deeper at a player and game level, this was the only Country specific visual I wanted to include. The table gives a breakdown of the `Profit` by the `Cash Profit` and `Bonus Profit` so that we can determine the quality of the profit.

#### Which game has the least profitable per spin
For this, I leveraged a game-focussed toplist with a few key metrics that would be of interest such as `Profit per Spin`, `Turnover per Spin`, `Turnover`,`Profit (Gross Result)`, `# of Bets`, `Return to Player`.

This enables the user to sort in any order they require for any of those metrics that they may require. This way, one single toplist catered to different ways of viewing the data.

In addition to visual-specific filter incase the user wants to look at, games with minimum 5 bets, I have also created a parameter that allows the toplist to **transform** into a toplist for game providers or even game categories. This way the user can view the same metrics but not at a higher level if needed.

##### Ranking Games in Game Toplist
Similar to the players toplist, I expreimented with an  `Overall Rank` which takes into account  `# of Bets`, `Profit per Spin`, `Turnover Per Spin`,`Profit (Gross Result)`, `Return to Player`. The Calculation ranks the players across each of these KPIs individually and then creates a consolidated rank by multiplying the ranks of each of thse KPIs. The Player with the lowest product of ranks is ranked the higest overall player and so on.

This is a rough approach to ranking players overall where each KPI contributes equally towards the overall rank. This ranking was built just to give a more holistic indication to the users with regards to the top players. 


#### What is significant for the best 100 customers
The data provided did not have enough players and data to evaluate 100 best customers. However, I wanted to instead look at TOP players and determine if they can be ranked by their playing pattern and behaviour. 

A player may be considered among the best players for the business if they are engaging with the products constantly/frequently and are playing with a relatively large turnover compared to the other players. For the business, these best players may not have a significantly high return to player, and have good profits for the company. 

Based on the above, I built the following metrics:
 -  **`Player Engagement`** : Measures the player engagement based on `# of Bets` and `# of Active Hours in a Day` . Any player that was in the upper half of the Quartile Range `(> 80%)` was considered a high-engagement player, mid-range as medium and the rest as low-engagement.

 -  **`Player Game Value`** : Measures the value of a player based on `Turnover`, `Profit` and `Return To Player` . Any player that was in the upper half of the Quartile Range `(> 70-80%)` and average `Return to Player` was considered a high-value player.

The values from both these metrics helped me build a TIER structure to categorize players, where the top or VIP Tier would have been the Best Customers:

| Engagement | Game Value | Player Status                        |
| ---------- | ---------- | ------------------------------------ |
| High       | High       | VIP                                  |
| High       | Medium     | VIP                                  |
| Medium     | High       | Plays Heavily, Profitable Player     |
| Medium     | Medium     | Plays Heavily, Profitable Player     |
| High       | Low        | Plays Heavily, Not Profitable Player |
| Medium     | Low        | Plays Heavily, Not Profitable Player |
| Low        | High       | High Stakes, Low Frequency Player    |
| Low        | Medium     | High Stakes, Low Frequency Player    |
| Low        | Low        | Low Value Casual Player              |


#### What is significant for the 100 most improving customers

This question leverages the solution mentioned above. Players that are playing Heavily but are not yet profitable could be considered as imporving players. This is one way of looking at improving customers.

Alternatively, it would have been ideal to have more days of data for gaming information. That would have enabled us to measure a day-to-day growth in player engagement and game value. Since the metric for `improving customers` would be time based, it is important to have a larger timeframe of information. If players are playing at the same hours everyday, or playing for a number of days in succession or if they are improving their turnover or number of bets over a period of days then that would be ideal for identifying `improving customers`


## Answers


#### What other KPIs do you think is important for an online casino?

For an online casino, it is important to understand the value gained from the players and games. The case study provides some data for player and game usage, but it could be beenficial to have additional digital product analytics data.

 - **`Player Relevant KPIs`**
	 - Tracking *conversion* from sign-up to active player
	 - `Bonus vs Cash Driven Profit`, `Player Lifetime Value`
	 - `Player Churn Rate`, `Retention Rate (or Repeat Deposit Rate)`, 
	 - Time spent on the platform browsing and engaging - like a click through rate
	 - Spread of players across the range of products - are there a few heavy players or do we have a decent mix of casual to serious players across a lot of games.
	 - Player behaviour and playing pattern - `Average Bets per Active Player`,`Average Games per Active Player`,`Number of Withdrawals` etc.
	 - 
 - **`Game Relevant KPIs`**
	 - `Player Conversion` - Players who actually placed a first bet on the game vs players who just launched the game
	 - `Weekly replay rate`, `Impressions per game`
	 - Game Value - not just from a player engagement perspective, but also from a business perspective. What games are profitable, what games are popular among players, what games are dropping off in usage and engagement, are there any bugs/issues causing drop-offs etc.
	 - Game Categories - analysing usage patterns across game categories -  `Return To Player by Game`, `Average Stake by Game`, `Players by Game`, `Gross Result per Bet` etc

 - **`Additional User-Defined KPIs`** : Below are some of the derived/additional metrics I created for the case study:
	 - `Score Game Popularity = 0.40 * [Players] + 0.30 * [Bets] + 0.30 * [Turnover]` Calculating game popularity by a weighted score using number of players, number of bets and turnover for the game
	 - `Overall Game Rank` which takes into account  `# of Bets`, `Profit per Spin`, `Turnover Per Spin`,`Profit (Gross Result)`, `Return to Player`. The Calculation ranks the players across each of these KPIs individually and then creates a consolidated rank by multiplying the ranks of each of thse KPIs. The Player with the lowest product of ranks is ranked the higest overall player and so on.
	 - `Overall Player Rank` which takes into account  `Cash Turnover`, `Net Deposit`, `Turnover Per Spin`,`Profit (Gross Result)`, `Return to Player`. 
	 - `Player Active Hours = COUNTD(DATETRUNC('hour', [Wager txDate])` to calculate the hours in a day the user has engaged with the products
	 - `Day Zone` Splitting the day into morning/afternoon/night/latenight zones for analysing player behaviour


#### How to measure different levels of VIP?
As detailed above, in the `100 best customers` section, I measured different levels of VIP based on the players engagement and the value of the games they played. This allowed me to look at player performance across 5 metrics to categorize them. Based on their playing behaviour, we can assign VIP Tier Statuses to players:

| Player Category                      | VIP Tier |
| ------------------------------------ | -------- |
| VIP                                  | VIP      |
| Plays Heavily, Profitable Player     | Premium  |
| Plays Heavily, Not Profitable Player | Rising   |
| High Stakes, Low Frequency Player    | No Tier  |
| Low Value Casual Player              | No Tier  |

#### How did you convert currency?

To avoid repeating conversion logic later in the reporting layer, I standardised monetary values to EUR early in the ETL process. I joined the tables on a key of Date and Currency so that only the currency exchange rate relevant to the date of the transaction was considered.

##### ***Future Considerations:*** 
Although statically standardizing the currency worked for this project, it could be ideal to have dynamic currency conversion within tableau as well. Maybe having a toggle switch to view the metric in either Local Currency or Company Currency (EUR).

#### What did you find most difficult?

Across the stages of development, there were challenges of varying degrees of difficulty. Highlighting some of them below:

 - Coming up with thresholds and scoring/ranking methods to identify valuable players using a combination of different metrics. With additional context and data, this could have been an easier problem to tackle.
 - Working with rankings that are context and visual dependednt so they keep chaging on using filters. 
 - Handling multiple data integrity issues especially with number formats and dates. A lot of edge cases in game transaction table. Interesting challenge to take a call on how to handle them
 - Limited dataset size : Restricted the kind of KPIs I had to work with
	 - Day-on-day or week-on-week calculations not possible due to 1-day data available
	 - Small set of players with not enough variety of transactions
 - Building visuals that were meaningful and at the same time not just tables on a dashboard. 


#### How much time did you spend to build the application?

I invested between 13-16 hours overall across:
 - Data Analysis/Modelling (30%)
 - Dashboard Building (50%)
 - Documentation (20%)
