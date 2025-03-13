<style>
table {
    display: block;
    overflow-x: auto;
    white-space: nowrap;
    max-width: 100%;
}
table th, table td {
    padding: 8px;
    border: 1px solid #ddd;
}
</style>

# League of Legends: Analyzing Void Grubs
Authors: Elvin Chen

## Introduction
### General Introduction
League of Legends is a popular multiplayer online battle arena (MOBA) game developed by Riot Games, where two teams of five players battle to destroy the enemy's Nexus while strategizing with unique champions, abilities, and team compositions. Since its release in 2009, it has grown into one of the most played games worldwide, boasting a massive competitive scene. The League of Legends esports ecosystem includes regional leagues like the LCS, LEC, LCK, and LPL, culminating in the prestigious World Championship, where the best teams from around the globe compete for glory and millions in prize money.

Focusing on Esports, the dataset being used for this project is a professional dataset developed by [Oracle's Elixir](https://oracleselixir.com/about). This dataset records the professional matches for every single year of esports, but for this project the 2024 Esports data will be used. Oracle's Elixir recorded the intricacies for every single match, such as the kill death ratio (KDA), gold spent, minion kills, objectives, etc. This allows for a very deep look for the overall match dynamics and opens the door for statistical analysis.

In League of Legends, there exists many different neutral objectives players must take in order to create a lead against the enemy team. One of the newest neutral objectives is the Void Grubs, introduced in 2024. To give a brief summary, Void Grubs spawn in two waves as groups of three; one at 6 minutes, and the second wave four minutes after the first group is taken. Each Grub taken provides a team wide buff to tower damage, which can help speed up the progression of the slayers team. While taking Void Grubs provides an immediate boost for the team that took it, taking Void Grubs often have long-term effects on the game, shaping the overall team strategies and opposing team interactions.

The Research Question we want to explore further is **How does securing different amounts of Void Grubs impact a team's overall gameplay performance and win rate in League of Legends?** Looking through the dataset, we want to utilize data analysis to find to what degree of impact Void Grubs has on the overall team performance, in-game metrics and match outcomes. Later on we would use the individual's performances to predict the role (top, jungle, mid, bot, support) the player is.

### Dataset Introduction
The dataset has an extensive array of columns including gameplay metrics and match outcomes for professional leagues for League of Legends Esports Matches. In total, the raw dataset has around 117576 rows and 161 columns. However, I will be splitting up the dataset to focus on the Hypothesis testing and the Predictive model.

While creating 2 datasets, the columns used are described here:
- `gameid` - This column contains a unique id for every match played. This makes distinguishing between different matches easy.
- `side` - This column represents the side of the map the team is on. This is a binary column that contains either 'Blue' or 'Red' team
- `league` - This column represents the professional league/tournament the game played is in. Some examples of leagues are LPL, LCK, LEC, etc.
- `participantid` - This column represents the id of each player in each match. Usually it is between 1-10 for players, reserving 100 and 200 for overall team performance
- `position` - This column contains the position of the player. The positions are: 'top', 'jungle', 'mid', 'bot', 'support'.
- `voidgrubs` - This column contains the total number of void grubs taken by each team. Values range from 1-6 inclusive.
- `kills` - The 'kills' column contains the number of enemy champions a team/player eliminated from the match.
- `deaths` - The 'deaths' column records the number of times a team/player was eliminated from the match
- `assists` - The 'assists' column represents the number of assists a team/player has in a match. An 'assist' is when a player contributes in eliminating an enemy champion, but didn't get the final hit
- `totalCS` - The 'Creep score' is a metric which contains the number of minions, neutral objectives, and monster camps a team/player took.
- `totaldamage` - The 'damage' done in this column is only towards enemy champions. This represents the numerical damage a player did towards the opposing players
- `totalgold` - 'gold' is a currency in the game given when eliminating champions, turrets, minions/monsters and neutral objectives. This represents the total gold gained by a player
- `result` - This column contains a binary True of False, which represents if a team won a game or not.

## Data Cleaning and Exploratory Data Analysis
One of the quirks in the Oracle's Elixir dataset is how the rows are set up between teams and individual players. Each game has twelve rows, ten rows being for the players, and two being for each team summarizing their overall team metrics. For this project, we need both the player rows and the team rows, but to keep the dataset clean the dataset was split between the player rows and the team rows. 

### Missing values
In both datasets, there were a few rows that contained missing values. The player rows contained no missing values for the relevant columns, however, the team rows contained a few missing values. Looking through the team rows, metrics such as `totalcs` were only present in the player rows. To solve this, the total damage was extracted from each player on every team by utilizing groupby on columns `gameid` and `side`. 

One important row that contained a few missing values was the `voidgrubs` column. Since this column is integral to the overall statistical analysis, conditional probabilistic imputation was used to impute the missing `voidgrubs`. By running a permutation test on missingness, it was shown that the missingness of `voidgrubs` is dependent on the kills column. While the essence of the permutation is crucial to understanding, the intricices of the hypothesis test is expanded on in [Assessment of Missingness](#assessment-of-missingness)

This is the head to the cleaned, imputed team rows dataset (internally called cleaned_team_filled) that is used for hypothesis testing:

|    | gameid             | side   | league   |   participantid |   voidgrubs |   kills |   totalcs | result   |
|---:|:-------------------|:-------|:---------|----------------:|------------:|--------:|----------:|:---------|
|  0 | 10660-10660_game_1 | Blue   | DCup     |             100 |           2 |       3 |      1043 | False    |
|  1 | 10660-10660_game_1 | Red    | DCup     |             200 |           2 |      16 |      1100 | True     |
|  2 | 10660-10660_game_2 | Blue   | DCup     |             100 |           0 |       3 |       956 | False    |
|  3 | 10660-10660_game_2 | Red    | DCup     |             200 |           5 |      17 |      1114 | True     |
|  4 | 10660-10660_game_3 | Blue   | DCup     |             100 |           0 |      21 |       786 | True     |

This is the head to the cleaned player rows dataset (internally called cleaned_player) that will be used for the predictive model:

|    | gameid             |   participantid | position   |   kills |   deaths |   assists |   totaldamage |   totalcs |   totalgold |
|---:|:-------------------|----------------:|:-----------|--------:|---------:|----------:|--------------:|----------:|------------:|
|  0 | 10660-10660_game_1 |               1 | top        |       1 |        3 |         1 |          7092 |       279 |       11083 |
|  1 | 10660-10660_game_1 |               2 | jng        |       0 |        4 |         3 |          7361 |       153 |        8636 |
|  2 | 10660-10660_game_1 |               3 | mid        |       0 |        2 |         0 |         10005 |       270 |       10743 |
|  3 | 10660-10660_game_1 |               4 | bot        |       2 |        4 |         0 |         10892 |       311 |       12224 |
|  4 | 10660-10660_game_1 |               5 | sup        |       0 |        3 |         3 |          6451 |        30 |        7221 |

### Univariate Analysis
To fully understand the dataset, we must look at the distributions of metrics in our dataset. Since `voidgrubs` is imperative in our analysis, looking at the distribution of `voidgrubs` is imperative.

<iframe
src= 'assets/univariate1.html'
width = 600
height = 400
frameborder = 0
></iframe>

Looking at the graph, it shows that most games have teams take either 0, 3, or 6 grubs respectively.

Additionally, gauging how the distribution looks for `totalcs` since it is an integral part of our hypothesis testing later on

<iframe
src= 'assets/univariate2.html'
width = 600
height = '400'
frameborder = '0'
></iframe>

Looking at the distribution of total cs, we see that it is relatively normally distributed, and shows that it is a good statistic for analyzing the impact of `voidgrubs`

### Bivariate Analysis
Since match wins are an important metric when preforming analysis on League of Legends data, looking at the relationship between number of `voidgrubs` and `result` become important in understanding the importance of `voidgrubs`

<iframe
src= 'assets/bivariate.html'
width = 600
height = '400'
frameborder = '0'
></iframe>

According to the plots, it is shown that 'games_won' overtake the 'games_lost' when having 3+ grubs. This shows that there might be a correlation between winning games and having more void grubs, especially having 6 grubs, which have a much higher ratio of 'games_won' compared to 'games_lost'

### Interesting Aggregate
Aggregating the cleaned_team_filled, and interesting pattern appears:

|   voidgrubs |   kills |   totalcs |   result |
|------------:|--------:|----------:|---------:|
|           0 | 14.0272 |   1012.7  | 0.407293 |
|           1 | 14.3886 |   1026.74 | 0.443273 |
|           2 | 14.7525 |   1042.56 | 0.467936 |
|           3 | 15.398  |   1048.3  | 0.504607 |
|           4 | 15.3335 |   1045.04 | 0.527848 |
|           5 | 16.2454 |   1037.41 | 0.58623  |
|           6 | 16.2573 |   1029.11 | 0.617502 |

To achieve this aggregate dataframe, the cleaned_team_filled dataset was slimmed down to only include: `voidgrubs`, `kills`, `totalcs`, `result`. This slimmed down dataframe was then condensed using groupby on `voidgrubs` with mean as it's aggregate function. Looking at kills, the average number of kills goes up slightly as more `voidgrubs` are taken. `totalcs`, however, is shown to be relatively the same, since there number go up and down as `voidgrubs` goes up. `result`, however, goes up as `voidgrubs` goes up, implying there is a relationship between winning and winning games.

## Assessment of Missingness
### NMAR Analysis 

## Hypothesis Testing
## Framing a Prediction Problem
## Baseline Model
## Final Model
## Fairness Analysis