---
layout: page
title: Game Statistics
---

Create the following methods:

| Method | Description | Return Value |
| ------ | ----------- | ------------ |
|`StatTracker#highest_total_score`| Highest sum of the winning and losing teams' scores | Integer |
|`StatTracker#lowest_total_score`| Lowest sum of the winning and losing teams' scores | Integer |
|`StatTracker#biggest_blowout`| Highest difference between winner and loser| Integer |
|`StatTracker#count_of_ties`| Number of ties across all seasons and games | Integer |
|`StatTracker#most_popular_venue`| Name of the venue with the most total games played | String |
|`StatTracker#least_popular_venue`| Name of the venue with the fewest total games played|  String |
|`StatTracker#percentage_home_wins`| Percentage of time that a home team has won a game (rounded to the nearest 100th) | Decimal |
|`StatTracker#percentage_visitor_wins`| Percentage of games that a visitor wins (rounded to the nearest 100th)  |  Decimal |
|`StatTracker#percentage_ties`| Percentage of games that ended in a tie (rounded to the nearest 100th) |  Decimal |
|`StatTracker#season_with_most_games`| Integer representation (e.g. 20122013) of the season in which the most games were held | Integer |
|`StatTracker#season_with_fewest_games`| Integer representation (e.g. 20122013) of the season with the fewest games | Integer |
|`StatTracker#count_of_games_by_season`| A hash with season names (e.g. 20122013) as keys and counts of games as values  | Hash |
|`StatTracker#average_goals_per_game`| Average number of goals scored in a game across all seasons (rounded to the nearest 100th)| Decimal |
|`StatTracker#average_goals_by_season`| Average number of goals scored in a game organized in a hash with season names (e.g. 20122013) as keys and a decimal representing the average number of goals in a game for that season as a key (rounded to the nearest 100th)| Hash |
