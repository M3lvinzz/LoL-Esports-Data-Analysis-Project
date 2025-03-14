<style>
table {
    width: 106%;
    border-collapse: collapse;
    text-align: center;
    margin-left: auto;
    margin-right: auto;
    overflow-x: auto;
    display: block;
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
- `totalcs` - The 'Creep score' is a metric which contains the number of minions, neutral objectives, and monster camps a team/player took.
- `totalgold` - 'gold' is a currency in the game given when eliminating champions, turrets, minions/monsters and neutral objectives. This represents the total gold gained by a player
- `result` - This column contains a binary True of False, which represents if a team won a game or not.

## Data Cleaning and Exploratory Data Analysis
One of the quirks in the Oracle's Elixir dataset is how the rows are set up between teams and individual players. Each game has twelve rows, ten rows being for the players, and two being for each team summarizing their overall team metrics. For this project, we need both the player rows and the team rows, but to keep the dataset clean the dataset was split between the player rows and the team rows. 

### Missing values
In both datasets, there were a few rows that contained missing values. The player rows contained no missing values for the relevant columns, however, the team rows contained a few missing values. Looking through the team rows, metrics such as `totalcs` were only present in the player rows. To solve this, the total damage was extracted from each player on every team by utilizing groupby on columns `gameid` and `side`. 

One important row that contained a few missing values was the `voidgrubs` column. Since this column is integral to the overall statistical analysis, conditional probabilistic imputation was used to impute the missing `voidgrubs`. By running a permutation test on missingness, it was shown that the missingness of `voidgrubs` is dependent on the kills column. While the essence of the permutation is crucial to understanding, the intricices of the hypothesis test is expanded on in [Assessment of Missingness](#assessment-of-missingness)

This is the head to the cleaned, imputed team rows dataset (internally called `cleaned_team_filled`) that is used for hypothesis testing:

| gameid             | side   | league   |   participantid |   voidgrubs |   kills |   totalcs | result   |
|:-------------------|:-------|:---------|----------------:|------------:|--------:|----------:|:---------|
| 10660-10660_game_1 | Blue   | DCup     |             100 |           1 |       3 |      1043 | False    |
| 10660-10660_game_1 | Red    | DCup     |             200 |           3 |      16 |      1100 | True     |
| 10660-10660_game_2 | Blue   | DCup     |             100 |           0 |       3 |       956 | False    |
| 10660-10660_game_2 | Red    | DCup     |             200 |           2 |      17 |      1114 | True     |
| 10660-10660_game_3 | Blue   | DCup     |             100 |           0 |      21 |       786 | True     |

When looking at the predictive model dataset, we are looking at the player rows instead of the team rows. However, since the `voidgrubs` column was imputed in `cleaned_team_filled`, the imputed `voidgrubs` in `cleaned_team_filled` must be merged with the player rows to get the void grubs taken for each jungler in each game.

This is the head to the cleaned player rows dataset (internally called `cleaned_jungle`) that will be used for the predictive model:

| gameid             | side   | position   |   kills |   deaths |   assists |   totalcs |   totalgold |   voidgrubs | result   |
|:-------------------|:-------|:-----------|--------:|---------:|----------:|----------:|------------:|------------:|:---------|
| 10660-10660_game_1 | Blue   | jng        |       0 |        4 |         3 |       153 |        8636 |           4 | False    |
| 10660-10660_game_1 | Red    | jng        |       1 |        0 |        12 |       169 |       10590 |           5 | True     |
| 10660-10660_game_2 | Blue   | jng        |       0 |        5 |         2 |       181 |        9526 |           3 | False    |
| 10660-10660_game_2 | Red    | jng        |       1 |        1 |        14 |       190 |       11770 |           6 | True     |
| 10660-10660_game_3 | Blue   | jng        |       6 |        1 |         7 |       162 |       10791 |           5 | True     |
### Univariate Analysis
To fully understand the dataset, we must look at the distributions of metrics in our dataset. Since `voidgrubs` is imperative in our analysis, looking at the distribution of `voidgrubs` is imperative.

<iframe
src= 'assets/univariate1.html'
width = 700
height = 450
frameborder = 0
></iframe>

Looking at the graph, it shows that most games have teams take either 0, 3, or 6 grubs respectively.

Additionally, gauging how the distribution looks for `totalcs` since it is an integral part of our hypothesis testing later on

<iframe
src= 'assets/univariate2.html'
width = 700
height = '450'
frameborder = '0'
></iframe>

Looking at the distribution of total cs, we see that it is relatively normally distributed, and shows that it is a good statistic for analyzing the impact of `voidgrubs`

### Bivariate Analysis
Since match wins are an important metric when preforming analysis on League of Legends data, looking at the relationship between number of `voidgrubs` and `result` become important in understanding the importance of `voidgrubs`

<iframe
src= 'assets/bivariate.html'
width = 700
height = '450'
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
In the dataset, there are many columns that have missing values. One column that is Not Missing at Random (NMAR) is `url`. This is because of the requirements for a column to be NMAR:
1. Not dependent on other columns
2. Can determine why a value is missing based on other values in the column/itself

The column `url` at a first glance is not dependent on any other columns, as the url for any game could be missing. For example, if a certain game was more under the radar, or the league itself didn't have a stats page for that game, then it could be missing. Additionally, we can determine missingness of the column by assuming that if the url is missing, it is because the league or viewers didn't make a page for this particular match.

A column that could help make this Missing at Random instead of Not Missing at Random is documenting where the data came from. For example, if the owner got the data from [Leaguepedia](https://lol.fandom.com/wiki/League_of_Legends_Esports_Wiki), then it would show where data is gotten. If they got the data from watching the [Twitch](https://www.twitch.tv/) stream, then it would list that. This way the missingness of `url` could be determined by where the data came from.

### Missingness Dependency

#### Void Grubs Missingness Dependency on `kills`
Since `voidgrubs` surprisingly has missing values, finding out the missingness of `voidgrubs` is crucial in figuring out which columns could be used to help impute `voigrubs`. In the above section, [Missing Values](#missing-values), the missingness of `voidgrubs` was imputed with `kills`. In this section, it will show the permutation testing used to determine the dependency on kills.

For this, the test statistic being used is Absolute Mean Difference (AMD) because `kills` is a numerical category, therefore AMD is better than Total Variation Distance (TVD), which is better for categorical distributions.

**Null Hypothesis** - The Absolute Mean Difference of `kills` when `voidgrubs` is missing is the same as the Absolute Mean Difference of `kills` when voidgrubs` is not missing

**Alternate Hypothesis** - The distribution of `kills` when `voidgrubs` is missing is **Not** the same as the distribution of `kills` when voidgrubs` is not missing

**Significance Level** - The significance level used is 0.05 (5%)

<iframe
src= 'assets/imputation_kills_missingness.html'
width = 700
height = '450'
frameborder = '0'
></iframe>

From the graph, and after the permutation testing, the observed AMD is a heavy outlier when preforming the permutation testing on missingness of `voidgrubs`. Since the p-value is 0.0, we **reject the null hypothesis**, therefore showing that there is a dependency on `kills` for the missingness of `voidgrubs`.

#### Void Grubs Missingness Dependency on `result`
However, when looking at the missingness of `voidgrubs` when paired with `result`, we get a different result. For this permutation test, we are using Total Variation Distance (TVD) because `result` is a categorical column, and when going through categorical columns, the more appropriate test statistic is TVD. 

The distribution of `voidgrubs` missingness and `result` can be seen in this table:

| result   |   voidgrubs = False |   voidgrubs = True |
|:---------|--------------------:|-------------------:|
| False    |            0.500059 |           0.500359 |
| True     |            0.499941 |           0.499641 |

**Null Hypothesis** - The distribution of `result` when `voidgrubs` is missing is the same as the distribution of `result` when `voidgrubs` is not missing.

**Alternate Hypothesis** - The distribution of `result` when `voidgrubs` is missing is the **Not** same as the distribution of `result` when `voidgrubs` is not missing.

**Significance Level** - The significance level used is the same as the last permutation test, 0.05 (5%)

<iframe
src= 'assets/missingness_result_grubs.html'
width = 700
height = '450'
frameborder = '0'
></iframe>

Looking at the graph, the observed TVD is very common in terms of the permutated TVDs. Because the p-value is ~0.981, we fail to reject the null hypothesis, showing that there is no dependence of `voidgrubs` missingness on `result`

## Hypothesis Testing
In hypothesis testing, the goal is to assess if there is a significant difference in the observed statistic. In this project, we are interested in how `totalcs` is affected by the presence of more grubs. 

For background, in League of Legends, having more Void Grubs generally leads to more towers being taken because of the increased tower damage. This leads to more minion waves being pushed, therefore leading to generally a higher increased `totalcs`. 

For this hypothesis test, Absolute Mean Difference (AMD) is more useful than TVD, because the column `totalcs` is given as a numerical value. This, along with `totalcs` being roughly normally distributed, leads to AMD being more sutible than TVD.

When deciding how to split up `voidgrubs` into two groups to allow for permutation testing, I decided to split it as:
- 0 - 3 void grubs
- 4 - 6 void grubs 

This is because since there are about three usual outcomes: getting 0 grubs, 3 grubs and 6 grubs, it's hard to split up the games evenly. I decided that 0 - 3 grubs is a group because it shows that a team either didn't prioritize grubs, or got them to trade for an early dragon. However, having more than 3 grubs shows that a team is prioritizing them and actively gives up other neutral objectives in order to get more grubs.

**Null Hypothesis** - The mean of `totalcs` for teams with more than 3 `voidgrubs` is the same for teams with 3 or less void grubs

**Alternate Hypothesis** - The mean of `totalcs` for teams with more than 3 `voidgrubs` is **not** the same for teams with 3 or less void grubs

**Significance Level** - The significance level is the other tests, 0.05 (5%)

<iframe
src= 'assets/hypothesis_testing.html'
width = 700
height = '450'
frameborder = '0'
></iframe>

Based on the hypothesis test preformed, a p-value of 0.108 leads the us **Failing to reject the Null Hypothesis**. This leads us to assume that getting more void grubs does not lead to getting more `totalcs`. This is because the difference observed is not statistically significant, so the distributions roughly look the same.

## Framing a Prediction Problem
Following the theme of analyzing the impact of Void Grubs, creating a predictive model based on Void Grubs becomes intriguing. Since the jungler for each team is usually responsible in gathering all the neutral objectives for each team, we should analyze the metrics from the jungler along with the number of Void Grubs to predict a game being won or lost. 

To be more precise, the predictive model in this project will be designed around the question: **Can we predict the result of the game based on the team's jungler statistics?**

This model will be predicting the binary values of if a game was won or not, and because of this, the model is considered a `Binary Classification Model`. The metric used to evaluate the model will be Accuracy, because since the amounts of wins and losses would be the same, which makes an even split between winning and losing. This makes accuracy better than F-1 because winning and losing are evenly distributed and F-1 works better if `results` were not evenly distributed. At the time of prediction, the data that would be known is: `kills`, `deaths`, `assists`, and `voidgrubs`. To prevent overfitting, the data in cleaned_jungle will be split into 80% training data and 20% testing data. Examples of what the data looks like will be provided in their respective sections.

## Baseline Model
For the baseline model, the sklearn model used is Random Forest Classifier because for classification models, it is very resistant to overfitting and allows for a more accurate prediction. The features used are: `kills`, `deaths`, `assists`, and `voidgrubs`.

This is what the X features look like in the training set:
|   kills |   deaths |   assists |   voidgrubs |
|--------:|---------:|----------:|------------:|
|       5 |        6 |         5 |           0 |
|       3 |        4 |         7 |           6 |
|       3 |        2 |         2 |           0 |
|       1 |        1 |        10 |           1 |
|       5 |        1 |        14 |           0 |

After splitting the data into training and testing data, a pipeline was made to standardize the X variables since they are quantitative variables, and to make all the statistics look similar. This was achieved by utilizing StandardScaler Transformer to do so. After fitting the model onto the training data, the model gave an Accuracy score of **0.854**, which shows that this baseline model can accurately predict the result of games **~85.4%** of the time. 

When calling the Classification Report, these statistics were listed:
| Class    | Precision | Recall | F1-Score | Support |
|----------|-----------|--------|----------|---------|
| False    | 0.86      | 0.85   | 0.86     | 1998    |
| True     | 0.85      | 0.85   | 0.85     | 1922    |
| **Accuracy** |        |        | **0.85** | 3920    |
| **Macro Avg** | 0.85  | 0.85   | 0.85     | 3920    |
| **Weighted Avg** | 0.85 | 0.85 | 0.85     | 3920    |
*Note that False = Game Lost, True = Game Won

This classification report shows that recall and precision are relatively equally balanced and show no distinct outliers. Additionally, the advent of having an 85% accurate test shows that this model, at the very least, preforms admirably without any fine tuning. However, while this model preforms well, in the next section some adjustments will be made to further improve this model's accuracy.

## Final Model

To improve the model, I added two more features, `totalgold` and `totalcs`. `totalgold` is added because generally, in League of Legends, having more gold leads to more items and a generally stronger player. Having a stronger player means that they would be more likely to win. `totalcs` is added because the amount the jungler is farming dictates a few other metrics in the game, such as experience points (XP), gold gained, and neutral objectives taken. Having a higher Creep Score (CS) generally leads to a stronger player, and therefore a highly likelyhood of winning games. I expect these features to give the predictive model a higher accuracy.

To ensure the same training data used from the [Baseline Model](#baseline-model), the base training and test data was merged with the new columns based on index. This ensures the same data being used for both models, and allows for an accurate assessment between both models. The new columns are also added into the pipeline and standardized using Standard Scaler Transformer since `totalcs` and `totalgold` are quantitative variables.

This is what the X features look like in the improved training set:

|   kills |   deaths |   assists |   voidgrubs |   totalcs |   totalgold |
|--------:|---------:|----------:|------------:|----------:|------------:|
|       5 |        6 |         5 |           0 |       148 |        9854 |
|       3 |        4 |         7 |           6 |       202 |       11539 |
|       3 |        2 |         2 |           0 |       173 |        9671 |
|       1 |        1 |        10 |           1 |       262 |       13423 |
|       5 |        1 |        14 |           0 |       301 |       16289 |

*Notice how the training data in [Baseline Model](#baseline-model) is the same here, just with more columns added.

The final model used is the same Random Forest Classifier used in the [Baseline Model](#baseline-model). However, I added more optimized hyperparameters utilizing GridSearchCV. Inputting hyperparameters, `max_depth`, `min_sample_split`, `n_estimators` and `criterion`. After running GridSearchCV, the optimal hyperparameters are:
- `max_depth` - 10
- `min_sample_split` - 10
- `n_estimators` - 200
- `criterion` - entropy

After running the improved model, an accuracy of **0.8854** is achieved. This shows that instead of the Baseline Model's accuracy of **~85.4%**, the model can accurately predict the outcome of games **~88.54%** of the time. 

When calling the Classification Report, these statistics were listed:

|               | Precision | Recall | F1-Score | Support |
|---------------|-----------|--------|----------|---------|
| **False**     | 0.89      | 0.87   | 0.88     | 1955    |
| **True**      | 0.88      | 0.90   | 0.89     | 1965    |
| **Accuracy**  |           |        | 0.89     | 3920    |
| **Macro avg** | 0.89      | 0.89   | 0.89     | 3920    |
| **Weighted avg** | 0.89   | 0.89   | 0.89     | 3920    |

This classification report shows marginal improvement from both Precision and Recall. Additionally since all metrics (Precision, Recall, Accuracy) are higher than the Baseline Model, this shows that the improved model is more effective in predicting power.

Displayed is the Confusion Matrix for the improved model:

![confusion_graph]('/assets/confusion_fig.png')

Looking at the Confusion Matrix, it can be seen that not a lot of false positives or false negatives were seen when predicting the model. This shows a visualization of the model's accuracy, and therefore the predicting power of the improved model.

## Fairness Analysis
For determining if our model is fair, I split the data into the 2 groups used for [Hypothesis Testing](#hypothesis-testing). For a refresher, the groups are:
- **X** - 0 - 3 void grubs
- **Y** 4 - 6 void grubs 

The metric used is, however, different from the one used to determine the predictive power in [Final Model](#final-model). This time, since the groups 0-3 and 4-6 void grubs are different sizes, I will be using the **F-1 Score** to describe the difference between predicted and true values.

**Null Hypothesis** - The final model is fair, as the F-1 Score for players with 0-3 void grubs is the same as players who have 4-6 void grubs

**Alternate Hypothesis** - The final model is not fair, as the F-1 Score for players with 0-3 void grubs is the same as players who have 4-6 void grubs

**Significance Level** - The significance level is the other tests, 0.05 (5%)

<iframe
src= 'assets/fairness_assessment.html'
width = 700
height = '450'
frameborder = '0'
></iframe>

After preforming the permutation testing, it is shown that the p_value is 0.0, which is much smaller than the significance value of 0.05. This means the test **Rejected the Null Hypothesis**, unfortunately implying that the model is biased in it's predictions. Specifically, it seems to be biased towards players with 4-6 grubs. 
